# Spring Security

**Índice**

1. [Introduccion a Spring Security](#id1)
2. [Spring Security](#id2)

<div id='id1' />

## 1. Introduccion a Spring Security

<div id='id2' />

## 2. Spring Security

1. Instalamos la dependencia de **_spring-boot-spring-security_**

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

2. Creamos la entidad para el manejo del usuario **_User_**

```java
@Entity
@Data
@Table(name = "tbl_user")
@AllArgsConstructor
@NoArgsConstructor
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    private String password;
}
```

3. Esta entidad **_User_** implementa la clase que se llama **_UserDetails_** que pertenece al **_Core_** de **_Spring Security_**:

`implements UserDetails` con todos sus metodos implementados.

```java
public class User implements UserDetails {...}
```

4. Creamos una **_Enum_** llamado **_Role_**:

```java
public enum Role { USER, ADMIN }
```

```java
public class User implements UserDetails {
    // demas campos
    ...
    @Enumerated(EnumType.ORDINAL)
    private Role role;
    ...

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority(role.name()));
    }
    @Override
    public String getUsername() {
        return this.email;
    }

    @Override
    public String getPassword() {
        return this.password;
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

5. Creamos el package **_repository_** y dentro creamos el **_UserRepository_**

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findUserByEmail(String email);
}
```

6. Creamos una clase para el filtro **_JwtFilter_**, creamos el package **_config_**

```java
@Component
@RequiredArgsConstructor
public class JwtFilter extends OncePerRequestFilter {

    private final UserDetailsService userDetailsService;
    private final JwtService jwtService;

    @Override
    protected void doFilterInternal(@NonNull HttpServletRequest request, @NonNull HttpServletResponse response, @NonNull FilterChain filterChain) throws ServletException, IOException {
        final String authorizationHeader = request.getHeader("Authorization");
        final String jwt;
        final String userEmail;

        if (authorizationHeader == null && !authorizationHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }
        jwt = authorizationHeader.substring(7);
        userEmail = jwtService.getUserEmail(jwt);
        if (userEmail != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = this.userDetailsService.loadUserByUsername(userEmail);
            if (jwtService.validateTken(jwt, userDetails)) {
                UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities());
            }

            authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
            SecurityContextHolder.getContext().setAuthentication(authenticationToken);
        }
        filterChain.doFilter(request, response);

    }
}
```

`NOTA:` no se nos olvide en este paso haber instalado estas dependecias para el funcionamiento correctamente, se pueden buscar en [Maven repository](https://mvnrepository.com/)

```xml
<dependency>
	<groupId>io.jsonwebtoken</groupId>
	<artifactId>jjwt-api</artifactId>
	<version>0.11.5</version>
</dependency>
<dependency>
	<groupId>io.jsonwebtoken</groupId>
	<artifactId>jjwt-impl</artifactId>
	<version>0.11.5</version>
</dependency>
<dependency>
	<groupId>io.jsonwebtoken</groupId>
	<artifactId>jjwt-jackson</artifactId>
	<version>0.11.5</version>
</dependency>
```

**_jjwt-api_** contiene interfaces para trabajar con JWT, nos proporciana abstracciones

**_jjwt-impl_** contiene las implementaciones de la anterior

**_jjwt-jackson_** es para la serializacion

7. Creamos el filtro del servicio para JWT y le pasamos el **_secret key_** del algun **_secret key generator_** como metodo de prueba, en el package **_config_** creamos la clase **_JwtService_**.

```java
@Service
public class JwtService {
    private static final String SECRET_KEY = "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08";

    public String getUserEmail(String token) {
        return getClaim(token, Claims::getSubject);
    }

    public <T> T getClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = getAllClaims(token);

        return claimsResolver.apply(claims);
    }

    private Claims getAllClaims(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(getSignIngKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    private Key getSignIngKey() {
        byte[] keyBytes = Decoders.BASE64.decode(SECRET_KEY);
        return Keys.hmacShaKeyFor(keyBytes);
    }

    public boolean validateTken(String token, UserDetails userDetails) {
        final String userEmail = getUserEmail(token);

        return (userEmail.equals(userDetails.getUsername()) && !itTokenExpired(token));
    }

    private boolean itTokenExpired(String token) {
        return getExpiration(token).before(new Date());
    }

    private Date getExpiration(String token) {
        return getClaim(token, Claims::getExpiration);
    }
}
```

8. Generamos el token en la clase **_JwtService_**

```java
@Service
public class JwtService {
    ...
    public String generateToken(UserDetails userDetails) {
        return generateToken(new HashMap<>(), userDetails);
    }

    public String generateToken(Map<String, Object> extraClaims, UserDetails userDetails) {
        return Jwts.builder()
                .setClaims(extraClaims)
                .setSubject(userDetails.getUsername())
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 24))
                .signWith(getSignIngKey(), SignatureAlgorithm.ES256)
                .compact();
    }
    ...
}
```

9. Creamos **_AppConfig_** dentro de el package **_config_**.

```java
@Configuration
@RequiredArgsConstructor
public class AppConfig {

    private final UserRepository userRepository;

    @Bean
    public UserDetailsService userDetailsService() {
        return username -> userRepository.findUserByEmail(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));
    }
}
```

10. Creamos **_SecurityConfig_** dentro de el package **_config_**.

**_nota:_** **_`@EnableWebSecurity`_** es importante porque estamos habilitando seguridades dentro de nuestra **_ApiRestfull_**.

Aca es donde vamos a lanzar nuestro filtro de **_JwtFilter_**

```java
@ConcreteProxy
@EnableWebSecurity
@RequiredArgsConstructor
@EnableMethodSecurity
public class SecurityConfig {
    private final JwtFilter jwtFilter;
    private final AuthenticationProvider authenticationProvider;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(auth -> auth.requestMatchers("")
                        .permitAll()
                        .anyRequest().authenticated())
                .sessionManagement(sess -> sess.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .authenticationProvider(authenticationProvider)
                .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
        return httpSecurity.build();
    }
}
```

En **_AppConfig_** creamos los **_`@Bean`_** **_authenticatorProvider_** y **_passwordEncoder_**

```java
...
public class AppConfig {
    ...

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();
        authenticationProvider.setUserDetailsService(userDetailsService());
        authenticationProvider.setPasswordEncoder(passwordEncoder());
        return authenticationProvider;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

11. Creamos **_AuthenticationController_** con los endpoint para registrar y authenticar usuarios. En el package **_/controllers_** creamos el **_AuthenticationController_**.

```java
@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthenticationController {

    @PostMapping("/register")
    public ResponseEntity<AuthResponse> register(@RequestBody RegisterRequest request){
        return ResponseEntity.ok(authService.register(request));
    }

    @PostMapping("/authenticate")
    public ResponseEntity<AuthResponse> authenticate(@RequestBody AuthenticationRequest request){
        return ResponseEntity.ok(authService.authenticate(request));
    }
}
```

Para continuar con las buenas practicas en el mismo paquete se crea otro paquete llamado **_`.controllers.models`_**. Dentro creamos varias clases importantes.

- **_AuthenticationRequest_** ()
- **_RegisterRequest_** ()
- **_AuthResponse_** ()

**AuthenticationRequest**

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class AuthenticationRequest {
    private String email;
    private String password;
}
```

**RegisterRequest**

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class RegisterRequest {
    private String firstName;
    private String lastName;
    private String email;
    private String password;
}
```

**AuthResponse**

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class AuthResponse {
    private String token;
}
```
