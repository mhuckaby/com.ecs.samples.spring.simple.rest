#Introduction

The goal of this project project is to bootstrap a secure Spring MVC REST API. 

The project contains 

- a simple OAuth2 secured REST API acting as an example.
- a set of administrative pages 

##OAuth2 Secured API
 
The API included in this project allows for you to manage your locations.

It exposes an API that allows you to

- Retrieve your current location
- Update your current location
- Retrieve your location history
- Retrieve a past location
- Add past locations
- Remove a location (TODO).

### OAuth2 Bearer Tokens

In order to execute these REST calls you need to provide a valid access token. (More on that later).

The access token is a `String` that can be passed in the `Authorization` header of the request as a `bearer token`. Performing such a request using `curl` looks like this:

	curl -H "Authorization: Bearer 1/fFBGRNJru1FQd44AzqT3Zg" https://www.googleapis.com/oauth2/v1/userinfo

All non-authorized calls will return the following JSON string

	{	"error":"unauthorized",
		"error_description":"An Authentication object was not found in the SecurityContext"
	} 

Note that the curl samples below don't contain the authorization header for simplicity.

### Retrieve current location

We can access our current location by accessing the `current location endpoint` with a `GET` request

	curl --silent "${current_location_endpoint}"

### Update current location

We can update our current location with a `POST` request on the `current location endpoint`. 
We need to provide a JSON payload containing (at least) the latitude and longitude.
 
	curl --silent  -H "Content-Type: application/json" -d '{"latitude":1.0,"longitude":1.0}' -X POST http://localhost:6002/com.ecs.samples.spring.simple.rest/currentLocation


curl --silent -H "Authorization: Bearer c9adf323-ef57-400f-8a23-0a5f529e3a47" -H "Content-Type: application/json" -d '{"latitude":1.0,"longitude":1.0}' -X POST http://localhost:6002/com.ecs.samples.spring.simple.rest/currentLocation


### Retrieving location history

We can retrieve our location history as a list of location resources through the `location endpoint` with a `GET` request.

	curl --silent "${location_endpoint}"
	
If we know the exact timestamp of a location we can go ahead and fetch that single location from the past.
	
	curl --silent "${location_endpoint}/1378113390550"

We can also provide a `min-time` and `max-time` parameter to define a start and end time.
	
	curl --silent "${location_endpoint}/?min-time=1378111961098&max-time=1378111963779"

## Add past locations

Adding locations in the past is also possible using by performing a `POST` request on the `location endpoint`.

	curl --silent  -H "Content-Type: application/json" -d '{"accuracy":null,"altitude":null,"altitudeAccuracy":null,"heading":null,"latitude":10.0,"longitude":10.0,"speed":null,"timestampMs":1378112866695}' -X POST http://localhost:6002/com.ecs.samples.spring.simple.rest/location

##Administrative pages
The project also contains a set of administrative pages allowing you look at information about the registered users and issues tokens.

###User overview

The [user overview](http://localhost:6002/com.ecs.samples.spring.simple.rest/user/list.html) can be accessed via the following URL:

	http://localhost:6002/com.ecs.samples.spring.simple.rest/user/list.html

Screenshot:

![User overview](https://dl.dropboxusercontent.com/u/13246619/Blog%20Articles/SpringOAuth/users.PNG)

###Token overview

The token overview can be accessed via the following URL:

	http://localhost:6002/com.ecs.samples.spring.simple.rest/accesstoken/list.html

![Token overview](https://dl.dropboxusercontent.com/u/13246619/Blog%20Articles/SpringOAuth/tokens.PNG)

###Oauth Client overview

TODO : create CRUD pages for registering oauth clients.

# Project Setup

This section describes the local setup that is required to run the project. We've tried to kept the dependencies to a minimum.

## Environment variables

*Optional : * You can export the following environment variables to facilitate testing.

### Local setup
	
	export host=localhost
	export port=6002
	export location_endpoint=http://${host}:${port}/com.ecs.samples.spring.simple.rest/location
	export current_location_endpoint=http://${host}:${port}/com.ecs.samples.spring.simple.rest/currentLocation

### System properties

You can export the following maven options so that the project is started in debug mode when launched through `mvn tomcat:run`.
It allows for a very convenient way to test the application.

- on Linux / Unix / Mac OS X

		export MAVEN_OPTS="-Xdebug -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n -DlogFileLocation=/tmp"

- on Windows :

		set MAVEN_OPTS=-Xdebug -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n -DlogFileLocation=C:/TEMP


# Debugging

You can turn on Spring Security Debugging by adding the `debug` component in your spring context.

	2013-09-07 00:55:10,635 WARN  Spring Security Debugger - 
	
	********************************************************************
	**********        Security debugging is enabled.       *************
	**********    This may include sensitive information.  *************
	**********      Do not use in a production system!     *************
	********************************************************************



# Testing

## Oauth2Test 

There is a Oauth2Test class included in the project that allows you to launch an OAuth2 flow. It's not a completely automated test and it does require user-input.
It even goes as far as starting a browser so that the user can login and authorize our application.  

### Login page

If the user is not authenticated, he'll need to provide the proper security credentials

![Login](https://dl.dropboxusercontent.com/u/13246619/Blog%20Articles/SpringOAuth/login.png)	

If authentication is succesfull, the user will be redirected to the authorization page

### Authorization page

After having logged in, the user needs to authorize our application (our client the `my-trusted-client`) to access his data.

![Authorization](https://dl.dropboxusercontent.com/u/13246619/Blog%20Articles/SpringOAuth/authorize.png)	

The page URL is the OAuth2 authorization URL, As you can see if passes a number of parameters including

-client id
-redirect uri
-response type
-scope
	
The full URL looks like this	
	
	http://localhost:6002/com.ecs.samples.spring.simple.rest/oauth/authorize?client_id=my-trusted-client&redirect_uri=http://localhost:64938/Callback&response_type=code&scope=location%20

### Tokens

The test stores its tokens in the following location:

	~/.store/oauth2_sample/StoredCredential 

In order to remove that local cache, simply remove the folder before running the test again,.

	rm -rf ~/.store/oauth2_sample/StoredCredential

You can create an alias for that command to save you some typing.

	alias clear_tokens='rm -rf ~/.store/oauth2_sample/StoredCredential'

## Using CURL

If you want to use curl to perform authorized calls you need to provide an OAuth2 Bearer Access Token.
We can again make this a bit easier for you by defining an alias like this :

	alias acurl='curl -H "Authorization: Bearer ${access_token}" $1'
	
That way, all you need to provide is an `access_token` like this:

	access_token=0e635793-c48e-483d-91cf-27c37f02c398

And call the secured url like this

	curl http://localhost:6002/com.ecs.samples.spring.simple.rest/whoami

IF all gors well you should get a response like this:

	{"password":"2a634b10e5c2b645008098465cce659c",
	"username":"ddewaele@gmail.com",
	"authorities":[{"authority":"ROLE_USER"}],
	"accountNonExpired":true,
	"accountNonLocked":true,
	"credentialsNonExpired":true,
	"enabled":true}


# Spring

Spring Security is the core framework used by this project. 

## Spring context

## Annotation and component scanning

- Added `<context:annotation-config />` allowing us to activate annotations in beans already registered in the application context (no matter if they were defined with XML or by package scanning).
- Added `<context:component-scan>` to scan packages to find and register beans within the application context.
- Added `<mvc:annotation-driven>` to support for new Spring MVC features such as declarative validation with @Valid, HTTP message conversion with @RequestBody/@ResponseBody, new field conversion architecture, etc
- Added `<tx:annotation-driven>` to enable the transactional behavior based on annotations.

##EntityManagerFactory

As we are in the business of storing / retrieving locations we need a persistence layer in place.
We're using standard JPA inside our Spring container, using Hibernate as the vendor. 

The first thing we need to define is an EntityManagerFactory. The simplest way to create an EntityManagerFactory bean is to define it like this:
	
	<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="persistenceUnitName" value="spring-jpa" />
		<property name="jpaVendorAdapter">
			<bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
				<property name="showSql" value="true" />
				<property name="generateDdl" value="true" />
				<property name="database" value="HSQL" />
			</bean>
		</property>
	</bean>
    
Notice how we point to

- a dataSource (this can point to a simple embedded database)
- a persistenceUnitName (points to the pu defined in the persistence.xml)
- a jpaVendorAdapter (uses the hibernate specific entity manager)

## The datasource

For the moment we're using an embedded database (HSQL). At a later stage this will be replaced with an actual database.

	<jdbc:embedded-database id="dataSource" type="HSQL" />    

## The persistence.xml

Notice how the `EntityManagerFactory` points to a `persistenceUnitName`. The `persistenceUnitName` is defined in a `persistence.xml` file that is located in the META-INF folder of the package that holds the Entity classes.

	<persistence version="2.0" xmlns="http://java.sun.com/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd">
		<persistence-unit name="spring-jpa">
			<provider>org.hibernate.ejb.HibernatePersistence</provider>
		</persistence-unit>
	</persistence>

## The transactionManager

We also need a transactionManager for our entityManager. 

	<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
			<property name="entityManagerFactory" ref="entityManagerFactory" />
	</bean>
	
# OAuth2

The REST API used in this project is being secured through Oauth2. This means that in order to access the API you need to be authorized to do so.

Attempting to access the API without being authorized results in the following error

	< HTTP/1.1 401 Unauthorized
	< Server: Apache-Coyote/1.1
	< Cache-Control: no-store
	< Pragma: no-cache
	< WWW-Authenticate: Bearer realm="sparklr2", error="unauthorized", error_description="An Authentication object was not found in the SecurityContext"
	< Content-Type: application/json;charset=UTF-8
	< Transfer-Encoding: chunked
	< Date: Tue, 03 Sep 2013 07:43:57 GMT
	< 
	* Connection #0 to host localhost left intact
	* Closing connection #0
	{"error":"unauthorized","error_description":"An Authentication object was not found in the SecurityContext"}

## web.xml

It's important to realize that all configuration done in the Spring Context is only put into effect if we have the following filter in place. 

	<filter>
		<filter-name>springSecurityFilterChain</filter-name>
		<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
		<init-param>
			<param-name>contextAttribute</param-name>
			<param-value>org.springframework.web.servlet.FrameworkServlet.CONTEXT.spring</param-value>
		</init-param>
	</filter>

	<filter-mapping>
		<filter-name>springSecurityFilterChain</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

In order to use Oauth2 in Spring MVC, we require a lot of configuration in the context.

## Spring Context setup

### Protected URLS

The first thing we'll do is list our protected URLs.

	<http pattern="/location/**" create-session="never" entry-point-ref="oauthAuthenticationEntryPoint"
		access-decision-manager-ref="accessDecisionManager" xmlns="http://www.springframework.org/schema/security">
		<anonymous enabled="false" />
		<intercept-url pattern="/location" access="ROLE_USER,SCOPE_LOCATIONHISTORY" />
		<custom-filter ref="resourceServerFilter" before="PRE_AUTH_FILTER" />
		<access-denied-handler ref="oauthAccessDeniedHandler" />
	</http>
	
This means that everyone attempting to access `/location/*` on our REST API will need to be authenticated and authorized to do so.
We also only allow entities with the role `USER` andd acope `LOCATIONHISTORY` to access this resource.
	
Notice how this `http` definition relies on

- Authentication Entry Point
- AccessDeniedHandler
- Custom Filter
- Access Decision Manager

We'll cover these different components in details in the following sections. 

### Authentication Entry Point

An Authentication Entry Point capable of commencing an authentication scheme. If authentication fails and the caller has asked for a specific content type response, this entry point can send one, along with a standard 401 status

	<bean id="oauthAuthenticationEntryPoint" class="org.springframework.security.oauth2.provider.error.OAuth2AuthenticationEntryPoint">
		<property name="realmName" value="sparklr2" />
	</bean>
	
This bean kicks in when things go wrong .... If for example we try to access the API without providing the proper authentication credentials the Spring security framework will throw an `AuthenticationCredentialsNotFoundException`.
(other examples of exceptions include expired accounts, locked accounts, invalid account credentials,....)

When such an authentication error occurs, this `OAuth2AuthenticationEntryPoint` (that also acts as an ExceptionHandler) will handle the exception by enhancing the response.

It will add the `WWW-Authenticate` HTTP header with the following value
 
	Bearer realm="sparklr2", error="unauthorized", error_description="An Authentication object was not found in the SecurityContext" 

### AccessDeniedHandler

If authorization fails and the caller has asked for a specific content type response, this entry point can send one, along with a standard 403 status. 

	<bean id="oauthAccessDeniedHandler" class="org.springframework.security.oauth2.provider.error.OAuth2AccessDeniedHandler" />

### Custom Filter

OAuth Resource Server ( = API server used to access the user's information.) is defined by a `resource id` and a `token service`.

	<oauth:resource-server id="resourceServerFilter" resource-id="latifyApi" token-services-ref="tokenServices" />	
	
### Access Decision Manager

We also need to define an accessDecisionManager, capable of making a final access control (authorization) decision

This particular one requires  all voters to abstain or grant access.

	<bean id="accessDecisionManager" class="org.springframework.security.access.vote.UnanimousBased">
		<constructor-arg>
			<list>
				<bean class="org.springframework.security.oauth2.provider.vote.ScopeVoter" />
				<bean class="org.springframework.security.access.vote.RoleVoter" />
				<bean class="org.springframework.security.access.vote.AuthenticatedVoter" />
			</list>
		</constructor-arg>
	</bean>
	
### TokenService

A Spring provided default implementation of a token service, using random UUID values for the access token and refresh token values
	
- The tokenService delegates the persistence of the tokens to a `tokenStore`. 
- To access client specific details it uses a `clientDetailsService`.
	
	<bean id="tokenServices" class="org.springframework.security.oauth2.provider.token.DefaultTokenServices">
		<property name="tokenStore" ref="tokenStore" />
		<property name="supportRefreshToken" value="true" />
		<property name="clientDetailsService" ref="clientDetails" />
	</bean>	

### TokenStore

In the initial version of the sample we were using an in-memory implementation.

	<bean id="tokenStore" class="org.springframework.security.oauth2.provider.token.InMemoryTokenStore" />

We've replaced the in-memory tokenStore with a JDBC based tokenStore.

	<bean id="tokenStore" class="org.springframework.security.oauth2.provider.token.JdbcTokenStore">
		<constructor-arg ref="dataSource" />
	</bean>

For that we created 2 new JPA entities

#### OAuthAccessToken

	package com.ecs.samples.spring.simple.rest.model;
	
	import javax.persistence.Entity;
	import javax.persistence.Id;
	import javax.persistence.Lob;
	import javax.persistence.Table;
	
	@Entity()
	@Table(name = "oauth_access_token")
	public class OAuthAccessToken {
	
		@Id
		private String token_id;
	
		@Lob
		private byte[] token;
	
		private String authentication_id;
	
		private String user_name;
	
		private String client_id;
	
		@Lob
		private byte[] authentication;
	
		private String refresh_token;
	
		public String getToken_id() {
			return token_id;
		}
	
		public void setToken_id(String token_id) {
			this.token_id = token_id;
		}
	
		public byte[] getToken() {
			return token;
		}
	
		public void setToken(byte[] token) {
			this.token = token;
		}
	
		public String getAuthentication_id() {
			return authentication_id;
		}
	
		public void setAuthentication_id(String authentication_id) {
			this.authentication_id = authentication_id;
		}
	
		public String getUser_name() {
			return user_name;
		}
	
		public void setUser_name(String user_name) {
			this.user_name = user_name;
		}
	
		public String getClient_id() {
			return client_id;
		}
	
		public void setClient_id(String client_id) {
			this.client_id = client_id;
		}
	
		public byte[] getAuthentication() {
			return authentication;
		}
	
		public void setAuthentication(byte[] authentication) {
			this.authentication = authentication;
		}
	
		public String getRefresh_token() {
			return refresh_token;
		}
	
		public void setRefresh_token(String refresh_token) {
			this.refresh_token = refresh_token;
		}
	
		@Override
		public String toString() {
			return "OAuthAccessToken [token_id=" + token_id
					+ ",  authentication_id=" + authentication_id + ", user_name="
					+ user_name + ", client_id=" + client_id + ", refresh_token="
					+ refresh_token + "]";
		}
	
	}

#### OAuthRefreshToken

	package com.ecs.samples.spring.simple.rest.model;
	
	import javax.persistence.Entity;
	import javax.persistence.Id;
	import javax.persistence.Lob;
	import javax.persistence.Table;
	
	@Entity
	@Table(name = "oauth_refresh_token")
	public class OAuthRefreshToken {
	
		@Id
		private String token_id;
	
		@Lob
		private byte[] token;
	
		@Lob
		private byte[] authentication;
	
		public String getToken_id() {
			return token_id;
		}
	
		public void setToken_id(String token_id) {
			this.token_id = token_id;
		}
	
		public byte[] getToken() {
			return token;
		}
	
		public void setToken(byte[] token) {
			this.token = token;
		}
	
		public byte[] getAuthentication() {
			return authentication;
		}
	
		public void setAuthentication(byte[] authentication) {
			this.authentication = authentication;
		}
	
		@Override
		public String toString() {
			return "OAuthRefreshToken [token_id=" + token_id + "]";
		}
	
	}


### Client Details Service

We start by creating a client details service that will list all of our OAuth clients. These include the `client id` and `client secret` that your API clients need to provide in order to initiate an Oauth flow.
Initially we maintained a simple static list inside the spring context.

	<oauth:client-details-service id="clientDetails">
		<oauth:client client-id="my-trusted-client" authorized-grant-types="password,authorization_code,refresh_token,implicit"
			secret="somesecret"  authorities="ROLE_CLIENT, ROLE_TRUSTED_CLIENT" scope="read,write,trust" access-token-validity="60" />
	</oauth:client-details-service>

We're going to replace this with a more dynamic administrative system where new clients can be registered through a web page.

The first step to accomplish this is to move away from the static list and replace it with a JDBC backed store.

	<bean id="clientDetails" class="org.springframework.security.oauth2.provider.JdbcClientDetailsService">
      <constructor-arg ref="dataSource" />
	</bean>

We'll also provide an `import.sql` file to provision the table that is used by the `JdbcClientDetailsService`.

	INSERT INTO OAUTH_CLIENT_DETAILS (client_id,client_secret,resource_ids,scope,authorized_grant_types,web_server_redirect_uri,authorities,access_token_validity,refresh_token_validity,additional_information) VALUES ('my-trusted-client','somesecret',null,'location,locationhistory','password,authorization_code,refresh_token,implicit',null,'ROLE_CLIENT, ROLE_TRUSTED_CLIENT',60,null,null);
	
# Accessing the user

## The UserDetailsManager

We can inject a `UserDetailsManager` into any controller like this :

	@Resource(name="jdbcUserDetailsService")
	private UserDetailsManager userDetailsService;
	
As we're using multiple UserDetailsManager types in our config, we're targetting a specific userDetailsService by name here. (the one capable of retrieving user details). 

Remember that we also had a UserDetailsService specifically for looking up Oauth2 Clients.

	<!-- This UserDetailsService is only used to lookup Oauth2 Clients, not actual users. -->
	<bean id="clientDetailsUserService" class="org.springframework.security.oauth2.provider.client.ClientDetailsUserDetailsService">
		<constructor-arg ref="clientDetails" />
	</bean> 	

The UserDetailsManager allows us to access the `User` or `UserDetails` object


I'v created a very simple `whoami` request in the ServiceUserController

	@RequestMapping("/whoami")
	@Controller
	public class ServiceUserController {
	
		@Resource(name="jdbcUserDetailsService")
		private UserDetailsManager userDetailsService;
		
		@ResponseBody
		@RequestMapping("")
		public UserDetails getPhotoServiceUser(Principal principal)
		{
			UserDetails userDetails = userDetailsService.loadUserByUsername(principal.getName());
			return userDetails;
		}
	} 

When invoked like this:

	curl -H "Authorization: Bearer 1f3b2699-2dd5-4a5e-8575-0d72c6aa3fbd" http://localhost:6002/com.ecs.samples.spring.simple.rest/whoami
	
It will return the UserDetails object

{"password":"2a634b10e5c2b645008098465cce659c","username":"ddewaele@gmail.com","authorities":[{"authority":"ROLE_USER"}],"accountNonExpired":true,"accountNonLocked":true,"credentialsNonExpired":true,"enabled":true}


## The injected principal

Spring can automatically inject the Principal if one is provided as an argument to a REST function 

	org.springframework.security.core.userdetails.User@73ffe800: 
		Username: ddewaele@gmail.com; 
		Password: [PROTECTED]; 
		Enabled: true; 
		AccountNonExpired: true; 
		credentialsNonExpired: true; 
		AccountNonLocked: true; 
		Granted Authorities: ROLE_USER; 
		Credentials: [PROTECTED]; 
		Authenticated: true; 
		Details: remoteAddress=0:0:0:0:0:0:0:1%0, , 
		tokenValue=<TOKEN>; 
		Granted Authorities: ROLE_USER

So if you have a method like this, the Principal will be made available by Spring.

	@Transactional
	@RequestMapping(method=RequestMethod.POST,consumes = APPLICATION_JSON,produces= APPLICATION_JSON)
	public @ResponseBody Location updateCurrentLocation(@RequestBody Location location,Principal principal) {
		....
	}

## The Security Context

You can also get a hold of an Authentication object. 

	Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

The authentication object contains the same principal as the one we saw when we added the Principal argument to our methods. 

	org.springframework.security.oauth2.provider.OAuth2Authentication@4e3c521f: 
		Principal: org.springframework.security.core.userdetails.User@73ffe800: 
			Username: ddewaele@gmail.com; 
			Password: [PROTECTED]; 
			Enabled: true; 
			AccountNonExpired: true; 
			credentialsNonExpired: true; 
			AccountNonLocked: true; 
			\Granted Authorities: ROLE_USER; 
			Credentials: [PROTECTED]; 
			Authenticated: true; 
			Details: remoteAddress=0:0:0:0:0:0:0:1%0, , 
			tokenValue=<TOKEN>; 
			Granted Authorities: ROLE_USER


# Errors occured


## Spring MVC errors 

TODO : try to reproduce.

- The request sent by the client was syntactically incorrect ().
(This was caused by invalid JSON being sent - attribute names were not encapsulated in quotes)
- The server refused this request because the request entity is in a format not supported by the requested resource for the requested method ().
- The resource identified by this request is only capable of generating responses with characteristics not acceptable according to the request "accept" headers ().

## Oauth2 Authorization URL error

### Description

An error occured when the oauth2 authorization was hit.

	http://localhost:6002/com.ecs.samples.spring.simple.rest/oauth/authorize?client_id=my-trusted-client&redirect_uri=http://localhost:50577/Callback&response_type=code&scope=read%20write%20trust

Full stack trace

	org.springframework.security.authentication.InsufficientAuthenticationException: User must be authenticated with Spring Security before authorization can be completed.
		org.springframework.security.oauth2.provider.endpoint.AuthorizationEndpoint.authorize(AuthorizationEndpoint.java:145)
		sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
		sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
		sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
		java.lang.reflect.Method.invoke(Method.java:597)
		org.springframework.web.method.support.InvocableHandlerMethod.invoke(InvocableHandlerMethod.java:219)
		org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:132)
		org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:104)
		org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandleMethod(RequestMappingHandlerAdapter.java:745)
		org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:686)
		org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:80)
		org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:925)
		org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:856)
		org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:936)
		org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:827)
		javax.servlet.http.HttpServlet.service(HttpServlet.java:617)
		org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:812)
		javax.servlet.http.HttpServlet.service(HttpServlet.java:717)
		org.springframework.security.web.FilterChainProxy.doFilterInternal(FilterChainProxy.java:186)
		org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:160)
		org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:346)
		org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:259)

## Solution

This means that we didn't enter the Oauth authorization flow with an authenticated user.
Keep in mind that as you are authorizating an application access to your data, you need to have 
a logged in user at this point.

Therefor, we need to provide a login page so that the user can login to the system if he didn't do so already.

In this sample, we're using a simple form based login.

	<http access-denied-page="/login.jsp?authorization_error=true" disable-url-rewriting="true"
		xmlns="http://www.springframework.org/schema/security">
		<intercept-url pattern="/oauth/**" access="ROLE_USER" />
		<intercept-url pattern="/**" access="IS_AUTHENTICATED_ANONYMOUSLY" />

		<form-login authentication-failure-url="/login.jsp?authentication_error=true" default-target-url="/index.jsp"
			login-page="/login.jsp" login-processing-url="/login.do" />
		<logout logout-success-url="/index.jsp" logout-url="/logout.do" />
		<anonymous />
	</http>


## Failure after capturing authorization code

### Description

After the user clicks authorize, we receive the authorization code but fail to swap it for a token.

This is because the oauth token url is not properly setup 

	http://localhost:6002/com.ecs.samples.spring.simple.rest/oauth/token
	
We need to have the following beans in place	
	
	<http pattern="/oauth/token" create-session="stateless" authentication-manager-ref="clientAuthenticationManager"
		xmlns="http://www.springframework.org/schema/security">
		<intercept-url pattern="/oauth/token" access="IS_AUTHENTICATED_FULLY" />
		<anonymous enabled="false" />
		<http-basic entry-point-ref="clientAuthenticationEntryPoint" />
		<!-- include this only if you need to authenticate clients via request parameters -->
		<custom-filter ref="clientCredentialsTokenEndpointFilter" after="BASIC_AUTH_FILTER" />
		<access-denied-handler ref="oauthAccessDeniedHandler" />
	</http>
	
	<bean id="clientAuthenticationEntryPoint" class="org.springframework.security.oauth2.provider.error.OAuth2AuthenticationEntryPoint">
		<property name="realmName" value="com.ecs.latify.api/client" />
		<property name="typeName" value="Basic" />
	</bean>
	
	<sec:authentication-manager id="clientAuthenticationManager">
		<sec:authentication-provider user-service-ref="clientDetailsUserService" />
	</sec:authentication-manager>
 		
	<bean id="clientCredentialsTokenEndpointFilter" class="org.springframework.security.oauth2.provider.client.ClientCredentialsTokenEndpointFilter">
		<property name="authenticationManager" ref="clientAuthenticationManager" />
	</bean>
		
## authenticationManager not defined.

### Error message  

No bean named 'org.springframework.security.authenticationManager' is defined

### Description

The following error occurs if you don't have an authentication provider in your Spring Context.

	org.springframework.beans.factory.NoSuchBeanDefinitionException: 
	No bean named 'org.springframework.security.authenticationManager' is defined: 
	Did you forget to add a gobal <authentication-manager> element to your configuration 
	(with child <authentication-provider> elements)? 
	Alternatively you can use the authentication-manager-ref attribute on your <http> and <global-method-security> elements.

### Solution

In order to solve this you need to have an `authentication-manager` defined with one or more `authentication-provider` elements.
In this case we;re using a very simple user service that defines the users in a status way.

	<sec:authentication-manager alias="authenticationManager">
		<sec:authentication-provider>
			<sec:user-service id="userDetailsService">
				<sec:user name="marissa" password="koala" authorities="ROLE_USER" />
				<sec:user name="paul" password="emu" authorities="ROLE_USER" />
			</sec:user-service>
		</sec:authentication-provider>
	</sec:authentication-manager>	

## Oauth2 authorization page not available

## Error message

HTTP 404 when going to the oauth authorization url.

### Description

Part of the OAuth32 flow is the authorization flow, where the user will need to allow a third party app to access his data. This is typically handled through an authorization page. For this particular error the authorization page doesn't pop. 

	http://localhost:6002/com.ecs.samples.spring.simple.rest/oauth/authorize?client_id=my-trusted-client&redirect_uri=http://localhost:51122/Callback&response_type=code&scope=read%20write%20trust

### Solution

You need to setup an authorization-server

	<oauth:authorization-server 
	    client-details-service-ref="clientDetails" 
	    token-services-ref="tokenServices"
		user-approval-handler-ref="userApprovalHandler">
		<oauth:authorization-code />
		<oauth:implicit />
		<oauth:refresh-token />
		<oauth:client-credentials />
		<oauth:password />
	</oauth:authorization-server>

## Insufficient scope

### Error message

HTTP 403 Forbidden occurs when trying to access the API

	{"error":"insufficient_scope","error_description":"Insufficient scope for this resource","scope":"LOCATION"}

### Solution


##

### error

org.springframework.beans.factory.parsing.BeanDefinitionParsingException: Configuration problem: No AuthenticationEntryPoint could be established. Please make sure you have a login mechanism configured through the namespace (such as form-login) or specify a custom AuthenticationEntryPoint with the 'entry-point-ref' attribute 
Offending resource: class path resource [spring-servlet.xml]



Make sure you provide the correct scopes.. (elaborate(.

# References

## Security

- [Spring OAuth 2 Developers Guide](https://github.com/SpringSource/spring-security-oauth/wiki/oAuth2)

# TODOs

- Remove in-mem-DB with a real DB
- Find out how the AccessDeniedHandler works and how it is triggered.

# Notes

Distributed / lightweight systems & services

REST / HTTP / JSON 



Resource owner ( the user)
Resource server
Authorization Server
Client application
	is the client_id / client_secret
	


Presenting the bearer token.
I bear the token

what happens when the resource server receives the bearer token


echo $TOKEN | cut -d '.' -f 2 | base64 -d


Ideas for a screencast
Howto build up

1. describe the REST API
	do some curl calls
	
2. start protecting your url
	get the dependencies sorted out
	
	
	
Why is all of this going through the filters

Request received for '/resources/images/forward_disabled.png':

org.apache.catalina.connector.RequestFacade@767ebd7d

servletPath:/resources/images/forward_disabled.png
pathInfo:null

Security filter chain: [
  SecurityContextPersistenceFilter
  LogoutFilter
  UsernamePasswordAuthenticationFilter
  RequestCacheAwareFilter
  SecurityContextHolderAwareRequestFilter
  AnonymousAuthenticationFilter
  SessionManagementFilter
  ExceptionTranslationFilter
  FilterSecurityInterceptor
]
	