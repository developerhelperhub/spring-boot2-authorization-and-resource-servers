# Spring Boot 2.2.5 Oauth2 Authorization Server and Resource Server

This repository contains the Oauth2 authorization server and resource server implementation with JWT token store. This example is continuation of the [Oauth2 Autherization Server and Client Application](https://github.com/developerhelperhub/spring-boot2-oauth2-server-grant-password-refresh-token/) example. I would suggest, please look previous implementation before looking this source code. In the previous example, I used same authentication server as resource. In this example we have separate resource server. Whenever we have authentication server and resoruces server are different, we need to use central token management. Currently I am using the JWT token store and pervious example we used in memory token store. sprint boot provides default in memory token store.

This repository contains four maven project. 
* my-cloud-service: Its main module, it contains the dependecy management of our application.
* identity-service: This authentication server service. 
* client-application-service: This client application for authentication server.
* resource-service: This resource server to provide the resource services for our application.

### Updation and additions in the identity-service
We need to add maven dependency to manage and support the JWT token service in the ```pom.xml```. Spring boot provides the below dependency to support it.

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-jwt</artifactId>
    <version>1.1.0.RELEASE</version>
</dependency>
```

We need to add the additional code in the ```AuthorizationServerConfig``` to configure and manage the JWT token management. The below code are implemented.

* Need to create the ```JwtAccessTokenConverter``` bean to convert the JWT token. In this example, we are using sign key method to convert JWT token for authorization server and resource server.

```java
	@Bean
	public JwtAccessTokenConverter accessTokenConverter() {
		JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
		converter.setSigningKey("123456");
		return converter;
	}
```

* Need to create the TokenStore bean to specify which type of toke store, here we need to specify the ```JwtTokenStore```

```java
	@Bean
	public TokenStore tokenStore() {
		return new JwtTokenStore(accessTokenConverter());
	}
```

* Need to add the token store service in the endpoint configuration of ```AuthorizationServerEndpointsConfigurer```.

```java
	@Override
	public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
		endpoints.authenticationManager(authenticationManager).userDetailsService(userDetailsService)
				.accessTokenConverter(accessTokenConverter());
	}
```

We need to make sure to add ```@EnableResourceServer``` annotation in the spring boot ```IdentityServiceApplication``` main class to access the resource services from the authroization server.

Above all changes added in the respective classes, we can run the spring boot application, this application run on 8081 and the context path will ```/auth```. We can use this url ```http://localhost:8081/auth/login``` to check, whether it is working or not.

**Note:** I got on error while implementing the JWT token in the authroization server. ```Cannot convert access token to JSON``` this error we got because of I missed to ```@Bean``` in the ```accessTokenConverter``` method.

### To generate the tokens with grant type "password"

Here, I am using Postman to test the grant types. Please open the Postman and open a new tab. We have to add below configuration and data in the tab.
* Method: POST
* URL: http://localhost:8081/auth/oauth/token
* Select the "Autherization" tab and change the type to "Basic Auth". Enter the username and password of client id and client secrete. Click the "Update Request" button
* Select the "Body" tab and select "x-www-form-urlencoded" option
* Add the keys and values in the form
  - grant_type is password
  - username is mycloud
  - password is mycloud@1234
* Click the "Send" button.

The API give the response contains
```json
{
    "access_token": "590adfd7-3503-446a-ac0c-3c65341aaf12",
    "token_type": "bearer",
    "refresh_token": "a0cae88d-eac7-4688-b09d-8c05b61ffe96",
    "expires_in": 43198,
    "scope": "user_info"
}
```

### To generate the tokens with grant type "refresh_token"

Open a new tab. We have to add below configuration and data in the tab.
* Method: POST
* URL: http://localhost:8081/auth/oauth/token
* Select the "Autherization" tab and change the type to "Basic Auth". Enter the username and password of client id and client secrete. Click the "Update Request" button
* Select the "Body" tab and select "x-www-form-urlencoded" option
* Add the keys and values in the form
  - grant_type is refresh_token
  - refresh_token is ```a0cae88d-eac7-4688-b09d-8c05b61ffe96```
* Click the "Send" button.
 
The API give the response contains
```json
{
    "access_token": "162544f0-5dd5-4500-abe7-58c4be74bfab",
    "token_type": "bearer",
    "refresh_token": "c6c647d1-4985-4474-8374-e0f0b1bccf90",
    "expires_in": 43198,
    "scope": "user_info"
}
```

### Reference
* [developer.okta.com](https://developer.okta.com/blog/2019/03/12/oauth2-spring-security-guide)
* [Oauth2 Autherization Server and Client Application](https://github.com/developerhelperhub/spring-boot2-oauth2-server-and-client)
* [Fix for "unsupported grant type"](https://stackoverflow.com/questions/52194081/spring-boot-oauth-unsupported-grant-type)
