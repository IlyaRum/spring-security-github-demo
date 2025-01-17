Пример настройки OAuth2 авторизации с использованием Spring Boot версии 3.4.1 и GitHub в качестве
поставщика.

### Шаг 1: Регистрация приложения в GitHub

Зарегистрируйте своё приложение в GitHub, чтобы получить Client ID и Client Secret (см. файл GetClientId.md).

### Шаг 2: Добавление зависимостей

Добавьте необходимые зависимости в ваш `pom.xml`:

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity6</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### Шаг 3: Конфигурационный класс

Создайте конфигурационный класс для настройки OAuth2 с GitHub:

```java
package com.example.oauthconfig;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.OAuth2AuthorizationServerConfiguration;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public UserDetailsService userDetailsService() {
        var users = new InMemoryUserDetailsManager();
        users.createUser(
                User.withDefaultPasswordEncoder()
                        .username("user")
                        .password("password")
                        .roles("USER")
                        .build()
        );
        return users;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((requests) -> requests
                        .requestMatchers("/", "/login").permitAll()
                        .anyRequest().authenticated())
                .formLogin(Customizer.withDefaults()) // Включаем форму логина
                .oauth2Login(Customizer.withDefaults()); // Включаем OAuth2 Login
        return http.build();
    }
}
```

### Шаг 4: Конфигурационные свойства

Создайте файл `application.properties` в директории `src/main/resources` и добавьте туда следующие строки:

```properties
spring.security.oauth2.client.registration.github.client-id=${YOUR_CLIENT_ID}
spring.security.oauth2.client.registration.github.client-secret=${YOUR_CLIENT_SECRET}
spring.security.oauth2.client.provider.github.authorization-uri=https://github.com/login/oauth/authorize
spring.security.oauth2.client.provider.github.token-uri=https://github.com/login/oauth/access_token
spring.security.oauth2.client.provider.github.user-info-uri=https://api.github.com/user
spring.security.oauth2.client.provider.github.user-name-attribute=login
```

Замените `YOUR_CLIENT_ID` и `YOUR_CLIENT_SECRET` на значения, полученные при регистрации приложения в GitHub.

### Шаг 5: Контроллеры и представления

Создайте контроллер для обработки запросов:

```java
package com.example.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String home(Model model) {
        model.addAttribute("message", "Welcome to the Home Page!");
        return "home";
    }
}
```

И создайте представление `home.html` в директории `src/main/resources/templates`:

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Home</title>
</head>
<body>
<h1 th:text="${message}">Message</h1>
<div sec:authorize="isAuthenticated()">
    Logged in as: <span sec:authentication="principal.name"></span>
</div>
<div sec:authorize="!isAuthenticated()">
    <a href="/oauth2/authorization/github">Login with GitHub</a>
</div>
</body>
</html>
```

### Шаг 6: Запуск приложения

Соберите проект и запустите его. Вы можете сделать это через IDE или командную строку:

```bash
mvn spring-boot:run
```

### Шаг 7: Тестирование

Перейдите по адресу `http://localhost:8080/`. Нажмите на ссылку **Login with GitHub** и следуйте инструкциям для
авторизации через GitHub.

Если всё сделано правильно, после успешной авторизации вы вернётесь на главную страницу с сообщением о том, что вы вошли
в систему.
