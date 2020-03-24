---
title: Spring解决CORS问题
tags:
  - Java
  - Spring
  - CORS
categories: Java
abbrlink: 241123514
date: 2020-03-22 10:19:45
---
# 前言
出于安全原因，浏览器禁止AJAX调用当前来源之外的资源，跨域资源共享（CORS）是由大多数浏览器实施的W3C规范，使您可以灵活地指定对哪种跨域请求进行授权。
<!-- more -->
从Spring Framework 4.2开始，开箱即用地支持CORS。 CORS请求（包括带有OPTIONS方法的预检请求）将自动分派到各种已注册的HandlerMappings。
# Controller方法的CORS配置
- 可以在方法上加上**@CrossOrigin**注解
```java
@RestController
@RequestMapping("/account")
public class AccountController {

	@CrossOrigin
	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
```
- 也可以在Controller上进行全局配置
```java
@CrossOrigin(origins = "http://domain2.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
```
- 同时也可以使用Controller级别和方法级别的注解
```java
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

	@CrossOrigin("http://domain2.com")
	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
```
# 全局配置
## Java类配置
```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/api/**")
			.allowedOrigins("http://domain2.com")
			.allowedMethods("PUT", "DELETE")
			.allowedHeaders("header1", "header2", "header3")
			.exposedHeaders("header1", "header2")
			.allowCredentials(false).maxAge(3600);
	}
}
```
## xml配置文件配置
```xml
<mvc:cors>

	<mvc:mapping path="/api/**"
		allowed-origins="http://domain1.com, http://domain2.com"
		allowed-methods="GET, PUT"
		allowed-headers="header1, header2, header3"
		exposed-headers="header1, header2" allow-credentials="false"
		max-age="123" />

	<mvc:mapping path="/resources/**"
		allowed-origins="http://domain1.com" />

</mvc:cors>
```
# Filter配置支持CORS
```java
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

public class MyCorsFilter extends CorsFilter {

	public MyCorsFilter() {
		super(configurationSource());
	}

	private static UrlBasedCorsConfigurationSource configurationSource() {
		CorsConfiguration config = new CorsConfiguration();
		config.setAllowCredentials(true);
		config.addAllowedOrigin("http://domain1.com");
		config.addAllowedHeader("*");
		config.addAllowedMethod("*");
		UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
		source.registerCorsConfiguration("/**", config);
		return source;
	}
}
```