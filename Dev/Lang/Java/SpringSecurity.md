# SpringSecurity Login

여러 강의와 프로젝트를 진행하면서 만들었던 로그인 기능 하지만 막상 설명하라고 하면 잘 설명을못하고 기존의 코드를 복붙해서 사용했다 그래서 오늘 자세히 파악해볼려고한다.

<img src='/img/springSecurityImg.png'>

## JwtTokenFilter

```java
package com.yourLuck.yourLuck.configuration.Filter;

import com.yourLuck.yourLuck.model.User;
import com.yourLuck.yourLuck.service.UserService;
import com.yourLuck.yourLuck.util.JwtTokenUtils;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpHeaders;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@Slf4j
@RequiredArgsConstructor
public class JwtTokenFilter extends OncePerRequestFilter {

    private final String key;
    private final UserService userService;
    private final static List<String> TOKEN_IN_PARAM_URLS = List.of("/api/v1/users/login", "/api/v1/users/join");

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        final String token;
        try {
            if (TOKEN_IN_PARAM_URLS.contains(request.getRequestURI())) {
                filterChain.doFilter(request, response);

                return;
            }else{
                final String header = request.getHeader(HttpHeaders.AUTHORIZATION);
                log.info("AUTHRIZATION!!! : {}", header);
                if(header == null || !header.startsWith("Bearer ")){
                    throw new RuntimeException("INVALID / MISSING HEADER");
                }
                token = header.split(" ")[1].trim();
            }
            if(JwtTokenUtils.isExpired(token,key)){
                throw new RuntimeException("Expired Key");
            }
            String userName = JwtTokenUtils.getUserName(token, key);
            User user = userService.loadUserByUserName(userName);

            UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
                    user, null, user.getAuthorities());

            authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
            SecurityContextHolder.getContext().setAuthentication(authentication);
        } catch(RuntimeException e) {
            log.error("Error while validating. {}", e.toString());
            filterChain.doFilter(request,response);
            return;
        }
        filterChain.doFilter(request, response);
    }
}

```

기본 정의 : JwtTokenFilter 는 HTTP 요청을 인터셉트하여 헤더에서 JWT를 추출하고, 이를 검증하는 필터입니다. 이 필터는 보통 스프링 시큐리티의 필터 체인에 추가되어, 요청이 서버의 다른 부분에 도달하기 전에. 실행됩니다. 요청 헤더에서 JWT를 읽어 JwtTokenUtils 를 사용하여 검증하며, 토큰이 유효하면 요청에 대한 사용자 인증을 수행합니다

나의 코드 : 
****1. `TOKEN_IN_PARAM_URLS` 리스트를 사용하여 특정 URL 경로에 대해서는 토큰 검증을 건너뛰게 설정하였습니다. 예를 들어, 로그인이나 회원가입과 같은 경로에서는 JWT 검증을 수행하지 않습니다.

2. 요청 헤더에서 "Bearer " 접두사를 가진 토큰을 추출합니다. 그리고 해당 토큰이 유효한지, 만료되지 않았는지를 `JwtTokenUtils.isExpired` 메소드를 통해 검증합니다.

3. 토큰이 유효하다면, `JwtTokenUtils.getUserName` 메소드를 사용하여 토큰에서 사용자 이름을 추출하고, 이를 이용해 `UserService`를 통해 해당 사용자 정보를 불러옵니다.

4.마지막으로, 인증된 사용자 정보를 바탕으로 `UsernamePasswordAuthenticationToken`을 생성하여 스프링 시큐리티의 `SecurityContextHolder`에 설정함으로써, 이후 요청 처리 과정에서 해당 사용자가 인증된 사용자로 처리될 수 있도록 합니다.

코드를 보면 저는 [OncePerRequestFilter](https://www.notion.so/OncePerRequestFilter-128edf0fe7c442c79f8cf4a221978fc7?pvs=21) 를 extends 하고있으며 요청당 한 번씩 실행되는 필터를 만듭니다. 이는 모든 HTTP 요청에 대해 특정 로직(위 코드에서는 JWT 인증)을 실행하고자 할 때 사용합니다..

### doFilterInternal 메소드

- 요청의 URI가 로그인 또는 회원가입과 관련된 경우, 필터 체인을 계속 진행합니다(`filterChain.doFilter(request, response);`). 이는 인증이 필요 없는 엔드포인트를 처리하기 위함입니다.
- 그렇지 않은 경우, 요청 헤더에서 "Bearer "로 시작하는 `Authorization` 값을 추출합니다. 이 값이 없거나 형식에 맞지 않으면 예외를 발생시킵니다.
- 토큰이 만료되었는지 검사합니다. 만료된 경우 예외를 발생시킵니다.
- 토큰이 유효한 경우, 토큰으로부터 사용자 이름을 추출하고, 해당 사용자 정보를 `UserService`를 통해 로드합니다.
- `UsernamePasswordAuthenticationToken`을 생성하여, `SecurityContextHolder`에 설정함으로써, Spring Security 컨텍스트에 인증 정보를 등록합니다. 이는 다음 필터 또는 요청 처리 과정에서 현재 사용자가 인증되었다는 것을 알 수 있게 합니다.

## AuthenticationConfig

```java
package com.yourLuck.yourLuck.configuration;

import com.yourLuck.yourLuck.configuration.Filter.JwtTokenFilter;
import com.yourLuck.yourLuck.exception.CustomAuthenticationEntryPoint;
import com.yourLuck.yourLuck.service.UserService;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class AuthenticationConfig extends WebSecurityConfigurerAdapter {

    private final UserService userService;

    @Value("${jwt.token.secret}")
    private String secretKey;
    @Value("${jwt.refresh-expired}")
    private long refreshExpiredTimeMs;
    @Value("${jwt.access-expired}")
    private long accessExpiredTimeMs;

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().regexMatchers("^(?!/api/).*")
                .antMatchers(HttpMethod.POST,"/api/*/users/join","/api/*/users/login");
    }
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                .antMatchers("/api/**").authenticated()
                .and()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .addFilterBefore(new JwtTokenFilter(secretKey,userService), UsernamePasswordAuthenticationFilter.class)
                .exceptionHandling()
                .authenticationEntryPoint(new CustomAuthenticationEntryPoint())
        ;
    }
}

```

**이 코드는 Spring Security 를 사용하여 인증과 인가를 구현한 설정 클래스입니다.** 

`@EnableWebSecurity` 애너테이션은 Spring Security 설정을 활성화시키며, 웹 보안을 위한 여러 기본 설정을 자동으로 해줍니다.

`web.ignoring().regexMatchers("^(?!/api/).*")`는 `/api/`로 시작하지 않는 모든 요청에 대해 Spring Security 보안을 적용하지 않도록 설정합니다.

`.antMatchers(HttpMethod.POST,"/api/*/users/join","/api/*/users/login")`는 사용자 가입과 로그인 API에 대해서도 보안을 적용하지 않습니다.

### configure(HttpSecurity http)

- `http.csrf().disable()`는 CSRF 보호 기능을 비활성화합니다. REST API를 구현할 때는 일반적으로 CSRF 보호를 비활성화합니다.
- `.authorizeRequests().antMatchers("/api/**").authenticated()`는 `/api/`로 시작하는 모든 요청에 대해 인증을 요구합니다.
- `.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)`는 세션을 사용하지 않고, 매 요청마다 토큰을 확인하여 인증을 처리하도록 설정합니다. 이는 JWT 토큰 기반 인증에 적합합니다.
- `.addFilterBefore(new JwtTokenFilter(secretKey,userService), UsernamePasswordAuthenticationFilter.class)`는 `UsernamePasswordAuthenticationFilter` 전에 `JwtTokenFilter`를 추가합니다. 이 필터에서는 HTTP 요청의 헤더에서 JWT 토큰을 읽고 검증합니다.
- `.exceptionHandling().authenticationEntryPoint(new CustomAuthenticationEntryPoint())`는 인증 과정에서 예외가 발생했을 때, `CustomAuthenticationEntryPoint`를 통해 예외 처리를 합니다.

궁금증 : 

이 두가지의 설명을보면 한가지는 보안을 적용하지않고 한가지는 인증을 요구하는데 두개가 같이있는게 괜찮을걸까?

> **web.ignoring().regexMatchers("^(?!/api/).*")는 /api/로 시작하지 않는 모든 요청에 대해 Spring Security 보안을 적용하지 않도록 설정합니다..authorizeRequests().antMatchers("/api/**").authenticated()는 /api/로 시작하는 모든 요청에 대해 인증을 요구합니다.**
> 

**보안을 적용하지 않는 설정** (`web.ignoring().regexMatchers("^(?!/api/).*")`): 이 설정은 /api/로 시작하지 않는 모든 요청에 대해서 Spring Security 의 보안 처리를 거치지 않도록 설정합니다. 즉, /api/ 경로와 관련없는 정적 자원(image, css, js 파일등..)이나, 특정 페이지에 대한 요청 등을 처리 할 때 사용할 수 있습니다. 이러한 요청들은 인증 처리 없이 접근을 허용해야 할 수도 있기 때문에, 보안 처리를 무시하는것이 합리적입니다.

**인증을 요구하는 설정** (`.authorizeRequests().antMatchers("/api/**").authenticated()`): 반면, 이 설정은 `/api/`로 시작하는 모든 요청에 대해 사용자 인증을 요구합니다. 이는 애플리케이션의 API에 대한 접근을 보호하기 위한 것으로, 유효한 인증 정보(예: JWT 토큰) 없이는 접근할 수 없도록 합니다. API를 통해 처리되는 데이터나 기능은 민감할 수 있으며, 인증된 사용자만이 접근할 수 있도록 제한하는 것이 보안상 중요합니다.

이 두 설정이 함께 존재함으로써, 개발자는 애플리케이션의 보안 요구 사항에 따라 유연하게 보안 정책을 구성할 수 있습니다. `/api/` 경로를 통한 요청은 보안 처리를 엄격하게 적용하면서도, 그 외의 경로에 대해서는 보다 느슨한 보안 정책을 적용할 수 있습니다. 이런 방식은 애플리케이션의 성능 최적화

## JwtTokenUtils

```java
package com.yourLuck.yourLuck.util;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.security.Keys;

import java.nio.charset.StandardCharsets;
import java.security.Key;
import java.util.Date;

public class JwtTokenUtils {

    public static String getUserName(String token, String key) {
        return extracClaims(token,key).get("userName",String.class);
    }

    public static boolean isExpired(String token, String key) {
        Date expireDate = extracClaims(token,key).getExpiration();
        return expireDate.before(new Date());
    }

    private static Claims extracClaims(String token, String key) {
        return Jwts.parserBuilder().setSigningKey(getKey(key))
                .build().parseClaimsJws(token).getBody();
    }

    public static String generateToken(String userName, String key, long expiredTimeMs) {
        Claims claims = Jwts.claims();
        claims.put("userName" ,userName);

        return Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + expiredTimeMs))
                .signWith(getKey(key), SignatureAlgorithm.HS256)
                .compact();
    }

    public static Key getKey(String key){
        byte[] keyBytes = key.getBytes(StandardCharsets.UTF_8);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}

```

이 코드는 JWT(Json Web Token)를 생성하고 검증하는 데 사용되는 유틸리티 클래스인 `JwtTokenUtils`를 정의합니다. JWT는 웹 표준으로, 두 당사자 사이에서 JSON 객체를 사용하여 안전하게 정보를 전송하기 위해 디자인되었습니다. 이 클래스는 JWT를 생성, 파싱, 검증하는 기능을 제공합니다.

### 메소드설명

• getUserName(String token, String key): 주어진 JWT 토큰과 비밀키를 사용하여 토큰에서 `userName`을 추출합니다. `extracClaims` 메소드를 통해 토큰의 내용(Claims)을 가져온 후, `userName`에 해당하는 값을 반환합니다.

• isExpired(String token, String key): 토큰이 만료되었는지 여부를 반환합니다. `extracClaims`를 사용하여 토큰의 만료 시간을 가져온 다음, 현재 시간과 비교하여 만료 여부를 확인합니다.

• extracClaims(String token, String key): 비공개 메소드로, 주어진 토큰과 비밀키를 사용하여 토큰에서 Claims(토큰의 내용)을 추출합니다. `Jwts.parserBuilder()`를 사용하여 파서를 생성하고, `setSigningKey` 메소드로 비밀키를 설정한 후 토큰을 파싱합니다. 마지막으로 토큰의 본문(Body)을 반환합니다.

• generateToken(String userName, String key, long expiredTimeMs): 사용자 이름, 비밀키, 그리고 토큰의 만료 시간(밀리초 단위)를 입력받아 새로운 JWT를 생성합니다. `Jwts.claims()`을 사용해 Claims 객체를 생성하고, 사용자 이름을 저장합니다. 그 후, 현재 시간과 만료 시간을 설정하고, `signWith` 메소드를 사용하여 비밀키와 함께 토큰에 서명합니다. 마지막으로 `compact` 메소드를 통해 생성된 토큰을 문자열로 반환합니다.

• getKey(String key): 문자열 형태의 비밀키를 바이트 배열로 변환한 후, `Keys.hmacShaKeyFor` 메소드를 사용하여 `Key` 객체로 변환합니다. 이 키는 JWT를 서명할 때 사용됩니다.

### JWT 처리과정

**1. 토큰 생성:** `generateToken` 메소드는 사용자 정보와 만료 시간을 포함하는 JWT를 생성합니다. 이 토큰은 서버에서 클라이언트로 전달됩니다.
**2. 토큰 검증 및 사용**: 클라이언트가 서버에 요청을 보낼 때 JWT를 포함시키면, 서버는 `getUserName` 또는 `isExpired` 메소드를 사용하여 토큰의 유효성을 검증하고, 토큰에 포함된 정보(예: 사용자 이름)를 사용할 수 있습니다.
이 클래스를 통해 JWT를 손쉽게 관리하고, 웹 어플리케이션에서 사용자 인증 및 정보 전송의 **보안을 강화할 수 있습니다.**

**Claims** 는 JWT(JSON Web Token) 에서 사용되는 용어로, 토큰에 담겨 있는 정보의 한 조각을 의미합니다. JWT 는 일반적으로 사용자 인증 정보 및 기타 필요한 정보를 안전하게 전송하기 위해 사용되며, 이 정보들은 Claims라는 이름의 key-value 쌍으로 구성됩니다. 각각의 Claim은 토큰의 사용자 또는 관련정보(ex: username, 권한, 토큰유효기간등.)를 나타냅니다.

`JwtTokenUtils` 클래스에서는 `io.jsonwebtoken.Claims`를 사용하여 JWT를 생성하고, 특정 토큰에서 정보를 추출하는 데 사용됩니다. 예를 들어:

`generateToken` 메소드는 사용자 이름과 만료 시간을 입력받아 새로운 JWT를 생성합니다. 여기서 `Claims` 객체에 사용자 이름을 `userName`이라는 키로 저장하고, 이후 토큰에 이 정보를 포함시킵니다.`getUserName`과 `isExpired` 메소드는 주어진 토큰에서 `Claims`를 추출하여, 각각 사용자 이름을 얻거나 토큰의 유효 기간이 만료되었는지를 확인합니다.

JWT의 `Claims`를 사용함으로써, 서버와 클라이언트 간에 안전하게 필요한 정보를 주고받을 수 있으며, 이를 통해 사용자 인증 및 권한 검증 등의 보안 관련 처리를 할 수 있습니다. `JwtTokenUtils` 클래스는 이러한 JWT 처리를 도와주는 유틸리티 클래스로, JWT 생성, 정보 추출, 유효성 검증 등의 기능을 제공합니다.