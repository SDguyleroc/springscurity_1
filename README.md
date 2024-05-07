# spring scurity _1
notes from: https://spring.io/guides/gs/securing-web

Starting with Spring Initializr

You can use this pre-initialized project and click Generate to download a ZIP file. This project is configured to fit the examples in this tutorial.
simple web app that we going to secure:
```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org">
    <head>
        <title>Spring Security Example</title>
    </head>
    <body>
        <h1>Welcome!</h1>

        <p>Click <a th:href="@{/hello}">here</a> to see a greeting.</p>
    </body>
</html>
```
This simple view includes link to the /hello page , which is defined in this Thymeleaf template ( from src/main/resources/templates/hello.html):
```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org">
    <head>
        <title>Hello World!</title>
    </head>
    <body>
        <h1>Hello world!</h1>
    </body>
</html>
```
The web application is based on Spring MVC. As result , you need to configure Spring MVC and set up view controllers to expose these tmplates
Spring MVC The following list (from src/main/java/com/example/securingweb/MvcConfig.java) shows a class that configures Spring MVC in the application:

```java
package com.example.securingweb;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MvcConfig implements WebMvcConfigurer {

	public void addViewControllers(ViewControllerRegistry registry) {
		registry.addViewController("/home").setViewName("home");
		registry.addViewController("/").setViewName("home");
		registry.addViewController("/hello").setViewName("hello");
		registry.addViewController("/login").setViewName("login");
	}

}
```
The addViewControllers() method(which overides the method of the same name in webMVCCongigure) adds four view controllers. Two of the view controllers reference the view whose name is home (defined in home.thml), and another references the view named hello (defined in hello.html). The fourth view controller references another view named login. You will create that view in the next section


## Set up Spring Security

Supposed you want to prevent unauthorized user from the viwing the greeting page at /hello. As it is now, if visitor click the link on the home page, they see the greeting with no barries to stop them. You need to add a barrier that forces the visitor to sing in before they can see that page.

 You do that by configuring Spring Secuirty in the application. If Spring Scurity is on the classpath, Spring boot automatically secures all htt endpoints with basic authentication. However, you can further customixed the scuirty settings. The first thing you need to do is add spring sucurity to the classpath.

 


With Maven, you need to add two extra entries (one for the application and one for testing) to the <dependencies> element in pom.xml, as the following listing shows:
```xml

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
	<groupId>org.thymeleaf.extras</groupId>
	<artifactId>thymeleaf-extras-springsecurity6</artifactId>
	<!-- Temporary explicit version to fix Thymeleaf bug -->
	<version>3.1.1.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-test</artifactId>
	<scope>test</scope>
```
The folowing security configuration ( from src/main/java/com/example/securingweb/WebSecurityConfig.java) ensures that only authenticated users can see the secrete greeting:


```java
package com.example.securingweb;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class WebSecurityConfig {

	@Bean
	public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
		http
			.authorizeHttpRequests((requests) -> requests
				.requestMatchers("/", "/home").permitAll()
				.anyRequest().authenticated()
			)
			.formLogin((form) -> form
				.loginPage("/login")
				.permitAll()
			)
			.logout((logout) -> logout.permitAll());

		return http.build();
	}

	@Bean
	public UserDetailsService userDetailsService() {
		UserDetails user =
			 User.withDefaultPasswordEncoder()
				.username("user")
				.password("password")
				.roles("USER")
				.build();

		return new InMemoryUserDetailsManager(user);
	}
}
```





















