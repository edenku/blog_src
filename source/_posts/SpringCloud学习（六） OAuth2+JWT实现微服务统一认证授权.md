---
title: SpringCloud学习 | 第六篇 Gateway + Spring Security + OAuth2 + JWT实现微服务统一认证授权
categories: SpringCloud从零开始构建微服务
tags: SpringCloud
date: 2021-05-20
---



### OAuth2 与 JWT

#### OAuth2

简单些说就是这套协议包含两个核心角色，认证服务器和资源服务器。

在我们的微服务结构中，网关对应资源服务，auth对应认证服务。

| 模块名         | 模块作用 | OAuth2角色 | 服务地址       |
| -------------- | -------- | ---------- | -------------- |
| ruipin-gateway | 网关     | 资源服务器 | localhost:9999 |
| ruipin-auth    | 认证中心 | 认证服务器 | localhost:8000 |

#### JWT

JWT(JSON Web Token)，一个无状态的token，本身携带用户的信息。

JWT字符串由Header、Payload、Signature三部分组成。

<!-- more -->

```
Header: Json对象，描述JWT的元数据，alg属性表示算法签名，type标识token类型
Payload：Json对象，重要部分，除默认字段，还可以扩展自定义字段，比如用户ID、姓名、角色等。
Signature：对Header、Payload两部分签名，认证服务器私钥签名，然后资源服务器用公钥验签，防止数据篡改。
```

JWT相比传统Cookie/Session会话管理的重要优势就在于无状态、去中心化，所以JWT更适合分布式场景。

#### OAuth2与JWT的关系

OAuth2是认证授权的协议规范。

JWT是基于token的安全认证协议的实现。

OAuth2认证服务器签发的token可以使用JWT实现，JWT轻量且安全。

#### OAuth2四种授权模式

+ Authorization Code（授权码模式）：正宗的OAuth2的授权模式，客户端先将用户导向认证服务器，登录后获取授权码，然后进行授权，最后根据授权码获取访问令牌
+ Implicit（简化模式）：和授权码模式相比，取消了获取授权码的过程，直接获取访问令牌；
+ Resource Owner Password Credentials（密码模式）：客户端直接向用户获取用户名和密码，之后向认证服务器获取访问令牌；
+ Client Credentials（客户端模式）：客户端直接通过客户端认证（比如client_id和client_secret）从认证服务器获取访问令牌。

此处我们主要选择密码模式

![image-20210520104647787](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210520104647787.png)

+ 客户端从用户获取用户名和密码。
+ 客户端通过用户名和密码访问认证服务器。
+ 认证服务器返回访问令牌。



### SpringCloud整合OAuth2和JWT

涉及到的服务主要有：

| 模块           | 端口 | 描述                  |
| -------------- | ---- | --------------------- |
| Nacos servr    | 8848 | 注册中心              |
| ruipin-sys     | 8060 | 服务提供方            |
| ruipin-sys-api | --   | 服务API               |
| ruipin-auth    | 8000 | 认证服务器            |
| ruipin-gateway | 9999 | 网关服务器+资源服务器 |

这里大致梳理下认证的一个简单流程：

+ 前端请求 http://ip:9999/auth/oauth/token服务获取token。
  + 请求auth，auth先判断client是否有效，再调用用户模块验证用户是否存在，
  + 存在，则将用户信息（基本信息+角色权限信息）写入token
+ 前端携带token访问http://ip:9999/sys/serviceA/服务访问具体服务。
+ 网关根据token 解析，判断用户是否有权限访问接口。如有，则转发请求到sys服务，如无则返回无权限。

#### ruipin-sys

目前主要提供3个功能：

1. 提供获取用户信息接口，用于获取token时的鉴权。
2. 提供获取用户信息接口，用于后面用户携带token的鉴权测试。（此处我是偷懒用了同一个接口）
3. 提供初始化角色权限job，项目启动时redis缓存 各接口所需角色信息，便于2的鉴权。

##### pom依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- SpringCloud从2020版本后对配置文件加载进行了重构，
        默认不加载bootstrap配置文件，添加bootstrap依赖解决该问题 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>

<!-- 注册中心 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>

<!-- 配置中心 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>

<dependency>
    <groupId>com.ruipin</groupId>
    <artifactId>ruipin-sys-api</artifactId>
    <version>${ruipin.version}</version>
</dependency>
<dependency>
    <groupId>com.ruipin</groupId>
    <artifactId>common-mybatis</artifactId>
    <version>${ruipin.version}</version>
</dependency>
<dependency>
    <groupId>com.ruipin</groupId>
    <artifactId>common-redis</artifactId>
    <version>${ruipin.version}</version>
</dependency>
```

##### 配置文件

```yaml
server:
  port: 8060

spring:
  application:
    name: ruipin-sys
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://49.4.78.75:3306/ruipin_test?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: ruipintest
    password: pass@word1
  cloud:
    nacos:
      discovery:
        server-addr: http://localhost:8848
      config:
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        file-extension: yaml
mybatis-plus:
  mapper-locations: classpath:mapping/*.xml
  configuration:
    # 驼峰下划线转换
    map-underscore-to-camel-case: true
    # 这个配置会将执行的sql打印出来，在开发或测试的时候可以用
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

这里可以使用Mybatis-Plus Generator插件，对应数据库生成mybatis相关代码。我们这里暂时都是写死的逻辑，所以可以不需要。

```
// 因为我们写死逻辑，所以配置目前不必需
@Configuration
@MapperScan("com.ruipin.sys.mapper")
public class MybatisConfig {
}
```

提供一个获取用户信息的接口：

```java
@RestController
@RequestMapping("/userAccount")
public class UserAccountController {
		
	/*
	 * 接口直接构建了一个用户对象，密码根据目前的配置使用的BCypt加密的123456
	 * 该用户拥有ADMIN角色
	 */
    @GetMapping("/username/{username}")
    public Result getUserAccountByUsername(@PathVariable String username){
        AccountDto account = new AccountDto();
        account.setId(1l);
        account.setUsername(username);
        account.setPassword("$2a$10$cnaQ7AbWqzEs9EFg59xO5eLqJcs4..BY7mCI8pEQ6Cu.iF2YHPV.m");
        account.setRoles(Arrays.asList("ADMIN"));
        return Result.success(account);
    }
}
```

建一个Job，在项目启动时加载缓存信息：

```
@Component
@AllArgsConstructor
@Slf4j
public class InitPermissionJob implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        refreshFrontMenuCache();
    }
    
    @Override
    public boolean refreshFrontMenuCache() {
    	// 此处也是写死的逻辑，稍后调试通以后可以接入Mybatis数据库
        redisTemplate.delete(RedisKeys.RESOURCE_ROLES_MAP);
        Map<String, List<String>> map = new TreeMap<>();
        map.put("/sys/**", CollUtil.toList("ROLE_ADMIN")); // ROLE_是jwt
        redisTemplate.opsForHash().putAll(RedisKeys.RESOURCE_ROLES_MAP, map);
        return true;
    }
}
```

建立启动类

```
@SpringBootApplication
@EnableDiscoveryClient
public class SysApplication {
    public static void main(String[] args) {
        SpringApplication.run(SysApplication.class, args);
    }
}
```

#### ruipin-sys-api

##### pom依赖

```xml
<!-- 远程调用HTTP客户端 -->
<!-- openfeign依赖 1. http客户端选择okhttp 2. loadbalancer替换ribbon -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>

<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
</dependency>
```

##### feign接口

```
@FeignClient(name="ruipin-sys", contextId = "account")
public interface AccountFeignClient {

    @GetMapping("/userAccount/username/{username}")
    Result<AccountDto> getUserAccountByUsername(@PathVariable String username);
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public class AccountDto {

    private Long id;
    private String username;
    private String password;
    private List<String> roles;
}
```

#### ruipin-auth

auth主要有以下几个：

​	JwtTokenEnhancer： JWT增强配置

   AuthorizationServerConfig 认证核心配置

   WebSecurityConfig  访问核心配置

   AuthController 提供获取token接口

   PublicKeyController 提供公钥获取接口

   SecurityUser 提供token鉴权的用户信息

   UserDetailServiceImpl 前期用户查询

##### pom依赖

```
<dependencies>
    <!-- SpringCloud从2020版本后对配置文件加载进行了重构，
    默认不加载bootstrap配置文件，添加bootstrap依赖解决该问题 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bootstrap</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>
    <!-- 注册中心 -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>

    <!-- nacos-config 配置中心依赖-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>

    <!-- OAuth2 认证服务器-->
    <dependency>
        <groupId>org.springframework.security.oauth.boot</groupId>
        <artifactId>spring-security-oauth2-autoconfigure</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-jose</artifactId>
    </dependency>

    <dependency>
        <groupId>com.ruipin</groupId>
        <artifactId>common-core</artifactId>
    <version>${ruipin.version}</version>
    </dependency>

    <dependency>
        <groupId>com.ruipin</groupId>
        <artifactId>ruipin-demo-api</artifactId>
   		<version>${ruipin.version}</version>
    </dependency>

    <dependency>
        <groupId>com.ruipin</groupId>
        <artifactId>ruipin-sys-api</artifactId>
        <version>${ruipin.version}</version>
    </dependency>
    <dependency>
        <groupId>com.ruipin</groupId>
        <artifactId>common-mybatis</artifactId>
        <version>${ruipin.version}</version>
    </dependency>
</dependencies>
```

##### 配置文件

```yaml
server:
  port: 8000
spring:
  application:
    name: ruipin-auth
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3307/ruipin_2021?zeroDateTimeBehavior=convertToNull&useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai&autoReconnect=true
    username: root
    password: root
  cloud:
    nacos:
      discovery:
        server-addr: http://127.0.0.1:8848
      config:
        file-extension: yaml      # 默认值 properties
        group: DEFAULT_GROUP      # 默认值 DEFAULT_GROUP
        prefix: ${spring.application.name}
example:
  hello-word: Hello Ruipin
feign:
  okhttp:
    enabled: true
```

##### AuthorizationServerConfig

```java

@AllArgsConstructor
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    private final UserDetailServiceImpl userDetailsService;
    private final AuthenticationManager authenticationManager;
    private final JwtTokenEnhancer jwtTokenEnhancer;
    private DataSource dataSource;
    private PasswordEncoder passwordEncoder;


    @Override
    @SneakyThrows
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
            .withClient("client-app")//配置client_id
            .secret(passwordEncoder.encode("123456"))//配置client_secret
            .scopes("all")//配置申请的权限范围
            .authorizedGrantTypes("password", "refresh_token")
            //配置grant_type，表示授权类型
            .accessTokenValiditySeconds(3600)//配置访问token的有效期
            .refreshTokenValiditySeconds(86400);//配置刷新token的有效期
    }

    /**
     * 密码模式需要配置
     * @param endpoints
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        TokenEnhancerChain enhancerChain = new TokenEnhancerChain();
        List<TokenEnhancer> delegates = new ArrayList<>();
        delegates.add(jwtTokenEnhancer);
        delegates.add(accessTokenConverter());
        enhancerChain.setTokenEnhancers(delegates); //配置JWT的内容增强器
        endpoints.authenticationManager(authenticationManager)
                .userDetailsService(userDetailsService) //配置加载用户信息的服务
                .accessTokenConverter(accessTokenConverter())
                .tokenEnhancer(enhancerChain);
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.allowFormAuthenticationForClients();
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
        jwtAccessTokenConverter.setKeyPair(keyPair());
        return jwtAccessTokenConverter;
    }

    @Bean
    public KeyPair keyPair() {
        KeyStoreKeyFactory factory = new KeyStoreKeyFactory(new 	          ClassPathResource("ruipin.jks"), "123456".toCharArray());
        return factory.getKeyPair("ruipin", "rui_pin2021".toCharArray());
    }

}
```

##### WebSecurityConfig

```java

@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .requestMatchers(EndpointRequest.toAnyEndpoint()).permitAll()
                .antMatchers("/rsa/publicKey").permitAll()
                .anyRequest().authenticated();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

}
```

##### AuthController与PublicKeyController

```java
@RestController
@RequestMapping("/oauth")
@AllArgsConstructor
@Slf4j
public class AuthController {

    private TokenEndpoint tokenEndpoint;

    @PostMapping("/token")
    public OAuth2AccessToken postAccessToken(
            Principal principal,
            @RequestParam Map<String, String> parameters
    ) throws HttpRequestMethodNotSupportedException {
        OAuth2AccessToken oAuth2AccessToken;

        oAuth2AccessToken = tokenEndpoint.postAccessToken(principal, parameters).getBody();
        return oAuth2AccessToken;
    }
}


@RestController
@RequestMapping
@Slf4j
public class PublicKeyController {

    @Autowired
    private KeyPair keyPair;

    @GetMapping("/rsa/publicKey")
    public Map<String, Object> loadPublicKey() {
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        RSAKey key = new RSAKey.Builder(publicKey).build();
        return new JWKSet(key).toJSONObject();
    }

}
```

##### SecurityUser

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class SecurityUser implements UserDetails {

    private Long id;

    private String username;

    private String password;

    private String clientId;

    private Collection<SimpleGrantedAuthority> authorities;

    public SecurityUser(AccountDto account) {
        this.setId(account.getId());
        this.setUsername(account.getUsername());
//        this.setPassword(AuthConstants.BCRYPT + account.getPassword());
        this.setPassword(account.getPassword());
        if (CollectionUtil.isNotEmpty(account.getRoles())) {
            authorities = new ArrayList<>();
            account.getRoles().forEach(roleId -> authorities.add(new SimpleGrantedAuthority(String.valueOf(roleId))));
        }
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

##### UserDetailServiceImpl

```java

@Service
@AllArgsConstructor
@Slf4j
public class UserDetailServiceImpl implements UserDetailsService {

    @Autowired
    private AccountFeignClient accountFeignClient;
    @Autowired
    private PasswordEncoder passwordEncoder;
    private List<AccountDto> userList;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException{
        AccountDto accountDto = new AccountDto();
        Result result = accountFeignClient.getUserAccountByUsername(username);
        log.info("获取用户信息:{}", result);
        if (ResultCode.SUCCESS.getCode().equals(result.getCode())) {
            accountDto = (AccountDto) result.getData();
            securityUser = new SecurityUser(accountDto);
        }
        securityUser = new SecurityUser(accountDto);
        return securityUser;
    }
}
```

##### JWT增强

```
/**
 * JWT内容增强器
 */
@Component
public class JwtTokenEnhancer implements TokenEnhancer {
    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication) {
        SecurityUser securityUser = (SecurityUser) authentication.getPrincipal();
        Map<String, Object> info = new HashMap<>();
        //把用户ID设置到JWT中
        info.put("id", securityUser.getId());
        ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(info);
        return accessToken;
    }
}
```

##### 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class AuthApplication {
    public static void main(String[] args) {
        SpringApplication.run(AuthApplication.class, args);
    }
}
```

#### ruipin-gateway

##### pom依赖

```
<dependencies>
    <!-- SpringCloud从2020版本后对配置文件加载进行了重构，
    默认不加载bootstrap配置文件，添加bootstrap依赖解决该问题 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bootstrap</artifactId>
    </dependency>
    <!-- 2020Cloud版本，不加该依赖可能在路由转发时会报503 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>

    <!-- 注册中心 -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>

    <!-- oauth2 资源服务器-->
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-resource-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-jose</artifactId>
    </dependency>

    <dependency>
        <groupId>com.ruipin</groupId>
        <artifactId>common-core</artifactId>
    </dependency>
    <dependency>
        <groupId>com.ruipin</groupId>
        <artifactId>common-redis</artifactId>
    	<version>1.0</version>
    </dependency>
</dependencies>
```

##### 配置文件

```
server:
  port: 9999

spring:
  application:
    name: ruipin-gateway
  cloud:
    nacos:
      # 注册中心
      discovery:
        server-addr: http://127.0.0.1:8848
      # 配置中心
      config:
        server-addr: ${spring.cloud.nacos.discovery.server-addr}
        file-extension: yaml
    gateway:
      discovery:
        locator:
          enabled: true  # 启用动根据服务id生成路由
          lower-case-service-id: true # 设置路由的路径为小写的服务ID
      routes:
        - id: auth-route
          uri: lb://ruipin-auth
          predicates:
            - Path=/auth/**  # 根据路径进行匹配，所有/auth/**开头的请求都会转发到 lb://ruipin-auth
          filters:
            - StripPrefix=1 # 移除前缀 /auth
        - id: sys-route
          uri: lb://ruipin-sys
          predicates:
            - Path=/sys/**
          filters:
            - StripPrefix=1 # 移除前缀 /sys
  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: 'http://localhost:8000/rsa/publicKey'
white-list:
  urls:
    - "/auth/oauth/token"

```

##### ResourceServerConfig

```
package com.ruipin.gateway.config;

import cn.hutool.core.util.ArrayUtil;
import com.ruipin.common.constant.AuthConstants;
import com.ruipin.common.result.ResultCode;
import com.ruipin.gateway.security.AuthorizationManager;
import com.ruipin.gateway.util.WebUtils;
import lombok.AllArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.convert.converter.Converter;
import org.springframework.security.authentication.AbstractAuthenticationToken;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationConverter;
import org.springframework.security.oauth2.server.resource.authentication.JwtGrantedAuthoritiesConverter;
import org.springframework.security.oauth2.server.resource.authentication.ReactiveJwtAuthenticationConverterAdapter;
import org.springframework.security.web.server.SecurityWebFilterChain;
import org.springframework.security.web.server.ServerAuthenticationEntryPoint;
import org.springframework.security.web.server.authorization.ServerAccessDeniedHandler;
import reactor.core.publisher.Mono;

/**
 * 资源服务器配置
 */
@AllArgsConstructor
@Configuration
@EnableWebFluxSecurity
public class ResourceServerConfig {

    private AuthorizationManager authorizationManager;
    private WhiteListConfig whiteListConfig;

    @Bean
    public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
        http.oauth2ResourceServer().jwt()
                .jwtAuthenticationConverter(jwtAuthenticationConverter());
        //自定义处理JWT请求头过期或签名错误的结果
        http.oauth2ResourceServer().authenticationEntryPoint(authenticationEntryPoint());
        //处理白名单请求
        http.authorizeExchange()
                .pathMatchers(ArrayUtil.toArray(whiteListConfig.getUrls(), String.class)).permitAll()
                .anyExchange().access(authorizationManager)//鉴权管理器配置
                .and()
                .exceptionHandling()
                .accessDeniedHandler(accessDeniedHandler()) // 处理未授权
                .authenticationEntryPoint(authenticationEntryPoint()) //处理未认证
                .and().csrf().disable();
        return http.build();
    }

    /**
     * 未授权
     *
     * @return
     */
    @Bean
    ServerAccessDeniedHandler accessDeniedHandler() {
        return (exchange, denied) -> {
            Mono<Void> mono = Mono.defer(() -> Mono.just(exchange.getResponse()))
                    .flatMap(response -> WebUtils.writeFailedToResponse(response, ResultCode.ACCESS_UNAUTHORIZED));
            return mono;
        };
    }

    /**
     * token无效或者已过期自定义响应
     */
    @Bean
    ServerAuthenticationEntryPoint authenticationEntryPoint() {
        return (exchange, e) -> {
            Mono<Void> mono = Mono.defer(() -> Mono.just(exchange.getResponse()))
                    .flatMap(response -> WebUtils.writeFailedToResponse(response,ResultCode.TOKEN_INVALID_OR_EXPIRED));
            return mono;
        };
    }

    /**
     * @return
     * @link https://blog.csdn.net/qq_24230139/article/details/105091273
     * ServerHttpSecurity没有将jwt中authorities的负载部分当做Authentication
     * 需要把jwt的Claim中的authorities加入
     * 方案：重新定义R 权限管理器，默认转换器JwtGrantedAuthoritiesConverter
     */
    @Bean
    public Converter<Jwt, ? extends Mono<? extends AbstractAuthenticationToken>> jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter jwtGrantedAuthoritiesConverter = new JwtGrantedAuthoritiesConverter();
        jwtGrantedAuthoritiesConverter.setAuthorityPrefix(AuthConstants.AUTHORITY_PREFIX);
        jwtGrantedAuthoritiesConverter.setAuthoritiesClaimName(AuthConstants.JWT_AUTHORITIES_KEY);

        JwtAuthenticationConverter jwtAuthenticationConverter = new JwtAuthenticationConverter();
        jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(jwtGrantedAuthoritiesConverter);
        return new ReactiveJwtAuthenticationConverterAdapter(jwtAuthenticationConverter);
    }
}

```

##### WhiteListConfig

```
@Data
@Configuration
@ConfigurationProperties(prefix = "whitelist")
public class WhiteListConfig {

    private List<String> urls;

}
```

##### AuthorizationManager

```
@Component
@Slf4j
public class AuthorizationManager implements ReactiveAuthorizationManager<AuthorizationContext> {

    @Autowired
    private RedisTemplate redisTemplate;

    @Override
    public Mono<AuthorizationDecision> check(Mono<Authentication> mono, AuthorizationContext authorizationContext) {

        ServerHttpRequest request = authorizationContext.getExchange().getRequest();
        String restPath = request.getMethodValue() + "_" + request.getURI().getPath();
        log.info("请求路径：{}", restPath);
        PathMatcher pathMatcher = new AntPathMatcher();
        // 对应跨域的预检请求直接放行
        if (request.getMethod() == HttpMethod.OPTIONS) {
            return Mono.just(new AuthorizationDecision(true));
        }

//        // 非管理端路径无需鉴权直接放行
//        if (!pathMatcher.match(AuthConstants.ADMIN_URL_PATTERN, restPath)) {
//            log.info("请求无需鉴权，请求路径：{}", restPath);
//            return Mono.just(new AuthorizationDecision(true));
//        }

        // 从缓存取资源权限角色关系列表
        Map<Object, Object> permissionRoles = redisTemplate.opsForHash().entries(RedisKeys.RESOURCE_ROLES_MAP);
        Iterator<Object> iterator = permissionRoles.keySet().iterator();
        // 请求路径匹配到的资源需要的角色权限集合authorities统计
        Set<String> authorities = new HashSet<>();
        while (iterator.hasNext()) {
            String pattern = (String) iterator.next();
            if (pathMatcher.match(pattern, request.getURI().getPath())) {
                authorities.addAll(Convert.toList(String.class, permissionRoles.get(pattern)));
            }
        }

        Mono<AuthorizationDecision> authorizationDecisionMono = mono
                .filter(Authentication::isAuthenticated)
                .flatMapIterable(Authentication::getAuthorities)
                .map(GrantedAuthority::getAuthority)
                .any(roleId -> {
                    // roleId是请求用户的角色(格式:ROLE_{roleId})，authorities是请求资源所需要角色的集合
                    log.info("访问路径：{}", request.getURI().getPath());
                    log.info("用户角色：{}", roleId);
                    log.info("资源需要角色：{}", authorities);
                    return authorities.contains(roleId);
                })
                .map(AuthorizationDecision::new)
                .defaultIfEmpty(new AuthorizationDecision(false));
        return authorizationDecisionMono;
    }

}
```

##### WebUtils

```

public class WebUtils {

    public static Mono writeFailedToResponse(ServerHttpResponse response, ResultCode resultCode){
        response.setStatusCode(HttpStatus.OK);
        response.getHeaders().set(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);
        response.getHeaders().set("Access-Control-Allow-Origin", "*");
        response.getHeaders().set("Cache-Control", "no-cache");
        String body = JSONUtil.toJsonStr(Result.failed(resultCode));
        DataBuffer buffer = response.bufferFactory().wrap(body.getBytes(Charset.forName("UTF-8")));
        return response.writeWith(Mono.just(buffer))
                .doOnError(error -> DataBufferUtils.release(buffer));
    }

}
```

##### 启动类

```

@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

#### 测试

启动nacos、sys、auth和gateway服务。

检查redis

![image-20210520115808394](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210520115808394.png)

不携带token访问接口ip:9999/sys/userAccount/username/jack,提示token无效或过期

![image-20210520115909741](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210520115909741.png)

获取token

![image-20210520115957591](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210520115957591.png)

拿access_token再次访问接口ip:9999/sys/userAccount/username/jack

![image-20210520120058738](https://gitee.com/ruocy/image_repo/raw/master/images/image-20210520120058738.png)

### 总结

到此，SpringCloud整合OAuth和JWT就完毕了。其中里面有很多理论性的东西，我也不是很明白，只能等后续弄清楚再继续了。

### 参考链接

[[Spring Cloud Security：Oauth2使用入门](http://www.macrozheng.com/#/cloud/oauth2?id=spring-cloud-security：oauth2使用入门)]

[[Spring Cloud实战 | 第六篇：Spring Cloud Gateway + Spring Security OAuth2 + JWT实现微服务统一认证鉴权](https://www.cnblogs.com/haoxianrui/p/13719356.html)]

