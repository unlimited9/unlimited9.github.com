# OAuth : Open Authorization, Open Authentication

## authorization server

#### add dependencies : gradle
```
...
implementation 'org.springframework.boot:spring-boot-starter-security'
...
implementation 'org.springframework.security.oauth:spring-security-oauth2:2.4.0.RELEASE'
implementation 'org.springframework.security:spring-security-jwt:1.1.0.RELEASE'
...
```

#### spring oauth2 configuration
`AuthorizationServerConfigurator extends AuthorizationServerConfigurerAdapter`  
```
package com.mobon.context.config.security;

import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.token.store.JdbcTokenStore;

import javax.sql.DataSource;

@Configuration
@EnableAuthorizationServer
@RequiredArgsConstructor
public class AuthorizationServerConfigurator extends AuthorizationServerConfigurerAdapter {

	private final DataSource jdbcDataSource;
	private final PasswordEncoder passwordEncoder;
	private final UserDetailsService userDetailService;

	//@Value("${security.oauth2.jwt.signkey}")
	private String signKey = "mobon2.0";

	//clients
	public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
		clients.jdbc(jdbcDataSource).passwordEncoder(passwordEncoder);
//		insert into oauth_client_details(client_id, resource_ids,client_secret,scope,authorized_grant_types,web_server_redirect_uri,authorities,access_token_validity,refresh_token_validity,additional_information,autoapprove)
//		values('mobon.service.product.server',null,'{bcrypt}$2a$10$R5RXa.7mlf6qX/93iX3lU.aMcCxm59PBWqhXf2eNxi7Fj.si3k.yy','read,write','authorization_code,refresh_token','http://dev.hajimaro.com:18070/auth/autho/callback','ROLE_USER',36000,50000,null,null);

/*
		clients.inMemory()
				.withClient("mobon.service.product.server")
				.secret(passwordEncoder.encode("P@ssw0rd")) //{bcrypt}$2a$10$R5RXa.7mlf6qX/93iX3lU.aMcCxm59PBWqhXf2eNxi7Fj.si3k.yy
				.redirectUris("http://dev.hajimaro.com:18070/auth/autho/callback")
				.authorizedGrantTypes("authorization_code")
				.scopes("read", "write")
				.accessTokenValiditySeconds(30000);
*/
	}

	//tokens
	@Override
	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
		endpoints.tokenStore(new JdbcTokenStore(jdbcDataSource)).userDetailsService(userDetailService);
	}
/*
	//jwt access token
	@Override
	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
		super.configure(endpoints);
		endpoints.accessTokenConverter(jwtAccessTokenConverter()).userDetailsService(userDetailService);
	}

	@Bean
	public JwtAccessTokenConverter jwtAccessTokenConverter() {
		JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
		converter.setSigningKey(signKey);
		return converter;
	}
*/

	@Override
	public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
		security.tokenKeyAccess("permitAll()")
				.checkTokenAccess("isAuthenticated()"); //allow check token
//				.allowFormAuthenticationForClients();
	}
}
```
>#### client details
>`redirectUri` : 인증 완료 후 이동할 클라이언트 웹 페이지 주소(with code parameter)
>
>`authorizedGrantTypes` : 주로 authorization_code 사용
>- Authorization Code  
>1.Service Provider가 제공하는 인증 화면에 로그인  
>2.클라이언트가 요청하는 리소스 접근 요청을 승인  
>3.redirect_uri로 code를 전달  
>4.access_token을 발급
>- Implicit  
>1.Service Provider가 제공하는 인증 화면에 로그인  
>2.클라이언트가 요청하는 리소스 접근 요청을 승인  
>3.redirect_uri로 access_token을 직접 전달 (Authorization Code Type과 다른점)
>- Password Credential  
>1.Resource Owner가 직접 Client에 아이디와 패스워드를 입력(Client에 아이디 패스워드가 노출)  
>2.Authorization 서버에 해당 정보로 인증
>3.access_token을 직접 획득
>- client credential : server to server에 주로 사용   
>1.인증 key(secret)로 요청해 access_token을 획득
>
>`scopes` : 인증된 accessToken으로 접근할 수 있는 리소스의 범위 resource서버(api서버)에서는 scope 정보로 클라이언트에게 제공할 리소스를 제한하거나 노출
>
>`accessTokenValiditySeconds` : 발급된 accessToken의 유효시간(sec)
>

`UserDetailsService implements org.springframework.security.core.userdetails.UserDetailsService`  
```
package com.mobon.context.config.security;

import com.mobon.service.security.jpa.entity.Account;
import com.mobon.service.security.jpa.repository.AccountJpaRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.AccountStatusUserDetailsChecker;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class UserDetailsService implements org.springframework.security.core.userdetails.UserDetailsService {
    private final AccountJpaRepository accountJpaRepository;
    private final AccountStatusUserDetailsChecker detailsChecker = new AccountStatusUserDetailsChecker();

    @Override
    public UserDetails loadUserByUsername(String name) {
        Account account = accountJpaRepository.findByUid(name).orElseThrow(() -> new UsernameNotFoundException("user is not exists"));
        detailsChecker.check(account);
        return account;
    }
}
```

#### spring security configuration

`WebSecurityConfigurator extends WebSecurityConfigurerAdapter`  
```
package com.mobon.context.config.security;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
@EnableWebSecurity
@Slf4j
public class WebSecurityConfigurator extends org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter {

	@Autowired
	private com.mobon.context.config.security.AuthenticationProvider authenticationProvider;

	@Override
	public void configure(WebSecurity web) throws Exception {
		web.ignoring().antMatchers("/resource/**","/static/**","/h2console/**");
		//web.ignoring().antMatchers("/resource/**","/static/**","/h2console/**","/dspt/admin/**","/dspt/securtiy/group/**");
	}
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		//http.csrf().ignoringAntMatchers("/h2console/**"); // h2 console 사용을 위한 설정
		http.headers()
				.frameOptions().sameOrigin();
//		http.anonymous()
//				.principal("guest").authorities("ROLE_GUEST");
		http.authorizeRequests()
				//.antMatchers("/resources/**","/static/**","/h2console/**", "**"+AUTHE_PATH+"/**").permitAll() //해당 url을 허용한다.
				.antMatchers("/oauth/**","/autho/callback").permitAll() //해당 url을 허용한다.
				//.antMatchers("/admin/**").hasAuthority("ROLE_ADMIN") //admin 폴더에 경우 admin 권한이 있는 사용자에게만 허용
				.antMatchers("/status/**").authenticated(); // /dspt/status
//				.anyRequest().authenticated();
		http.formLogin()
				.permitAll();
		http.logout()
				.permitAll();
		http.csrf().disable();
	}

	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth.authenticationProvider(authenticationProvider);
/*
		PasswordEncoder encoder = passwordEncoder();
		auth.inMemoryAuthentication()
				.withUser("user")
				.password(encoder.encode("password"))
				.roles("USER")
				.and()
				.withUser("admin")
				.password(encoder.encode("admin"))
				.roles("USER", "ADMIN");
*/
	}

	@Bean
	public PasswordEncoder passwordEncoder(){
		return PasswordEncoderFactories.createDelegatingPasswordEncoder(); //default : "bcrypt" : new BCryptPasswordEncoder();
		//return org.springframework.security.crypto.password.NoOpPasswordEncoder.getInstance(); // "noop"
		//return new org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder(); // "bcrypt"
	}
}
```

`AuthenticationProvider implements org.springframework.security.authentication.AuthenticationProvider`  
```
package com.mobon.context.config.security;

import com.mobon.service.security.jpa.entity.Account;
import com.mobon.service.security.jpa.repository.AccountJpaRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Component;

@Component
@Slf4j
public class AuthenticationProvider implements org.springframework.security.authentication.AuthenticationProvider {
    @Autowired
    private PasswordEncoder passwordEncoder;

    @Autowired
    private AccountJpaRepository accountJpaRepository;

    @Override
    public Authentication authenticate(Authentication authentication) {

        String name = authentication.getName();
        String password = authentication.getCredentials().toString();

        Account user = accountJpaRepository.findByUid(name).orElseThrow(() -> new UsernameNotFoundException("user is not exists"));

        if (!passwordEncoder.matches(password, user.getPassword()))
            throw new BadCredentialsException("password is not valid");

        return new UsernamePasswordAuthenticationToken(name, password, user.getAuthorities());
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(UsernamePasswordAuthenticationToken.class);
    }

}
```

#### UserDetails JPA

`Account implements UserDetails`  
```
package com.mobon.service.security.jpa.entity;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.stream.Collectors;

@Builder
@Entity //jpa entity
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Table(name = "t_account") // 't_account' 테이블과 매핑됨을 명시
public class Account implements UserDetails {
    @Id // pk
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long msrl;
    @Column(nullable = false, unique = true, length = 50)
    private String uid;
    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Column(length = 100)
    private String password;
    @Column(nullable = false, length = 100)
    private String name;
    @Column(length = 100)
    private String provider;

    @ElementCollection(fetch = FetchType.EAGER)
    @Builder.Default
    private List<String> groups = new ArrayList<>();

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return this.groups.stream().map(SimpleGrantedAuthority::new).collect(Collectors.toList());
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public String getUsername() {
        return this.uid;
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

`AccountJpaRepository extends JpaRepository`  
```
package com.mobon.service.security.jpa.repository;

import com.mobon.service.security.jpa.entity.Account;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface AccountJpaRepository extends JpaRepository<Account, Long> {
    Optional<Account> findByUid(String email);
}
```

#### spring oauth2 controller
`AuthorizationController`  
```
package com.mobon.service.security.controller;

import com.mobon.context.constant.ContextConfiguration;
import com.mobon.service.security.jpa.entity.Account;
import com.mobon.service.security.jpa.repository.AccountJpaRepository;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.codec.binary.Base64;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Controller;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Collections;
import java.util.Map;

/**
 * @author hajimaro
 */

@Controller()
@RequestMapping(value = "/autho")
@Slf4j
public class AuthorizationController {

    @Resource(name="restTemplate")
    private RestTemplate restTemplate;

    @Autowired
    private AccountJpaRepository accountJpaRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @RequestMapping(value = "/authorize", method = {RequestMethod.GET, RequestMethod.POST}, produces = "application/json")
    public @ResponseBody Object authorize(@RequestParam Map<String, String> parameters) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        headers.add("Authorization", "Basic " + encodedCredentials(parameters.get("client_id"), parameters.get("client_secret")));

        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        params.add("client_id", parameters.get("client_id"));
        params.add("redirect_uri", String.format("%s/auth/autho/callback", ContextConfiguration.getProperty("security.authorization.oauth.server")));
        params.add("response_type", "code");
        params.add("scope", "read");
        HttpEntity<MultiValueMap<String, String>> httpEntity = new HttpEntity<>(params, headers);
        return restTemplate.postForEntity(String.format("%s/auth/oauth/authorize", ContextConfiguration.getProperty("security.authorization.oauth.server")), httpEntity, String.class);
    }
    @RequestMapping(value = "/callback", method = {RequestMethod.GET, RequestMethod.POST}, produces = "application/json")
    public @ResponseBody Object callback(@RequestParam Map<String, String> parameters) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        headers.add("Authorization", "Basic " + encodedCredentials(parameters.get("client_id"), parameters.get("client_secret")));

        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        params.add("code", parameters.get("code"));
        params.add("grant_type", "authorization_code");
        params.add("redirect_uri", String.format("%s/auth/autho/callback", ContextConfiguration.getProperty("security.authorization.oauth.server")));
        HttpEntity<MultiValueMap<String, String>> httpEntity = new HttpEntity<>(params, headers);
        return restTemplate.postForEntity(String.format("%s/auth/oauth/token", ContextConfiguration.getProperty("security.authorization.oauth.server")), httpEntity, String.class);
    }

    @RequestMapping(value = "/token", method = {RequestMethod.GET, RequestMethod.POST}, produces = "application/json")
    public @ResponseBody Object grantToken(@RequestParam Map<String, String> parameters) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        headers.add("Accept", MediaType.APPLICATION_JSON_VALUE);
        headers.add("Authorization", "Basic " + encodedCredentials(parameters.get("client_id"), parameters.get("client_secret")));

        //grant_type(client_credentials/refresh_token), refresh_token(optional)
        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        for(String key : parameters.keySet()) {
            params.add(key, parameters.get(key));
        }
        HttpEntity<MultiValueMap<String, String>> httpEntity = new HttpEntity<>(params, headers);
        ResponseEntity<String> responseEntity =  restTemplate.postForEntity(String.format("%s/auth/oauth/token", ContextConfiguration.getProperty("security.authorization.oauth.server")), httpEntity, String.class);
        log.info(responseEntity.getBody());
        return responseEntity;
    }

    @RequestMapping(value = "/encode", method = {RequestMethod.GET, RequestMethod.POST}, produces = "application/json")
    public @ResponseBody Object passwordEncode(HttpServletRequest request, HttpServletResponse response, @RequestParam String plainText) {
        return passwordEncoder.encode(plainText);
    }

    public String encodedCredentials(String clientId, String clientSecret) {
        return new String(Base64.encodeBase64(String.format("%s:%s", clientId, clientSecret).getBytes()));
    }

}

```

#### grant_type : authorize code
>http://dev.hajimaro.com:18070/auth/oauth/authorize?client_id=mobon.service.product.server&redirect_uri=http://dev.hajimaro.com:18070/auth/autho/callback&response_type=code&scope=read

#### access_token
>http://dev.hajimaro.com:18070/auth/autho/callback?code=Dq4WfT&client_id=mobon.service.product.server&client_secret=password
```
{"access_token":"d5d1d9df-5615-4894-97fc-88a9603f34ad","token_type":"bearer","refresh_token":"28096409-efa5-488f-93b1-22bb515f0fd9","expires_in":32690,"scope":"read"}
```

#### grant_type : client credentials
>http://dev.hajimaro.com:18070/auth/autho/token?grant_type=client_credentials&client_id=mobon.service.product.server&client_secret=password

#### grant_type : refresh token
>http://127.0.0.1:18070/auth/autho/token?grant_type=refresh_token&refresh_token=a-b-c-d&client_id=mobon.service.product.server&client_secret=password  


## resource server

#### add dependencies : gradle
```
...
implementation 'org.springframework.boot:spring-boot-starter-security'
...
implementation 'org.springframework.security.oauth:spring-security-oauth2:2.4.0.RELEASE'
implementation 'org.springframework.security:spring-security-jwt:1.1.0.RELEASE'
...
```

#### spring oauth2 configuration
`AuthorizationServerConfigurator extends ResourceServerConfigurerAdapter`  
```
package com.mobon.context.config.security;

import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.token.RemoteTokenServices;
import org.springframework.security.oauth2.provider.token.store.JdbcTokenStore;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.sql.DataSource;

@Configuration
@EnableResourceServer
public class AuthorizationServerConfigurator extends ResourceServerConfigurerAdapter {

	@Value("${spring.security.oauth2.client.registration.api.client-id}")
	private String clientId;
	@Value("${spring.security.oauth2.client.registration.api.client-secret}")
	private String clientSecret;
	@Value("${spring.security.oauth2.client.registration.api.provider}")
	private String checkTokenEndpointUrl;

	@Override
	public void configure(HttpSecurity http) throws Exception {
		http.headers().frameOptions().disable();
		http.authorizeRequests()
				.antMatchers("/product/**","/inclination/**").access("#oauth2.hasScope('read')")
				.anyRequest().authenticated();
	}

	@Override
	public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
		resources.tokenServices(tokenServices()).resourceId("gateway.api.common");
	}

	@Primary
	@Bean
	public RemoteTokenServices tokenServices() {
		RemoteTokenServices tokenService = new RemoteTokenServices();
		tokenService.setCheckTokenEndpointUrl(checkTokenEndpointUrl);
		tokenService.setClientId(clientId);
		tokenService.setClientSecret(clientSecret);
		return tokenService;
	}

/*
	@Bean
	@Primary
	public DefaultTokenServices tokenServices() {
		DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
		defaultTokenServices.setTokenStore(tokenStore());
		return defaultTokenServices;
	}
	@Bean
	public TokenStore tokenStore() {
		return new JdbcTokenStore(dataSource());
	}
*/

}
```

#### spring security configuration

`WebSecurityConfigurator extends WebSecurityConfigurerAdapter`  
```
package com.mobon.context.config.security;

import javax.annotation.Resource;
import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;

import lombok.extern.slf4j.Slf4j;

@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
@EnableWebSecurity
@Slf4j
public class WebSecurityConfigurator extends org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter {

	@Override
	public void configure(WebSecurity web) throws Exception {
		web.ignoring().antMatchers("/resource/**","/static/**","/h2console/**");
		//web.ignoring().antMatchers("/resource/**","/static/**","/h2console/**","/dspt/admin/**","/dspt/securtiy/group/**");
	}
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		//http.csrf().ignoringAntMatchers("/h2console/**"); // h2 console 사용을 위한 설정
		http.headers()
				.frameOptions().sameOrigin();
//		http.anonymous()
//				.principal("guest").authorities("ROLE_GUEST");
		http.authorizeRequests()
				//.antMatchers("/resources/**","/static/**","/h2console/**", "**"+AUTHE_PATH+"/**").permitAll() //해당 url을 허용한다.
				//.antMatchers("/admin/**").hasAuthority("ROLE_ADMIN") //admin 폴더에 경우 admin 권한이 있는 사용자에게만 허용
				.antMatchers("/status/**").authenticated(); // /dspt/status
//				.anyRequest().authenticated();
		http.formLogin()
				.permitAll();
		http.logout()
				.permitAll();
		http.csrf().disable();
	}
	
	@Bean
	public PasswordEncoder passwordEncoder(){
		return NoOpPasswordEncoder.getInstance();
		//return PasswordEncoderFactories.createDelegatingPasswordEncoder();
		//return new BCryptPasswordEncoder();
	}

}
```

## 9. Appendix

#### reference site

* OAuth 2 Developers Guide  
https://projects.spring.io/spring-security-oauth/docs/oauth2.html

* Using JWT with Spring Security OAuth  
https://www.baeldung.com/spring-security-oauth-jwt

* Microsoft id 플랫폼 및 OAuth 2.0 클라이언트 자격 증명 흐름  
https://docs.microsoft.com/ko-kr/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow

+ Spring Security OAuth2구현  
https://minwan1.github.io/2018/02/24/2018-03-11-Spring-OAuth%EA%B5%AC%ED%98%84/

+ Spring Boot로 만드는 OAuth2 시스템 1  
https://brunch.co.kr/@sbcoba/1

+ OAuth 2를 이용한 SSO 환경 구축 (1/2)  
http://www.nextree.co.kr/oauth-2reul-iyonghan-sso-hwangyeong-gucug-1-2/

+ REST api에 OAuth2.0 구현  
http://jeonghwan-kim.github.io/oauth2-0-in-rest-api/

+ 14. 스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현하기 (1)
  https://for1123.tistory.com/16?category=779440

+ SpringBoot2로 Oauth2 서버 만들기  
https://daddyprogrammer.org/post/series/spring-boot-oauth2/

+ Spring Boot 2 – OAuth2 Auth and Resource Server  
https://howtodoinjava.com/spring-boot2/oauth2-auth-server/

+ [처음 배우는 스프링 부트2] 5. 스프링 부트 시큐리티 + OAuth2  
https://freedeveloper.tistory.com/24

+ Spring MSA (4) - 인증서비스  
https://taes-k.github.io/2019/06/20/spring-msa-4/

+ aop를 이용한 oauth2 캐시 적용하기  
https://woowabros.github.io/experience/2019/03/05/aop-oauth2-redis.html

+ SpringSecurity - Kakao OAuth2 Client 사용하기  
https://galid1.tistory.com/582

+ OAuth2 Authentication  
https://mailchimp.com/developer/guides/how-to-use-oauth2/

+ Spring Boot + OAuth 2 Password Grant - Hello World Example  
https://www.javainuse.com/spring/springboot-oauth2-password-grant

+ Using Spring Security 5 to integrate with OAuth 2-secured services such as Facebook and GitHub  
https://spring.io/blog/2018/03/06/using-spring-security-5-to-integrate-with-oauth-2-secured-services-such-as-facebook-and-github

+ OAUTH - Client Credentials  
https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/

+ OAuth2 인증 방식 정리  
https://cheese10yun.github.io/oauth2/

+ [oAuth_이론] 5. oAuth가 작동되는 과정_API_Refresh token [5/5]  
https://velog.io/@rohkorea86/oAuth%EC%9D%B4%EB%A1%A0-5.-oAuth%EA%B0%80-%EC%9E%91%EB%8F%99%EB%90%98%EB%8A%94-%EA%B3%BC%EC%A0%95APIRefresh-token-55
