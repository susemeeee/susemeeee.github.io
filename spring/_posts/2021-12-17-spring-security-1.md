---
layout: post
title: Spring Security 1
categories: Spring
---

# Spring Security (이론편)

![](..\..\images\SpringSecurityLogo.png){: width="70%" height="70%"}

## Spring Security란?

Spring Security는 스프링 기반 애플리케이션의 보안(인증, 권한)을 담당하는 스프링 하위 프레임워크이다. Spring Security는 Filter 기반으로 작동해 Dispatcher Servlet보다 요청을 먼저 받아 처리하고, 다양한 보안 관련 옵션으로 일일이 보안 관련 로직을 작성하지 않아도 되는 장점이 있다.

---



## 기본 용어

- 접근 주체(principal) : 보호된 자원에 접근하는 대상
- 인증(authentication) : 애플리케이션의 작업을 수행할 수 있는 주체임을 증명
- 인가(authorize) : 인증된 주체가 애플리케이션의 작업을 수행할 수 있는지 확인

---



## Spring Security Filter Chain

![](..\..\images\filterchain.png)

Spring Security는 서블릿 필터 기반으로 작동한다. 사용자가 요청을 보내면 컨테이너는 하나의 필터 체인을 생성한다. 필터 체인에는 필터가 순서대로 저장되어 있고, 마지막에는 서블릿이 있다. 

필터 체인 내부의 필터들은 들어온 요청을 처리해 다음 순서 필터로 넘기거나 넘기지 않을 수 있다. 마지막 필터는 서블릿으로 요청을 넘기게 된다.



![](..\..\images\filterchain2.png)

Spring Security의 필터들은 스프링 컨테이너에 등록된 빈이기 때문에, 서블릿 필터에서는 이를 인식할 수 없다. 따라서, Spring Security에서는 DelegatingFilterProxy라는 서블릿 필터의 구현체를 제공한다. DelegatingFilterProxy는 서블릿 필터에 등록될 수 있고, Spring Security의 필터 빈들을 의존성 주입할 수 있다.

```java
public class DelegatingFilterProxy extends GenericFilterBean {
   ...

   @Nullable
   private volatile Filter delegate;
    
   ...
        
   	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		// Lazily initialize the delegate if necessary.
		Filter delegateToUse = this.delegate;
		if (delegateToUse == null) {
			synchronized (this.delegateMonitor) {
				delegateToUse = this.delegate;
				if (delegateToUse == null) {
					WebApplicationContext wac = findWebApplicationContext();
					if (wac == null) {
						throw new IllegalStateException("No WebApplicationContext found: " +
								"no ContextLoaderListener or DispatcherServlet registered?");
					}
					delegateToUse = initDelegate(wac);
				}
				this.delegate = delegateToUse;
			}
		}

		// Let the delegate perform the actual doFilter operation.
		invokeDelegate(delegateToUse, request, response, filterChain);
	}
    
    ...
        
   	protected void invokeDelegate(
			Filter delegate, ServletRequest request, ServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		delegate.doFilter(request, response, filterChain);
	}
```

위의 코드를 보면, 내부 필터의 doFilter()를 호출하면서 작업을 위임하는 것을 볼 수 있다.



![](..\..\images\filterchain3.png)

DelegatingFilterProxy 내부에 FilterChainProxy를 둘 수 있다. FilterChainProxy는 DelegatingFilterProxy를 통해 받은 요청을 SecurityFilterChain에 전달해 작업을 위임한다.

```java
public class FilterChainProxy extends GenericFilterBean {
  
   ...

   private List<SecurityFilterChain> filterChains;
    
   ...
```

필터 체인을 List로 관리하기 때문에 여러 개의 필터체인이 존재할 수 있다.



![](..\..\images\filterchain4.png)

SecurityFilterChain은 여러 개의 Spring Security 필터가 담겨 있다. FilterChainProxy에서 위임받은 작업을 어떤 필터에서 수행할지 결정하는 역할을 한다.

![](..\..\images\filterchain5.png)

여러 개의 필터 체인을 등록해 URL에 따라 다른 필터 체인이 사용되도록 할 수도 있다.

---



## Spring Security Filter

![](..\..\images\securityfilterchain.png)

위에서 언급된 SecurityFilterChain은 여러 개의 필터들이 순서대로 연결되어 있다.

- SecurityContextPersistenceFilter : SecurityContextRepository에서 SecurityContext를 가져오거나 저장한다.
- LogoutFilter : 설정된 로그아웃 URL로 오는 요청에 대해 유저를 로그아웃 처리한다.
- UsernamePasswordAuthenticationFilter : 아이디, 비밀번호를 사용하는 form 기반의 유저 인증을 처리한다.
- DefaultLoginPageGeneratingFilter : 기본 로그인 페이지를 제공해 준다.
- BasicAuthenticationFilter : HTTPBasic 기반(요청 헤더를 이용한 방식)의 유저 인증을 처리한다.
- RememberMeAuthenticationFilter : 세션이 사라지거나 만료되더라도 쿠키 또는 DB를 사용해 저장된 토큰 기반 인증을 처리한다.
- SecurityContextHolderAwareRequestFilter : Spring Security에서 시큐리티 관련 서블릿 API를 구현해 준다.
- AnonymousAuthenticationFilter : SecurityContext에 Authentication 객체가 없을 경우 익명 Authentication 객체를 넣어준다.
- SessionManagementFilter : 세션 변조 공격 방지, 유효하지 않은 세션 접근 시 보낼 URL 설정, 최대 세션 수 설정, 세션 생성 전략 설정 기능을 제공한다.
- ExceptionTranslationFilter : 필터 체인 내에서 발생하는 Exception을 처리한다.
- FilterSecurityInterceptor : 인증된 사용자에 대해 요청의 승인 / 거부를 최종적으로 결정한다.

---



## Spring Security 인증 과정

![](..\..\images\spring-security-authentication-architecture.png)

여기서는 UsernamePasswordAuthenticationFilter 기준으로 설명한다.

1. 사용자가 아이디, 비밀번호를 입력하고 요청을 보낸다.

2. UsernamePasswordAuthenticationFilter가 요청을 받아서 UsernameAuthenticationToken을 생성하고 AuthenticationManager에게 처리를 위임한다.

   ```java
   @Override
   public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
         throws IOException, ServletException {
      doFilter((HttpServletRequest) request, (HttpServletResponse) response, chain);
   }
   
   private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
      
      ...
      
      try {
         Authentication authenticationResult = attemptAuthentication(request, response);
   
      ...
   }
   
   @Override
   public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
         throws AuthenticationException {
      if (this.postOnly && !request.getMethod().equals("POST")) {
         throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
      }
      String username = obtainUsername(request);
      username = (username != null) ? username : "";
      username = username.trim();
      String password = obtainPassword(request);
      password = (password != null) ? password : "";
      UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
   
      setDetails(request, authRequest);
      return this.getAuthenticationManager().authenticate(authRequest);
   }
   ```

   doFilter() 에서 attemptAuthentication()을 호출하고 attemptAuthentication() 에서 request 내용을 통해 UsernameAuthenticationToken 객체를 생성한다. 이후 AuthenticationManager의 authenticate() 호출로 UsernameAuthenticationToken 객체를 전달하면서 처리를 위임하는 모습을 볼 수 있다.

   ```java
   public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {
   
      private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;
   
      private final Object principal;
   
      private Object credentials;
       
       ...
   ```

   UserNameAuthenticationToken에는 아이디, 비밀번호가 각각 principal, credentials에 들어 있다.

3. AuthenticationManager를 구현한 ProviderManager가 처리를 실제로 위임받고, 인증 작업 처리가 가능한 AuthenticationProvider를 선택한다.

   ```java
   public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {
   
       ...
   
      private List<AuthenticationProvider> providers = Collections.emptyList();
   
       ...
          
      @Override
      public Authentication authenticate(Authentication authentication) throws AuthenticationException {
         Class<? extends Authentication> toTest = authentication.getClass();
         AuthenticationException lastException = null;
         AuthenticationException parentException = null;
         Authentication result = null;
         Authentication parentResult = null;
         int currentPosition = 0;
         int size = this.providers.size();
         for (AuthenticationProvider provider : getProviders()) {
            if (!provider.supports(toTest)) {
               continue;
            }
            if (logger.isTraceEnabled()) {
               logger.trace(LogMessage.format("Authenticating request with %s (%d/%d)",
                     provider.getClass().getSimpleName(), ++currentPosition, size));
            }
            try {
               result = provider.authenticate(authentication);
               if (result != null) {
                  copyDetails(authentication, result);
                  break;
               }
            }
   
             ...
            
      }
   ```

   ProviderManager는 AuthenticationProvider들을 List로 관리하고 있다. ProviderManager가 인증 처리를 위임받으면 List를 탐색하면서 각 AuthenticationProvider의 supports()를 호출해 인증 처리가 가능한 AuthenticationProvider를 선택해 authenticate() 를 호출한다.

4. AuthenticationProvider는 UserDetailsService를 통해 유저 정보를 가져와 아이디, 비밀번호를 확인 후 인증 성공 시 Authentication 객체를 반환한다.

   ```java
   @Override
   public Authentication authenticate(Authentication authentication) throws AuthenticationException {
   
       ...
       
         try {
            user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
         }
         catch (UsernameNotFoundException ex) {
            this.logger.debug("Failed to find user '" + username + "'");
            if (!this.hideUserNotFoundExceptions) {
               throw ex;
            }
            throw new BadCredentialsException(this.messages
                  .getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
         }
         Assert.notNull(user, "retrieveUser returned null - a violation of the interface contract");
      }
      try {
         this.preAuthenticationChecks.check(user);
         additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
      }
   
   	...
   
      return createSuccessAuthentication(principalToReturn, authentication, user);
   }
   
   @Override
   protected final UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication)
   			throws AuthenticationException {
   		prepareTimingAttackProtection();
   		try {
   			UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
   			if (loadedUser == null) {
   				throw new InternalAuthenticationServiceException(
   						"UserDetailsService returned null, which is an interface contract violation");
   			}
   			return loadedUser;
   		}
   		catch (UsernameNotFoundException ex) {
   			mitigateAgainstTimingAttack(authentication);
   			throw ex;
   		}
   		catch (InternalAuthenticationServiceException ex) {
   			throw ex;
   		}
   		catch (Exception ex) {
   			throw new InternalAuthenticationServiceException(ex.getMessage(), ex);
   		}
   	}
   
   @Override
   @SuppressWarnings("deprecation")
   protected void additionalAuthenticationChecks(UserDetails userDetails,
   			UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
   		if (authentication.getCredentials() == null) {
   			this.logger.debug("Failed to authenticate since no credentials provided");
   			throw new BadCredentialsException(this.messages
   					.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
   		}
   		String presentedPassword = authentication.getCredentials().toString();
   		if (!this.passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {
   			this.logger.debug("Failed to authenticate since password does not match stored value");
   			throw new BadCredentialsException(this.messages
   					.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
   		}
   	}
   
   @Override
   protected Authentication createSuccessAuthentication(Object principal, Authentication authentication,
   			UserDetails user) {
   		boolean upgradeEncoding = this.userDetailsPasswordService != null
   				&& this.passwordEncoder.upgradeEncoding(user.getPassword());
   		if (upgradeEncoding) {
   			String presentedPassword = authentication.getCredentials().toString();
   			String newPassword = this.passwordEncoder.encode(presentedPassword);
   			user = this.userDetailsPasswordService.updatePassword(user, newPassword);
   		}
   		return super.createSuccessAuthentication(principal, authentication, user);
   	}
   ```

   AuthenticationProvider의 authenticate()가 호출되면 retrieveUser()와 additionalAuthenticationChecks()를 통해 UserDetailService에서 DB에 있는 유저 정보를 가져와 아이디, 비밀번호가 맞는지 확인한다. 맞다면 createSuccessAuthentication()을 통해 Authentication 객체를 반환한다.

5. AuthenticationProvider -> ProviderManager -> UsernamePasswordAuthenticationFilter 순으로 Authentication 객체가 반환된다.

6. UsernamePasswordAuthenticationFilter는 Authentication 객체를 SecurityContext에 저장한다.

   ```java
   protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,
         Authentication authResult) throws IOException, ServletException {
      SecurityContext context = SecurityContextHolder.createEmptyContext();
      context.setAuthentication(authResult);
      SecurityContextHolder.setContext(context);
      if (this.logger.isDebugEnabled()) {
         this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));
      }
      this.rememberMeServices.loginSuccess(request, response, authResult);
      if (this.eventPublisher != null) {
         this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
      }
      this.successHandler.onAuthenticationSuccess(request, response, authResult);
   }
   ```

---



## Spring Security 인가 과정

![](..\..\images\spring-security-authorization.png)

인증된 사용자가 리소스에 접근할 수 있는지를 확인한다.

1. 사용자 요청이 FilterSecurityInterceptor에 도달한다.

2. FilterSecurityInterceptor는 AccessDecisionManager에게 인가 처리를 요청한다.

   ```java
   @Override
   public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
         throws IOException, ServletException {
      invoke(new FilterInvocation(request, response, chain));
   }
   
   public void invoke(FilterInvocation filterInvocation) throws IOException, ServletException {
   
       ...
       
   		InterceptorStatusToken token = super.beforeInvocation(filterInvocation);
       
   	...
   }
   
   protected InterceptorStatusToken beforeInvocation(Object object) {
   
           ...
           
   		attemptAuthorization(object, attributes, authenticated);
   
           ...
           
   		return new InterceptorStatusToken(SecurityContextHolder.getContext(), false, attributes, object);
   
   }
   
   private void attemptAuthorization(Object object, Collection<ConfigAttribute> attributes,
   			Authentication authenticated) {
   		try {
   			this.accessDecisionManager.decide(authenticated, object, attributes);
   		}
   		
       	...
           
    		finally {
   			super.finallyInvocation(token);
   		}
       
       	...
   }
   ```

   FilterSecurityInterceptor의 doFilter() -> invoke() -> beforeInvocation() -> attemptAuthorization() 순으로 호출되고, AccessDicisionManager의 decide()가 호출되면서 인가 처리를 요청한다.

3. AccessDicisionManager는 AccessDicisionVoter에게 승인 여부를 요청한다. 승인 거부 시 AccessDeniedException이 발생하게 된다. 이를 결정하는 것은 세 가지 유형이 있다.

   - AffirmativeBased : Voter 하나만 승인하면 접근 허가
   - ConsensusBased : 다수표에 의해 접근 여부 판단
   - UnanimousBased : 모든 Voter가 승인해야 접근 허가

   ```java
   @Override
   @SuppressWarnings({ "rawtypes", "unchecked" })
   public void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes)
         throws AccessDeniedException {
      int deny = 0;
      for (AccessDecisionVoter voter : getDecisionVoters()) {
         int result = voter.vote(authentication, object, configAttributes);
         switch (result) {
         case AccessDecisionVoter.ACCESS_GRANTED:
            return;
         case AccessDecisionVoter.ACCESS_DENIED:
            deny++;
            break;
         default:
            break;
         }
      }
      if (deny > 0) {
         throw new AccessDeniedException(
     this.messages.getMessage("AbstractAccessDecisionManager.accessDenied", "Access is denied"));
      }
   
      checkAllowIfAllAbstainDecisions();
   }
   ```

   위의 코드는 AccessDecisionManager를 구현한 AffirmativeBased의 모습이다. AccessDicisionManager는 AccessDecisionVoter를 List로 관리하고 있다. List를 탐색하면서 vote()를 호출해 승인 여부를 요청한다.

4. 최종 승인이 결정되면 SecurityContext에 저장한다.

   ```java
   protected void finallyInvocation(InterceptorStatusToken token) {
      if (token != null && token.isContextHolderRefreshRequired()) {
         SecurityContextHolder.setContext(token.getSecurityContext());
         if (this.logger.isDebugEnabled()) {
            this.logger.debug(LogMessage.of(
                  () -> "Reverted to original authentication " + token.getSecurityContext().getAuthentication()));
         }
      }
   }
   ```

