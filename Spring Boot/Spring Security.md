# Spring Security

**Índice**

1. [Introduccion a Spring Security](#id1)
   - [Configurar la Seguridad](#id1.1)
2. [Spring Security con JWT](#id2)
   - [Listas blancas (white list)](#id2.1)

<div id='id1' />

## 1. Introduccion a Spring Security

<div id='id1.1' />

### Configurar la Seguridad

Una vez instaladas las dependecias de **_Spring Security_** se nos agrega un paquete de seguridad en la web app con una autenticacion. Esto lleva mucha configuracion.

Se crea una clase llamada **_SpringConfig_** es donde ira toda la configuracion.

Con los `@Bean` @Configuration le estamos diciendo que esta sera una clase destinada a la configuracion y el `@Bean` @EnableWebSecurity le estamos diciendo que sera para la seguridad de la aplicacion.

**_SpringConfig_**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain configure(HttpSecurity httpSecurity) throws Exception {
        return httpSecurity.build();
    }
}
```

#### Propiedades necesarias de **_httpSecurity_** para la implementacion de la clase de seguridad.

**_httpSecurity_**

- **csrf()** -> (Cross-Site Request Forgery) es una vulnerabilidad que tienen las aplicaciones web.
  - csrf().disable() -> si vas a trabajar con formularios se desabilite
- **authorizeHttpRequests(auth -> {})** -> Aca es donde van a pasarle las url que se quieran o no proteger
  - **_auth.requestMatchers("").permitAll()_** -> son las **_url_** que no requiren estar autenticado
  - **_authorize.anyRequest().authenticated()_** -> aca si se requiere estar autenticado
  - **_formLogin((config) -> config.successHandler(successHandler()))_** -> **_url_** donde se va a redirigir luego de hacer **_login_**.
  - **_sessionManagement(sm -> {})_** -> para manejar las secciones dentro de la app.
    - **_sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS)_**
      - **_ALWAYS_** -> va a crear una seccion siempre no haya ninguna
      - **_IF_REQUIRED_** -> crea una seccion si es necesario
      - **_NEVER_** -> No crea seccion, pero si existe seccion la utiliza
      - **_STATELESS_** -> No crea ninguna seccion, todas las solicitudes las trabaja de formas independientes
    - **_m.invalidSessionUrl("/login")_** -> donde se va a redirigir si falla la authenticacion
    - **_sm.maximumSessions(1)_** -> numero maximo de sessiones
      - **_sm.maximumSessions(1)_**
        - **_...expiredUrl("/login")_** -> hacia donde se redirige cuando se expire la session
        - **_...sessionRegistry(sessionRegistry())_** -> nos va ayudar a obtener los datos de la session.
      - **_sm.sessionFixation()_** -> es otra vulnerabilidad que tienen las web app
        - **_...migrateSession()_** -> es una estrategia para los ataques de fijacion de seccion, registra otro id por cada seccion (**_recomendada_**)
        - **_...newSession()_** -> crea otra seccion sin copiar los datos de otras secciones abiertas
        - **_...none()_** -> inhabilita la seguridad en contra de la fijacion de seccion (**_no recomendada_**)
  - **_httpBasic()_** -> se usa cuando la seguridad no es tan importante. Porque se van a enviar las credenciales en el header de la peticion.

**_SecurityConfig_**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity
                .authorizeHttpRequests((authorize) -> {
                            authorize.requestMatchers(whiteList()).permitAll();
                            authorize.anyRequest().authenticated();
                })
                .formLogin((config) -> config.successHandler(successHandler()))
                .sessionManagement(sm -> {
                    sm.sessionCreationPolicy(SessionCreationPolicy.ALWAYS);
                    sm.invalidSessionUrl("/login");
                    sm.maximumSessions(1).expiredUrl("/login").sessionRegistry(sessionRegistry());
                    sm.sessionFixation().migrateSession();

                })
        ;
        return httpSecurity.build();
    }

    public String[] whiteList(){
        return new String[]{
                "/api/greeting/sayHelloPublic",
        };
    }

    public AuthenticationSuccessHandler successHandler(){
        return ((request, response, auth) -> {
            response.sendRedirect("/api/greeting/sayHelloProtected");
        });
    }

    @Bean
    public SessionRegistry sessionRegistry(){
        return new SessionRegistryImpl();
    }
}
```

### Como conocer la session del usuario

```java
@GetMapping("/session")
public ResponseEntity<?> getDetailsSession() {
    String sessionId = "";
    User userObject = null;

    List<Object> sessions = sessionRegistry.getAllPrincipals();

    for(Object session : sessions) {
        if (session instanceof User) {
            userObject = (User) session;
        }

        List<SessionInformation> sessionInformations = sessionRegistry.getAllSessions(session, false);
        for(SessionInformation sessionInformation : sessionInformations) {
            sessionId = sessionInformation.getSessionId();
        }
    }

    Map<String, Object> response = new HashMap<>();
    response.put("response", "Hello World");
    response.put("sessionId", sessionId);
    response.put("sessionUser", userObject);

    return ResponseEntity.ok(response);
}
```

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

12. Creamos el servicio para finalizar con la authentication. En el paquete **_.services_** creamos la **_interface_** -> **_AuthService_**

**_AuthService_**

```java
public interface AuthService {
    AuthResponse register(RegisterRequest request);
    AuthResponse authenticate(AuthenticationRequest request);
}
```

Para continuar con las buenas practicas en el mismo package creamos la implementacion del servicio con la clase -> **_AuthServiceImpl_** que implemente a **_AuthService_**.

> **_AuthServiceImpl_** implements **_AuthService_**

```java
@Service
@RequiredArgsConstructor
public class AuthServiceImpl implements AuthService {

    private final PasswordEncoder passwordEncoder;
    private final UserRepository userRepository;
    private final JwtService jwtService;
    private final AuthenticationManager authenticationManager;

    @Override
    public AuthResponse register(RegisterRequest request) {
        var user = User.builder()
                .firstName(request.getFirstName())
                .lastName(request.getLastName())
                .email(request.getEmail())
                .password(passwordEncoder.encode(request.getPassword()))
                .build();
        userRepository.save(user);
        var jwtToken = jwtService.generateToken(user);
        return AuthResponse.builder().token(jwtToken).build();
    }

    @Override
    public AuthResponse authenticate(AuthenticationRequest request) {
        authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                        request.getEmail(),
                        request.getPassword()
                )
        );
        var user = userRepository.findUserByEmail(request.getEmail()).orElseThrow();
        var jwtToken = jwtService.generateToken(user);
        return AuthResponse.builder().token(jwtToken).build();
    }
}
```

**Detalle importante:** necesitaremos del `@Bean` **_AuthenticationManager_** de **_Spring_** para terminar de configurar todo en el servicio.

```java
...
public class AppConfig {
    ...

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

Con esto ya quedaria todo configurado y funcionando.

<div id='id1' />

## Listas blancas (white list)

Son los **_endpoin_** que se puede exponer a cualquier usuario sin necesidad de ser autenticado o autorizado en la aplicacion.

En la clase -> **_SecurityConfig_** creamos el listado de los endpoint que vamos acceder sin estar autenticados llamados **_White List_**.

**config/SecurityConfig**

```java
...
public class SecurityConfig {
    ...
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
        ...
        auth.requestMatchers("") // ***before***
        auth.requestMatchers(publicEndpoints()) // ***white list***
        ...
    }


    private RequestMatcher publicEndpoints() {
        return new OrRequestMatcher(
                new AntPathRequestMatcher("/api/greeting/helloWorldPublic")
        );
    }
}
```

**controllers/HelloController**

```java
@RestController
@RequestMapping("/api/greeting")
public class HelloController {

    @GetMapping("/helloWorldPublic")
    public String hello(){
        return "Hello World";
    }

    @GetMapping("/helloWorldProtected")
    public String helloProtected(){
        return "Hello World protected";
    }
}
```
