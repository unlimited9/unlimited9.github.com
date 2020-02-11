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

#### spring security configuration

`WebSecurityConfigurator extends WebSecurityConfigurerAdapter`  
```
package com.mobon.context.config.security;

import lombok.extern.slf4j.Slf4j;
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
		PasswordEncoder encoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
		auth.inMemoryAuthentication()
				.withUser("user")
				.password(encoder.encode("password"))
				.roles("USER")
				.and()
				.withUser("admin")
				.password(encoder.encode("admin"))
				.roles("USER", "ADMIN");
	}

  @Bean
	public PasswordEncoder passwordEncoder(){
		return PasswordEncoderFactories.createDelegatingPasswordEncoder(); //default : "bcrypt" : new BCryptPasswordEncoder();
		//return org.springframework.security.crypto.password.NoOpPasswordEncoder.getInstance(); // "noop"
		//return new org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder(); // "bcrypt"
	}
}
```

#### spring oauth2 controller
`AuthorizationController`  
```
package com.mobon.service.application.controller;

import lombok.extern.slf4j.Slf4j;
import org.apache.commons.codec.binary.Base64;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
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

@Controller()
@RequestMapping(value = "/autho")
@Slf4j
public class AuthorizationController {

    @Resource(name="restTemplate")
    private RestTemplate restTemplate;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @RequestMapping(value = "/callback", method = {RequestMethod.GET, RequestMethod.POST}, produces = "application/json")
    public @ResponseBody Object callback(HttpServletRequest request, HttpServletResponse response, @RequestParam String code) {
        String credentials = "mobon.service.product.server:P@ssw0rd";

        String encodedCredentials = new String(Base64.encodeBase64(credentials.getBytes()));

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        headers.add("Authorization", "Basic " + encodedCredentials);

        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        params.add("code", code);
        params.add("grant_type", "authorization_code");
        params.add("redirect_uri", "http://dev.hajimaro.com:18070/auth/autho/callback");
        HttpEntity<MultiValueMap<String, String>> httpEntity = new HttpEntity<>(params, headers);
        return restTemplate.postForEntity("http://dev.hajimaro.com:18070/auth/oauth/token", httpEntity, String.class);
    }

    @RequestMapping(value = "/encode", method = {RequestMethod.GET, RequestMethod.POST}, produces = "application/json")
    public @ResponseBody Object passwordEncode(HttpServletRequest request, HttpServletResponse response, @RequestParam String plainText) {
        return passwordEncoder.encode(plainText);
    }

}
```

#### access_token 발급
>http://dev.hajimaro.com:18070/auth/oauth/authorize?client_id=mobon.service.product.server&redirect_uri=http://dev.hajimaro.com:18070/auth/autho/callback&response_type=code&scope=read

#### access_token 발급
>http://dev.hajimaro.com:18070/auth/autho/token/refresh?refreshToken=

## 9. Appendix

#### reference site

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
