# Project Structure

## Single-Service Layout

```
order-service/
├── src/
│   ├── main/
│   │   ├── java/com/shop/order/
│   │   │   ├── OrderServiceApplication.java   ← @SpringBootApplication entry point
│   │   │   ├── config/                        ← @Configuration, SecurityConfig, AsyncConfig
│   │   │   ├── controller/                    ← HTTP only — no business logic
│   │   │   ├── service/                       ← business rules, @Transactional
│   │   │   ├── repository/                    ← data access only
│   │   │   ├── entity/                        ← JPA @Entity classes
│   │   │   ├── dto/                           ← request/response records, MapStruct targets
│   │   │   ├── exception/                     ← custom exceptions + @RestControllerAdvice
│   │   │   ├── event/                         ← ApplicationEvent subclasses
│   │   │   └── client/                        ← @FeignClient interfaces
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-prod.yml
│   │       └── db/migration/                  ← Flyway V1__, V2__ scripts
│   └── test/
│       └── java/com/shop/order/
│           ├── controller/                    ← @WebMvcTest
│           ├── service/                       ← @ExtendWith(MockitoExtension.class)
│           └── repository/                    ← @DataJpaTest + Testcontainers
├── pom.xml
└── Dockerfile
```

Hard rule: controllers translate HTTP ↔ Java only. Services own all business rules. Repositories touch only the database. Crossing these boundaries is the most common structural mistake in Spring projects.

## Multi-Module Maven Project

Use when multiple services share domain types or utilities. Each module is a separate JAR; the parent coordinates versions.

```
shop/
├── pom.xml                   ← parent: version declarations, no code
├── shop-common/              ← shared DTOs, exceptions, utils — zero Spring deps here
│   └── pom.xml
├── shop-user-service/
│   └── pom.xml               ← depends on shop-common
├── shop-order-service/
│   └── pom.xml
└── shop-payment-service/
    └── pom.xml
```

---

# Maven

## pom.xml — Full Reference

```xml
<!-- shop-order-service/pom.xml -->
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <!-- Inherits dependency versions, plugin defaults, encoding, Java version -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
    </parent>

    <groupId>com.shop</groupId>
    <artifactId>shop-order-service</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>21</java.version>
        <spring-cloud.version>2023.0.3</spring-cloud.version>
        <mapstruct.version>1.6.0</mapstruct.version>
    </properties>

    <!-- Import Spring Cloud BOM — manages all spring-cloud-* versions in one line -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- ── Core ─────────────────────────────────────────── -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!-- pulls in: Spring MVC, embedded Tomcat, Jackson -->
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <!-- pulls in: Hibernate, Spring Data JPA, HikariCP -->
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
            <!-- pulls in: Hibernate Validator, Jakarta Validation API -->
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- ── Spring Cloud ──────────────────────────────────── -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
        </dependency>

        <!-- ── Database ──────────────────────────────────────── -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>   <!-- only needed at runtime, not compile -->
        </dependency>
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-database-postgresql</artifactId>
        </dependency>

        <!-- ── Messaging ─────────────────────────────────────── -->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>

        <!-- ── Cache ─────────────────────────────────────────── -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- ── Observability ─────────────────────────────────── -->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
        </dependency>
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-tracing-bridge-brave</artifactId>
        </dependency>
        <dependency>
            <groupId>io.zipkin.reporter2</groupId>
            <artifactId>zipkin-reporter-brave</artifactId>
        </dependency>

        <!-- ── Mapping & boilerplate ─────────────────────────── -->
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${mapstruct.version}</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>   <!-- compile-only; not bundled in the fat JAR -->
        </dependency>

        <!-- ── API Docs ───────────────────────────────────────── -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.6.0</version>
        </dependency>

        <!-- ── Test ──────────────────────────────────────────── -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <!-- pulls in: JUnit 5, Mockito, AssertJ, MockMvc, Testcontainers support -->
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Packages everything into a single executable fat JAR -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <!-- Lombok generated code is already compiled — don't bundle the JAR -->
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>

            <!-- Annotation processors — Lombok MUST be listed before MapStruct -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </path>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${mapstruct.version}</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>

            <!-- maven-surefire runs unit tests (*Test.java) — no Spring context, fast -->
            <!-- maven-failsafe runs integration tests (*IT.java) — full context, Testcontainers -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

    <profiles>
        <profile>
            <id>prod</id>
            <properties>
                <spring.profiles.active>prod</spring.profiles.active>
            </properties>
        </profile>
    </profiles>
</project>
```

**Parent POM for multi-module (versions centralised here):**

```xml
<!-- shop/pom.xml -->
<project>
    <groupId>com.shop</groupId>
    <artifactId>shop</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
    </parent>

    <modules>
        <module>shop-common</module>
        <module>shop-user-service</module>
        <module>shop-order-service</module>
        <module>shop-payment-service</module>
    </modules>

    <properties>
        <java.version>21</java.version>
        <spring-cloud.version>2023.0.3</spring-cloud.version>
        <mapstruct.version>1.6.0</mapstruct.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- Internal module version declared once -->
            <dependency>
                <groupId>com.shop</groupId>
                <artifactId>shop-common</artifactId>
                <version>${project.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

**Common Maven commands:**

```bash
./mvnw clean install              # build all modules, run unit tests
./mvnw clean verify               # build + unit + integration tests (Testcontainers fires here)
./mvnw clean install -DskipTests  # build without any tests
./mvnw spring-boot:run            # run locally without packaging
./mvnw dependency:tree            # show full dependency graph — find version conflicts
./mvnw versions:display-dependency-updates  # what's outdated
```

---

# Spring Boot

## IoC — Inversion of Control

Instead of your code creating objects (`new MyService()`), Spring creates and wires them for you. The framework owns the object lifecycle — you just declare what you need.

**Stereotype annotations** — mark a class so Spring picks it up:

| Annotation | Typical layer | Extra behavior |
|---|---|---|
| `@Component` | Generic bean | — |
| `@Service` | Business logic | — |
| `@Repository` | Data access | Translates SQL exceptions → `DataAccessException` |
| `@Controller` | Web layer (returns views) | — |
| `@RestController` | Web layer (returns JSON) | = `@Controller` + `@ResponseBody` on every method |

```java
@Service
public class OrderService {
    private final OrderRepository repo;   // Spring injects this

    // Constructor injection: preferred — makes dependencies explicit and testable
    public OrderService(OrderRepository repo) {
        this.repo = repo;
    }
}
```

**Manual beans** — use when you don't own the class (third-party library):

```java
@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper()
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    }
}
```

### Scopes

| Scope | Lifetime | How to declare |
|---|---|---|
| `singleton` | One instance per container — **default** | (none needed) |
| `prototype` | New instance on every injection | `@Scope("prototype")` |
| `request` | One per HTTP request | `@RequestScope` |
| `session` | One per HTTP session | `@SessionScope` |

```java
@Component
@Scope("prototype")          // each caller gets its own fresh instance
public class CsvReportBuilder { ... }
```

---

## AoP — Aspect-Oriented Programming

Cross-cutting concerns (logging, auth, metrics, transactions) would otherwise repeat across every class. AoP intercepts method calls via a **proxy** and runs extra logic without touching the original code.

**Vocabulary:**

| Term | Meaning |
|---|---|
| **Join point** | A point in execution (always a method call in Spring) |
| **Pointcut** | Expression that selects which join points to intercept |
| **Advice** | The code that runs at the selected join point |
| **Aspect** | Class that bundles a pointcut + its advice |

**Advice types:**

| Annotation | Runs |
|---|---|
| `@Before` | Before the method executes |
| `@AfterReturning` | After the method returns normally |
| `@AfterThrowing` | After the method throws an exception |
| `@After` | Always after (like `finally`) |
| `@Around` | Wraps the method — full control over invocation |

```java
@Aspect
@Component
public class LoggingAspect {

    // Intercepts every public method in any class inside the service package
    @Around("execution(* com.example.service.*.*(..))")
    public Object logExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();               // call the actual method
        long elapsed = System.currentTimeMillis() - start;
        log.info("{} completed in {}ms", pjp.getSignature(), elapsed);
        return result;
    }

    // Only fires when the method throws — logs the error then re-throws
    @AfterThrowing(pointcut = "execution(* com.example.service.*.*(..))", throwing = "ex")
    public void logException(JoinPoint jp, Exception ex) {
        log.error("Exception in {}: {}", jp.getSignature(), ex.getMessage());
    }
}
```

> Spring AoP uses JDK dynamic proxies (interface-based) or CGLIB (class-based). Self-invocation — calling `this.method()` inside the same class — bypasses the proxy, so advice won't fire.

---

## MVC

### Controller

```java
@RestController                               // @Controller + @ResponseBody on every method
@RequestMapping("/api/v1/orders")             // base path for all methods in this class
public class OrderController {

    private final OrderService service;

    public OrderController(OrderService service) {
        this.service = service;
    }

    // GET /api/v1/orders/42
    @GetMapping("/{id}")
    public ResponseEntity<OrderDto> getById(@PathVariable Long id) {
        return service.findById(id)           // returns Optional<OrderDto>
            .map(ResponseEntity::ok)          // present  → 200 OK + body
            .orElse(ResponseEntity.notFound().build()); // absent → 404
    }

    // POST /api/v1/orders
    // Body: { "customerId": 7, "items": [...] }
    @PostMapping
    public ResponseEntity<OrderDto> create(@RequestBody @Valid CreateOrderRequest req) {
        OrderDto created = service.create(req);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    // DELETE /api/v1/orders/42
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        service.delete(id);
        return ResponseEntity.noContent().build();   // 204
    }

    // GET /api/v1/orders?status=PENDING&page=0&size=20
    @GetMapping
    public ResponseEntity<Page<OrderDto>> list(
            @RequestParam(required = false) String status,
            Pageable pageable) {
        return ResponseEntity.ok(service.findAll(status, pageable));
    }
}
```

**Key annotations at a glance:**

| Annotation | What it does |
|---|---|
| `@PathVariable` | Extracts `{id}` from the URL path |
| `@RequestBody` | Deserializes the JSON body → Java object |
| `@RequestParam` | Extracts `?status=PENDING` from the query string |
| `@Valid` | Triggers Bean Validation on the request object |

---

### Service

Business logic lives here. Keep controllers thin — they only translate HTTP ↔ Java. Services own the rules.

`@Transactional` — Spring wraps the method in a DB transaction; rolls back automatically on unchecked exceptions.

```java
@Service
@Transactional          // all public methods are transactional by default
public class OrderService {

    private final OrderRepository orderRepo;
    private final CustomerRepository customerRepo;

    public OrderService(OrderRepository orderRepo, CustomerRepository customerRepo) {
        this.orderRepo = orderRepo;
        this.customerRepo = customerRepo;
    }

    @Transactional(readOnly = true)    // hint to DB: no writes → can use read replica
    public Optional<OrderDto> findById(Long id) {
        return orderRepo.findById(id).map(OrderDto::from);
    }

    public OrderDto create(CreateOrderRequest req) {
        Customer customer = customerRepo.findById(req.customerId())
            .orElseThrow(() -> new EntityNotFoundException("Customer " + req.customerId()));

        Order order = Order.builder()
            .customer(customer)
            .status(OrderStatus.PENDING)
            .items(req.items().stream().map(Item::from).toList())
            .build();

        return OrderDto.from(orderRepo.save(order));
    }

    public void delete(Long id) {
        if (!orderRepo.existsById(id)) {
            throw new EntityNotFoundException("Order " + id);
        }
        orderRepo.deleteById(id);
    }

    @Transactional(readOnly = true)
    public Page<OrderDto> findAll(String status, Pageable pageable) {
        if (status != null) {
            return orderRepo.findByStatus(OrderStatus.valueOf(status), pageable)
                            .map(OrderDto::from);
        }
        return orderRepo.findAll(pageable).map(OrderDto::from);
    }
}
```

---

### Repository

Extends Spring Data JPA interfaces — no SQL boilerplate for standard operations. Spring generates the implementation at startup.

```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {

    // Spring derives the query from the method name
    Page<Order> findByStatus(OrderStatus status, Pageable pageable);

    // Custom JPQL when derived queries get too verbose
    @Query("SELECT o FROM Order o WHERE o.customer.id = :customerId AND o.status = :status")
    List<Order> findByCustomerAndStatus(@Param("customerId") Long customerId,
                                        @Param("status") OrderStatus status);

    // Native SQL when you need DB-specific features
    @Query(value = "SELECT * FROM orders WHERE created_at > NOW() - INTERVAL '7 days'",
           nativeQuery = true)
    List<Order> findRecentOrders();
}
```

**`JpaRepository<T, ID>` gives you for free:**

| Method | What it does |
|---|---|
| `save(entity)` | Insert or update |
| `findById(id)` | Returns `Optional<T>` |
| `findAll(pageable)` | Paginated fetch |
| `deleteById(id)` | Delete by primary key |
| `existsById(id)` | Boolean existence check |
| `count()` | Row count |

---

## Spring Security

Two distinct problems:
- **Authentication** — who are you? (verify identity)
- **Authorization** — what can you do? (enforce permissions)

Every request passes through a **filter chain**. Spring Security inserts its own filters before your code runs.

### SecurityFilterChain

The single bean that defines all security rules:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)           // stateless API — no CSRF needed
            .sessionManagement(sm ->
                sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()  // public endpoints
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated())               // everything else requires login
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();   // never store plain-text passwords
    }
}
```

### JWT Authentication Flow

```
Client                          Server
  |                               |
  |-- POST /api/auth/login -----> |  1. verify credentials
  |                               |  2. sign JWT with secret key
  |<-- { token: "eyJ..." } ------ |
  |                               |
  |-- GET /api/orders             |
  |   Authorization: Bearer eyJ..|  3. JwtAuthFilter extracts token
  |                               |  4. validate signature + expiry
  |                               |  5. set SecurityContext → request proceeds
  |<-- 200 OK ------------------- |
```

**JWT filter** — runs before every request:

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest req,
                                    HttpServletResponse res,
                                    FilterChain chain) throws ServletException, IOException {

        String header = req.getHeader("Authorization");
        if (header == null || !header.startsWith("Bearer ")) {
            chain.doFilter(req, res);    // no token — pass through (will hit auth rules later)
            return;
        }

        String token = header.substring(7);
        String username = jwtService.extractUsername(token);

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails user = userDetailsService.loadUserByUsername(username);
            if (jwtService.isValid(token, user)) {
                UsernamePasswordAuthenticationToken auth =
                    new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities());
                auth.setDetails(new WebAuthenticationDetailsSource().buildDetails(req));
                SecurityContextHolder.getContext().setAuthentication(auth);
            }
        }
        chain.doFilter(req, res);
    }
}
```

**JwtService** — sign and validate tokens:

```java
@Service
public class JwtService {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiry-ms:86400000}")   // default 24 h
    private long expiryMs;

    public String generate(UserDetails user) {
        return Jwts.builder()
            .subject(user.getUsername())
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + expiryMs))
            .signWith(getKey())
            .compact();
    }

    public boolean isValid(String token, UserDetails user) {
        return extractUsername(token).equals(user.getUsername()) && !isExpired(token);
    }

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    private boolean isExpired(String token) {
        return extractClaim(token, Claims::getExpiration).before(new Date());
    }

    private <T> T extractClaim(String token, Function<Claims, T> resolver) {
        Claims claims = Jwts.parser().verifyWith(getKey()).build()
                            .parseSignedClaims(token).getPayload();
        return resolver.apply(claims);
    }

    private SecretKey getKey() {
        return Keys.hmacShaKeyFor(Decoders.BASE64.decode(secret));
    }
}
```

### UserDetailsService

Spring calls this to load the user during authentication:

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UserRepository userRepo;

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        return userRepo.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + email));
        // User entity implements UserDetails — returns username, password, roles
    }
}
```

### Method-Level Security

Fine-grained access control directly on service methods. Enable with `@EnableMethodSecurity`.

```java
@Service
public class OrderService {

    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
    public List<OrderDto> findByUser(Long userId) { ... }

    @PreAuthorize("hasRole('ADMIN')")
    public void delete(Long id) { ... }

    @PostAuthorize("returnObject.ownerId == authentication.principal.id")
    public OrderDto findById(Long id) { ... }   // checks after fetch — useful for ownership
}
```

| Annotation | Evaluated |
|---|---|
| `@PreAuthorize` | Before method executes |
| `@PostAuthorize` | After method returns (has access to `returnObject`) |
| `@Secured` | Simple role check — no SpEL, less flexible |

---

## Spring Cloud

Toolset for building distributed microservices. Each component solves one coordination problem.

```
                        ┌─────────────────┐
                        │   Config Server │  centralized config
                        └────────┬────────┘
                                 │
Client ──► API Gateway ──► Service Discovery (Eureka)
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
         Order Service     Payment Service     User Service
         (Feign client)                       (Feign client)
              │                                     │
         Circuit Breaker                       Circuit Breaker
         (Resilience4j)                        (Resilience4j)
```

### Service Discovery — Eureka

Services register themselves; others look them up by name instead of hard-coding IPs.

**Eureka Server:**
```java
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServerApplication { ... }
```

```yaml
# application.yml (Eureka Server)
server:
  port: 8761
eureka:
  client:
    register-with-eureka: false   # server doesn't register itself
    fetch-registry: false
```

**Eureka Client** (every microservice):
```java
@SpringBootApplication
@EnableDiscoveryClient
public class OrderServiceApplication { ... }
```

```yaml
# application.yml (Order Service)
spring:
  application:
    name: order-service           # the logical name used for discovery
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

---

### API Gateway — Spring Cloud Gateway

Single entry point for all clients. Handles routing, auth, rate limiting, and CORS centrally.

```yaml
# application.yml (Gateway)
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service          # lb:// = load-balanced via Eureka
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1                # removes /api before forwarding

        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/users/**
          filters:
            - AddRequestHeader=X-Internal-Request, true
```

**Global filter** — e.g. validate JWT at the gateway before any service sees the request:

```java
@Component
public class AuthGatewayFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String path = exchange.getRequest().getPath().toString();
        if (path.startsWith("/api/auth")) return chain.filter(exchange);  // skip public paths

        String header = exchange.getRequest().getHeaders().getFirst("Authorization");
        if (header == null || !jwtService.isValid(header.substring(7))) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }
}
```

---

### Config Server

Centralize all microservice config in one place (Git repo, filesystem, or Vault). Services fetch their config on startup.

**Config Server:**
```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication { ... }
```

```yaml
# application.yml (Config Server)
server:
  port: 8888
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/config-repo
          default-label: main
```

**Config Client** (every microservice):
```yaml
# bootstrap.yml (runs before application.yml)
spring:
  application:
    name: order-service
  config:
    import: configserver:http://localhost:8888
```

Config repo file naming: `{application-name}-{profile}.yml`
- `order-service.yml` — defaults
- `order-service-prod.yml` — prod overrides

---

### OpenFeign — Declarative HTTP Clients

Replace `RestTemplate` / `WebClient` boilerplate. Define an interface; Spring generates the implementation.

```java
@SpringBootApplication
@EnableFeignClients
public class OrderServiceApplication { ... }
```

```java
@FeignClient(name = "user-service")    // name matches Eureka registration
public interface UserClient {

    @GetMapping("/api/users/{id}")
    UserDto getUser(@PathVariable Long id);

    @PostMapping("/api/users/{id}/notify")
    void notify(@PathVariable Long id, @RequestBody NotificationRequest req);
}
```

```java
@Service
public class OrderService {

    private final UserClient userClient;   // injected — no HTTP code needed

    public OrderDto create(CreateOrderRequest req) {
        UserDto user = userClient.getUser(req.userId());  // Feign handles the HTTP call
        // ... build and save order
    }
}
```

---

### Circuit Breaker — Resilience4j

Prevents a failing downstream service from cascading failures across the system.

**States:**
```
CLOSED ──(failures exceed threshold)──► OPEN ──(wait duration)──► HALF_OPEN
  ▲                                                                     │
  └─────────────────(test call succeeds)───────────────────────────────┘
```

| State | Behavior |
|---|---|
| **CLOSED** | Normal — requests pass through |
| **OPEN** | Short-circuit — calls fail fast, fallback runs immediately |
| **HALF_OPEN** | Allows a few test requests to check if service recovered |

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      user-service:
        failure-rate-threshold: 50          # open after 50% of calls fail
        wait-duration-in-open-state: 10s    # stay open for 10s then try HALF_OPEN
        sliding-window-size: 10             # evaluated over last 10 calls
        permitted-number-of-calls-in-half-open-state: 3
```

```java
@Service
public class OrderService {

    private final UserClient userClient;

    @CircuitBreaker(name = "user-service", fallbackMethod = "fallbackUser")
    public UserDto getUser(Long userId) {
        return userClient.getUser(userId);
    }

    // Fallback — called when circuit is OPEN or the call throws
    private UserDto fallbackUser(Long userId, Exception ex) {
        log.warn("user-service unavailable, using stub for user {}", userId);
        return UserDto.unknown(userId);     // degrade gracefully
    }
}
```

**Combine with Feign:**
```java
@FeignClient(name = "user-service", fallback = UserClientFallback.class)
public interface UserClient {
    @GetMapping("/api/users/{id}")
    UserDto getUser(@PathVariable Long id);
}

@Component
public class UserClientFallback implements UserClient {
    @Override
    public UserDto getUser(Long id) {
        return UserDto.unknown(id);
    }
}

---

### Retry — Resilience4j

Automatically retries a failing call before giving up. Pair with Circuit Breaker: retry handles transient blips, circuit breaker handles sustained outages.

```yaml
resilience4j:
  retry:
    instances:
      user-service:
        max-attempts: 3
        wait-duration: 500ms
        retry-exceptions:
          - feign.FeignException.ServiceUnavailable
```

```java
@Retry(name = "user-service", fallbackMethod = "fallbackUser")
@CircuitBreaker(name = "user-service", fallbackMethod = "fallbackUser")
public UserDto getUser(Long userId) {
    return userClient.getUser(userId);
    // Retry fires first (up to 3 tries), then Circuit Breaker trips if still failing
}
```

---

### Distributed Tracing — Micrometer Tracing

Assigns a `traceId` that flows across every service in a request chain, so you can reconstruct the full call path in a tool like Zipkin.

```xml
<!-- pom.xml — add to every service -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>
```

```yaml
management:
  tracing:
    sampling:
      probability: 1.0       # 1.0 = trace every request (use ~0.1 in prod)
  zipkin:
    tracing:
      endpoint: http://zipkin:9411/api/v2/spans
```

Spring automatically propagates `traceId` and `spanId` through Feign headers and logs them via MDC — no manual wiring needed.

```
Gateway  ──traceId:abc──► Order Service ──traceId:abc──► User Service
                             spanId:1                       spanId:2
```

---

## Exception Handling

### @ControllerAdvice

One class catches exceptions from every controller. Keeps controllers free of try/catch.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 404 — resource not found
    @ExceptionHandler(EntityNotFoundException.class)
    public ProblemDetail handleNotFound(EntityNotFoundException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    }

    // 400 — @Valid failed
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail handleValidation(MethodArgumentNotValidException ex) {
        String detail = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, detail);
    }

    // 409 — business rule violated
    @ExceptionHandler(DuplicateResourceException.class)
    public ProblemDetail handleConflict(DuplicateResourceException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT, ex.getMessage());
    }

    // 500 — catch-all so stack traces never leak to clients
    @ExceptionHandler(Exception.class)
    public ProblemDetail handleGeneric(Exception ex) {
        log.error("Unhandled exception", ex);
        return ProblemDetail.forStatusAndDetail(
            HttpStatus.INTERNAL_SERVER_ERROR, "An unexpected error occurred");
    }
}
```

`ProblemDetail` is RFC 9457 — returns a standardized JSON error body:
```json
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "Order 42 not found"
}
```

---

## Bean Validation

`@Valid` on a `@RequestBody` triggers validation before the method runs. Failures throw `MethodArgumentNotValidException` — caught by the handler above.

**Built-in constraints:**

```java
public record CreateOrderRequest(
    @NotNull                          Long customerId,
    @NotBlank                         String deliveryAddress,
    @Size(min = 1, max = 50)          List<@Valid OrderItemRequest> items,
    @Future                           LocalDateTime scheduledFor,
    @DecimalMin("0.01") @DecimalMax("100000") BigDecimal totalAmount
) {}
```

**Custom validator** — when built-ins aren't enough:

```java
// 1. Define the annotation
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = ValidSkuValidator.class)
public @interface ValidSku {
    String message() default "Invalid SKU format";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 2. Implement the logic
public class ValidSkuValidator implements ConstraintValidator<ValidSku, String> {
    @Override
    public boolean isValid(String value, ConstraintValidatorContext ctx) {
        return value != null && value.matches("^[A-Z]{3}-\\d{6}$");  // e.g. ABC-001234
    }
}

// 3. Use it
public record OrderItemRequest(
    @ValidSku String sku,
    @Min(1)   int quantity
) {}
```

---

## @Transactional — Deep Dive

### Propagation

Controls what happens when a transactional method calls another transactional method.

| Propagation | Behavior |
|---|---|
| `REQUIRED` | Join existing transaction; create one if none exists — **default** |
| `REQUIRES_NEW` | Always suspend the current transaction and start a fresh one |
| `NESTED` | Run as a savepoint inside the current transaction; rollback only the nested part on failure |
| `SUPPORTS` | Join if one exists; run without one if not |
| `NOT_SUPPORTED` | Suspend current transaction; run without any |
| `NEVER` | Throw if a transaction exists |
| `MANDATORY` | Throw if no transaction exists |

```java
@Service
public class PaymentService {

    // REQUIRES_NEW — audit log must be saved even if the outer transaction rolls back
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logPaymentAttempt(Long orderId, String result) {
        auditRepo.save(new AuditLog(orderId, result, Instant.now()));
    }

    @Transactional
    public void processPayment(Long orderId) {
        // ... charge card (might throw)
        logPaymentAttempt(orderId, "SUCCESS");   // saved in its own transaction
    }
}
```

### Isolation Levels

Controls how a transaction sees concurrent changes from other transactions.

| Level | Dirty Read | Non-repeatable Read | Phantom Read |
|---|---|---|---|
| `READ_UNCOMMITTED` | possible | possible | possible |
| `READ_COMMITTED` | prevented | possible | possible |
| `REPEATABLE_READ` | prevented | prevented | possible |
| `SERIALIZABLE` | prevented | prevented | prevented |

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public OrderSummary generateReport(Long orderId) {
    // Two reads of the same row within this transaction always return the same data
    Order order = orderRepo.findById(orderId).orElseThrow();
    // ... more processing
    Order sameOrder = orderRepo.findById(orderId).orElseThrow(); // identical to first read
    return buildSummary(order, sameOrder);
}
```

> Default isolation is whatever the database uses (`READ_COMMITTED` for Postgres/MySQL).

### Rollback rules

```java
@Transactional(
    rollbackOn = Exception.class,          // also rollback on checked exceptions
    noRollbackOn = EmailSendException.class // don't rollback if email fails
)
public void placeOrder(CreateOrderRequest req) { ... }
```

By default Spring only rolls back on **unchecked** exceptions (`RuntimeException` and `Error`).

### Self-invocation — the silent killer

`@Transactional` works via a proxy. Calling a transactional method **from within the same class** bypasses the proxy — the transaction annotation is silently ignored.

```java
@Service
public class OrderService {

    public void processAll(List<Long> ids) {
        ids.forEach(this::process);   // calls this.process() directly — NO transaction
    }

    @Transactional
    public void process(Long id) {
        // ... this @Transactional has no effect when called from processAll above
    }
}
```

**Fix — inject self or extract to a separate bean:**

```java
// Option A: inject the proxied self
@Service
public class OrderService {

    @Autowired
    private OrderService self;   // Spring injects the proxy, not `this`

    public void processAll(List<Long> ids) {
        ids.forEach(id -> self.process(id));   // goes through proxy → transaction fires
    }

    @Transactional
    public void process(Long id) { ... }
}

// Option B (preferred): extract to a dedicated service
@Service
public class OrderBatchService {
    private final OrderService orderService;

    public void processAll(List<Long> ids) {
        ids.forEach(orderService::process);   // cross-bean call → proxy → transaction fires
    }
}
```

> Same rule applies to `@Async`, `@Cacheable`, and any AoP-proxied annotation called from within the same class.

---

## JPA — N+1 Problem

**The problem:** fetching a list of orders, then accessing `order.getItems()` for each one fires a separate query per order.

```java
// 1 query → 100 orders
List<Order> orders = orderRepo.findAll();

// 100 additional queries — one per order's items
orders.forEach(o -> process(o.getItems()));  // N+1
```

**Fix 1 — JOIN FETCH in JPQL:**

```java
@Query("SELECT o FROM Order o JOIN FETCH o.items WHERE o.status = :status")
List<Order> findWithItemsByStatus(@Param("status") OrderStatus status);
// Single query with JOIN — items loaded in one round trip
```

**Fix 2 — @EntityGraph** (keeps the repository method clean):

```java
@EntityGraph(attributePaths = {"items", "items.product"})
@Query("SELECT o FROM Order o WHERE o.status = :status")
List<Order> findWithItemsByStatus(@Param("status") OrderStatus status);
```

**Fix 3 — Batch fetching** (global setting in `application.yml`):

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 50   # groups N+1 into batches of 50 — 100 orders = 2 queries
```

> Use `JOIN FETCH` / `@EntityGraph` when you always need the association. Use batch size as a safety net for unexpected lazy loading.

---

## Configuration

### @ConfigurationProperties

Type-safe binding of `application.yml` properties to a Java class. Preferred over `@Value` for groups of related config.

```yaml
# application.yml
app:
  jwt:
    secret: my-256-bit-secret
    expiry-ms: 86400000
  mail:
    host: smtp.example.com
    port: 587
    from: noreply@example.com
```

```java
@ConfigurationProperties(prefix = "app.jwt")
@Validated                                   // enables Bean Validation on the config class
public record JwtProperties(
    @NotBlank String secret,
    @Min(3600000) long expiryMs
) {}

@ConfigurationProperties(prefix = "app.mail")
public record MailProperties(
    String host,
    int port,
    String from
) {}
```

```java
@SpringBootApplication
@EnableConfigurationProperties({JwtProperties.class, MailProperties.class})
public class Application { ... }
```

```java
@Service
public class JwtService {
    private final JwtProperties props;   // injected normally

    public String generate(UserDetails user) {
        return Jwts.builder()
            .expiration(new Date(System.currentTimeMillis() + props.expiryMs()))
            .signWith(Keys.hmacShaKeyFor(props.secret().getBytes()))
            .compact();
    }
}
```

### Profiles

Activate different config per environment:

```yaml
# application.yml — shared defaults
spring:
  datasource:
    url: jdbc:h2:mem:testdb

---
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:postgresql://prod-db:5432/orders
    username: ${DB_USER}
    password: ${DB_PASS}
```

```bash
java -jar app.jar --spring.profiles.active=prod
# or
SPRING_PROFILES_ACTIVE=prod java -jar app.jar
```

```java
@Profile("prod")
@Bean
public AuditLogger prodAuditLogger() { return new DatabaseAuditLogger(); }

@Profile("!prod")          // any profile that is NOT prod
@Bean
public AuditLogger devAuditLogger() { return new ConsoleAuditLogger(); }
```

---

## Async & Scheduling

### @Async

Runs the method in a separate thread. The caller gets back immediately.

```java
@SpringBootApplication
@EnableAsync
public class Application { ... }
```

```java
@Service
public class NotificationService {

    // Returns immediately — email sends in background thread
    @Async
    public CompletableFuture<Void> sendWelcomeEmail(String to) {
        emailClient.send(to, "Welcome!", renderTemplate("welcome"));
        return CompletableFuture.completedFuture(null);
    }

    // Caller can wait for the result if needed
    @Async
    public CompletableFuture<Report> generateReport(Long userId) {
        Report r = heavyComputation(userId);
        return CompletableFuture.completedFuture(r);
    }
}
```

> `@Async` uses a proxy — calling it from within the same class bypasses the proxy (same caveat as AoP).

**Custom thread pool** — don't use the default single-thread executor in prod:

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor exec = new ThreadPoolTaskExecutor();
        exec.setCorePoolSize(4);
        exec.setMaxPoolSize(20);
        exec.setQueueCapacity(100);
        exec.setThreadNamePrefix("async-");
        exec.initialize();
        return exec;
    }
}
```

### @Scheduled

```java
@SpringBootApplication
@EnableScheduling
public class Application { ... }
```

```java
@Component
public class ReportScheduler {

    // Fixed delay: 30s after the previous execution finishes
    @Scheduled(fixedDelay = 30_000)
    public void cleanExpiredTokens() { tokenRepo.deleteExpired(); }

    // Fixed rate: every 60s regardless of how long the task takes
    @Scheduled(fixedRate = 60_000)
    public void syncInventory() { inventorySync.run(); }

    // Cron: every day at 02:00
    @Scheduled(cron = "0 0 2 * * *")
    public void generateDailyReport() { reportService.daily(); }
}
```

Cron format: `second minute hour day-of-month month day-of-week`

### Spring Events

Decouple components: publisher knows nothing about subscribers.

```java
// 1. Define the event
public record OrderPlacedEvent(Long orderId, String customerEmail) {}

// 2. Publish it
@Service
public class OrderService {

    private final ApplicationEventPublisher publisher;

    public OrderDto create(CreateOrderRequest req) {
        Order order = orderRepo.save(build(req));
        publisher.publishEvent(new OrderPlacedEvent(order.getId(), req.email()));
        return OrderDto.from(order);
    }
}

// 3. Listen — in a completely separate class
@Component
public class OrderEventListener {

    @EventListener
    public void onOrderPlaced(OrderPlacedEvent event) {
        emailService.sendConfirmation(event.customerEmail(), event.orderId());
    }

    @EventListener
    @Async                          // don't block the publisher's thread
    public void onOrderPlacedAsync(OrderPlacedEvent event) {
        analyticsService.track(event.orderId());
    }
}
```

---

## Database Migrations — Flyway

Version-controlled schema changes. Flyway runs pending migrations automatically on startup before the app accepts traffic.

```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true    # safe for existing databases
```

File naming: `V{version}__{description}.sql` — the double underscore is required.

```
src/main/resources/db/migration/
├── V1__create_orders_table.sql
├── V2__add_status_index.sql
└── V3__add_customer_email_column.sql
```

```sql
-- V1__create_orders_table.sql
CREATE TABLE orders (
    id          BIGSERIAL PRIMARY KEY,
    customer_id BIGINT        NOT NULL,
    status      VARCHAR(20)   NOT NULL DEFAULT 'PENDING',
    total       NUMERIC(10,2) NOT NULL,
    created_at  TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);

-- V2__add_status_index.sql
CREATE INDEX idx_orders_status ON orders(status);

-- V3__add_customer_email_column.sql
ALTER TABLE orders ADD COLUMN customer_email VARCHAR(255);
```

> Never edit a migration that has already run in any environment — Flyway checksums each file and will refuse to start if one changes. Add a new migration instead.

---

## Actuator & Observability

### Spring Boot Actuator

Exposes operational endpoints over HTTP. Add `spring-boot-starter-actuator`.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
  endpoint:
    health:
      show-details: when-authorized   # full detail only for authenticated users
```

| Endpoint | What it shows |
|---|---|
| `/actuator/health` | App health + component status |
| `/actuator/info` | Build metadata, git commit |
| `/actuator/metrics` | All Micrometer metrics |
| `/actuator/prometheus` | Metrics in Prometheus scrape format |
| `/actuator/env` | All resolved config properties |
| `/actuator/loggers` | View/change log levels at runtime |

**Custom HealthIndicator:**

```java
@Component
public class PaymentGatewayHealthIndicator implements HealthIndicator {

    private final PaymentGatewayClient client;

    @Override
    public Health health() {
        try {
            boolean reachable = client.ping();
            return reachable
                ? Health.up().withDetail("gateway", "Stripe").build()
                : Health.down().withDetail("reason", "ping failed").build();
        } catch (Exception ex) {
            return Health.down(ex).build();
        }
    }
}
```

### Micrometer — Custom Metrics

Micrometer is the metrics facade — it works with Prometheus, Datadog, CloudWatch, etc.

```java
@Service
public class OrderService {

    private final Counter ordersCreated;
    private final Timer   orderProcessingTime;

    public OrderService(MeterRegistry registry) {
        this.ordersCreated = Counter.builder("orders.created")
            .description("Total orders placed")
            .tag("env", "prod")
            .register(registry);

        this.orderProcessingTime = Timer.builder("orders.processing.time")
            .description("Time to process an order")
            .register(registry);
    }

    public OrderDto create(CreateOrderRequest req) {
        return orderProcessingTime.record(() -> {
            OrderDto dto = doCreate(req);
            ordersCreated.increment();
            return dto;
        });
    }
}
```

Prometheus scrapes `/actuator/prometheus` and you query with:
```promql
rate(orders_created_total[5m])          # orders per second over last 5 min
histogram_quantile(0.99, orders_processing_time_seconds_bucket)  # p99 latency
```

---

## Testing

### Test Slices — The Mental Model

| Annotation | What loads | Use for |
|---|---|---|
| `@SpringBootTest` | Full application context | Integration tests, end-to-end |
| `@WebMvcTest` | Web layer only (controllers, filters, advice) | Controller logic, request mapping |
| `@DataJpaTest` | JPA layer only (repositories, entities) | Query correctness |

### @WebMvcTest — Controller Slice

```java
@WebMvcTest(OrderController.class)      // only loads OrderController + web infrastructure
class OrderControllerTest {

    @Autowired MockMvc mvc;
    @MockBean  OrderService service;    // service is mocked — not in the web slice

    @Test
    void getById_returnsOrder() throws Exception {
        given(service.findById(42L))
            .willReturn(Optional.of(new OrderDto(42L, "PENDING", BigDecimal.TEN)));

        mvc.perform(get("/api/v1/orders/42"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(42))
            .andExpect(jsonPath("$.status").value("PENDING"));
    }

    @Test
    void getById_returns404WhenMissing() throws Exception {
        given(service.findById(99L)).willReturn(Optional.empty());

        mvc.perform(get("/api/v1/orders/99"))
            .andExpect(status().isNotFound());
    }

    @Test
    void create_returns400OnInvalidBody() throws Exception {
        String badBody = """
            { "customerId": null, "items": [] }
            """;

        mvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(badBody))
            .andExpect(status().isBadRequest());
    }
}
```

### @DataJpaTest — Repository Slice

Spins up an in-memory H2 database. Runs Flyway migrations automatically.

```java
@DataJpaTest
class OrderRepositoryTest {

    @Autowired OrderRepository repo;
    @Autowired TestEntityManager em;

    @Test
    void findByStatus_returnsMatchingOrders() {
        em.persist(Order.builder().status(OrderStatus.PENDING).build());
        em.persist(Order.builder().status(OrderStatus.SHIPPED).build());
        em.flush();

        List<Order> pending = repo.findByStatus(OrderStatus.PENDING, Pageable.unpaged())
                                  .getContent();

        assertThat(pending).hasSize(1);
        assertThat(pending.get(0).getStatus()).isEqualTo(OrderStatus.PENDING);
    }
}
```

### @SpringBootTest — Full Integration Test

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class OrderIntegrationTest {

    @Autowired TestRestTemplate restTemplate;
    @Autowired OrderRepository  orderRepo;

    @Test
    void createOrder_persistsAndReturns201() {
        var request = new CreateOrderRequest(1L, "123 Main St",
                          List.of(new OrderItemRequest("ABC-001234", 2)));

        ResponseEntity<OrderDto> response =
            restTemplate.postForEntity("/api/v1/orders", request, OrderDto.class);

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody()).isNotNull();
        assertThat(orderRepo.existsById(response.getBody().id())).isTrue();
    }
}
```

### Testcontainers — Real Database in Tests

Spins up a real Postgres Docker container per test run. Eliminates H2 / prod divergence.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class OrderRepositoryTest {

    @Container
    @ServiceConnection                    // wires the container URL into Spring automatically
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16-alpine");

    @Autowired OrderRepository repo;

    @Test
    void findRecentOrders_returnsLastWeek() {
        // runs against real Postgres — Flyway migrations execute, types match, constraints apply
        List<Order> recent = repo.findRecentOrders();
        assertThat(recent).isNotNull();
    }
}
```

### @MockBean vs @Mock

| | `@Mock` (Mockito) | `@MockBean` (Spring) |
|---|---|---|
| Context | No Spring context needed | Requires Spring context |
| Use in | Unit tests (`@ExtendWith(MockitoExtension.class)`) | Slice tests (`@WebMvcTest`, `@SpringBootTest`) |
| Effect | Creates a Mockito mock | Creates mock AND replaces the bean in the ApplicationContext |

```java
// Unit test — no Spring, fast
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock        OrderRepository repo;
    @Mock        CustomerRepository customerRepo;
    @InjectMocks OrderService service;

    @Test
    void create_throwsWhenCustomerMissing() {
        given(customerRepo.findById(99L)).willReturn(Optional.empty());

        assertThatThrownBy(() -> service.create(new CreateOrderRequest(99L, "addr", List.of())))
            .isInstanceOf(EntityNotFoundException.class);
    }
}
```

---

## Full Example — E-Commerce Platform

Everything in these notes wired into one running system. Three backend services, three databases, a cache, a frontend, all containerised and deployed to Kubernetes.

### Architecture

```
Browser
  │
  ├── /          → Frontend  (Nginx + React,  port 80)
  │                   │  proxies /api/* to Gateway (avoids CORS)
  └── /api/**    → API Gateway              (port 8080)
                        │
          ┌─────────────┼──────────────┐
          ▼             ▼              ▼
    User Service   Order Service  Payment Service
    (port 8082)    (port 8081)    (port 8083)
    PostgreSQL      PostgreSQL     PostgreSQL
                    Redis cache
                    │         │
               Feign→User  Feign→Payment
                    │
               Circuit Breaker (Resilience4j)
               Spring Events → Email (async)

All services:
  ← pull config from Config Server (port 8888)
  → register with Eureka            (port 8761)
  → push traces to Zipkin           (port 9411)
  → expose /actuator/prometheus ← scraped by Prometheus → Grafana
```

### Project Structure

```
shop/
├── eureka-server/
│   └── src/main/
│       ├── java/.../EurekaServerApplication.java
│       └── resources/application.yml
│
├── config-server/
│   └── src/main/
│       ├── java/.../ConfigServerApplication.java
│       └── resources/application.yml
│
├── user-service/
│   └── src/main/
│       ├── java/com/shop/user/
│       │   ├── entity/User.java
│       │   ├── repository/UserRepository.java
│       │   ├── service/UserService.java
│       │   └── controller/UserController.java
│       └── resources/
│           ├── application.yml
│           └── db/migration/V1__create_users.sql
│
├── order-service/
│   └── src/main/
│       ├── java/com/shop/order/
│       │   ├── entity/Order.java + OrderItem.java
│       │   ├── repository/OrderRepository.java
│       │   ├── service/OrderService.java
│       │   ├── controller/OrderController.java
│       │   ├── client/UserClient.java + PaymentClient.java
│       │   └── listener/OrderEventListener.java
│       └── resources/
│           ├── application.yml
│           └── db/migration/
│               ├── V1__create_orders.sql
│               └── V2__create_order_items.sql
│
├── payment-service/
│   └── src/main/
│       ├── java/com/shop/payment/
│       │   ├── entity/Payment.java
│       │   ├── service/PaymentService.java
│       │   └── controller/PaymentController.java
│       └── resources/
│           ├── application.yml
│           └── db/migration/V1__create_payments.sql
│
├── api-gateway/
│   └── src/main/resources/application.yml
│
├── frontend/
│   ├── src/           (React source)
│   ├── nginx.conf
│   └── Dockerfile
│
├── docker-compose.yml
│
└── k8s/
    ├── namespace.yaml
    ├── secrets.yaml
    ├── infra/
    │   ├── postgres-user.yaml
    │   ├── postgres-order.yaml
    │   ├── postgres-payment.yaml
    │   ├── redis.yaml
    │   └── zipkin.yaml
    ├── eureka-server.yaml
    ├── config-server.yaml
    ├── user-service.yaml
    ├── order-service.yaml
    ├── payment-service.yaml
    ├── api-gateway.yaml
    ├── frontend.yaml
    └── ingress.yaml
```

---

### Eureka Server

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```yaml
server:
  port: 8761
spring:
  application:
    name: eureka-server
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
  server:
    wait-time-in-ms-when-sync-empty: 0   # don't wait for peers on startup
```

---

### Config Server

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

```yaml
server:
  port: 8888
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: file://${user.home}/shop-config   # local git repo; swap for GitHub URL in prod
          default-label: main
```

---

### User Service

```sql
-- V1__create_users.sql
CREATE TABLE users (
    id         BIGSERIAL    PRIMARY KEY,
    email      VARCHAR(255) NOT NULL UNIQUE,
    name       VARCHAR(255) NOT NULL,
    role       VARCHAR(20)  NOT NULL DEFAULT 'CUSTOMER',
    created_at TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
```

```java
@Entity @Table(name = "users")
@Getter @Setter @Builder @NoArgsConstructor @AllArgsConstructor
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String email;
    private String name;
    @Enumerated(EnumType.STRING)
    private Role role;
    private Instant createdAt;
}
```

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
}
```

```java
@Service
@Transactional
public class UserService {

    private final UserRepository repo;

    @Transactional(readOnly = true)
    public Optional<UserDto> findById(Long id) {
        return repo.findById(id).map(UserDto::from);
    }

    public UserDto create(CreateUserRequest req) {
        if (repo.existsByEmail(req.email()))
            throw new DuplicateResourceException("Email already registered: " + req.email());

        return UserDto.from(repo.save(User.builder()
            .email(req.email())
            .name(req.name())
            .role(Role.CUSTOMER)
            .createdAt(Instant.now())
            .build()));
    }
}
```

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService service;

    @GetMapping("/{id}")
    public ResponseEntity<UserDto> getById(@PathVariable Long id) {
        return service.findById(id).map(ResponseEntity::ok)
                      .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<UserDto> create(@RequestBody @Valid CreateUserRequest req) {
        return ResponseEntity.status(HttpStatus.CREATED).body(service.create(req));
    }
}
```

```yaml
# user-service/application.yml
spring:
  application:
    name: user-service
  config:
    import: configserver:http://config-server:8888
  datasource:
    url: jdbc:postgresql://postgres-user:5432/userdb
    username: ${DB_USER}
    password: ${DB_PASS}
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        default_batch_fetch_size: 50
  flyway:
    enabled: true
server:
  port: 8082
eureka:
  client:
    service-url:
      defaultZone: http://eureka-server:8761/eureka/
management:
  tracing:
    sampling:
      probability: 1.0
  zipkin:
    tracing:
      endpoint: http://zipkin:9411/api/v2/spans
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
```

---

### Order Service

The most complex service — calls User and Payment via Feign, caches reads in Redis, fires async events, and tracks metrics.

```sql
-- V1__create_orders.sql
CREATE TABLE orders (
    id         BIGSERIAL    PRIMARY KEY,
    user_id    BIGINT       NOT NULL,
    status     VARCHAR(20)  NOT NULL DEFAULT 'PENDING',
    total      NUMERIC(10,2) NOT NULL,
    created_at TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

-- V2__create_order_items.sql
CREATE TABLE order_items (
    id         BIGSERIAL    PRIMARY KEY,
    order_id   BIGINT       NOT NULL REFERENCES orders(id),
    sku        VARCHAR(50)  NOT NULL,
    quantity   INT          NOT NULL,
    unit_price NUMERIC(10,2) NOT NULL
);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

```java
@Entity @Table(name = "orders")
@Getter @Setter @Builder @NoArgsConstructor @AllArgsConstructor
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private Long userId;
    @Enumerated(EnumType.STRING)
    private OrderStatus status;
    private BigDecimal total;
    private Instant createdAt;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}

@Entity @Table(name = "order_items")
@Getter @Setter @Builder @NoArgsConstructor @AllArgsConstructor
public class OrderItem {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;

    private String sku;
    private int quantity;
    private BigDecimal unitPrice;
}
```

```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {

    @EntityGraph(attributePaths = "items")          // avoids N+1 on list queries
    Page<Order> findByUserId(Long userId, Pageable pageable);

    @EntityGraph(attributePaths = "items")
    Optional<Order> findWithItemsById(Long id);
}
```

```java
// Feign clients — Spring generates the implementation; lb:// resolves via Eureka

@FeignClient(name = "user-service", fallback = UserClientFallback.class)
public interface UserClient {
    @GetMapping("/api/users/{id}")
    UserDto getUser(@PathVariable Long id);
}

@Component
class UserClientFallback implements UserClient {
    @Override
    public UserDto getUser(Long id) { return UserDto.unknown(id); }
}

@FeignClient(name = "payment-service", fallback = PaymentClientFallback.class)
public interface PaymentClient {
    @PostMapping("/api/payments")
    PaymentDto charge(@RequestBody ChargeRequest req);
}

@Component
class PaymentClientFallback implements PaymentClient {
    @Override
    public PaymentDto charge(ChargeRequest req) {
        throw new ServiceUnavailableException("Payment service unavailable");
    }
}
```

```java
@Service
@Transactional
@EnableCaching
public class OrderService {

    private final OrderRepository        orderRepo;
    private final UserClient             userClient;
    private final PaymentClient          paymentClient;
    private final ApplicationEventPublisher publisher;
    private final Counter                ordersCreated;
    private final Timer                  processingTime;

    public OrderService(OrderRepository repo, UserClient uc, PaymentClient pc,
                        ApplicationEventPublisher pub, MeterRegistry registry) {
        this.orderRepo      = repo;
        this.userClient     = uc;
        this.paymentClient  = pc;
        this.publisher      = pub;
        this.ordersCreated  = Counter.builder("orders.created").register(registry);
        this.processingTime = Timer.builder("orders.processing.time").register(registry);
    }

    @Transactional(readOnly = true)
    @Cacheable(value = "orders", key = "#id")
    public Optional<OrderDto> findById(Long id) {
        return orderRepo.findWithItemsById(id).map(OrderDto::from);
    }

    @CircuitBreaker(name = "user-service", fallbackMethod = "rejectOrder")
    public OrderDto create(CreateOrderRequest req) {
        return processingTime.record(() -> {
            UserDto user = userClient.getUser(req.userId());   // fails fast if user-service is down

            List<OrderItem> items = req.items().stream()
                .map(i -> OrderItem.builder()
                    .sku(i.sku()).quantity(i.quantity()).unitPrice(i.unitPrice()).build())
                .toList();

            BigDecimal total = items.stream()
                .map(i -> i.getUnitPrice().multiply(BigDecimal.valueOf(i.getQuantity())))
                .reduce(BigDecimal.ZERO, BigDecimal::add);

            Order order = Order.builder()
                .userId(user.id()).status(OrderStatus.PENDING)
                .total(total).createdAt(Instant.now()).build();
            items.forEach(i -> i.setOrder(order));
            order.setItems(items);

            Order saved = orderRepo.save(order);

            // payment runs synchronously — if it fails, the whole transaction rolls back
            paymentClient.charge(new ChargeRequest(saved.getId(), total, req.paymentToken()));
            saved.setStatus(OrderStatus.CONFIRMED);

            // event fires after commit — listener sends email in a background thread
            publisher.publishEvent(new OrderPlacedEvent(saved.getId(), user.email()));
            ordersCreated.increment();

            return OrderDto.from(saved);
        });
    }

    private OrderDto rejectOrder(CreateOrderRequest req, Exception ex) {
        throw new ServiceUnavailableException("Cannot place order — user service is down");
    }

    @CacheEvict(value = "orders", key = "#id")
    public void cancel(Long id) {
        Order order = orderRepo.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("Order " + id));
        order.setStatus(OrderStatus.CANCELLED);
    }

    @Transactional(readOnly = true)
    public Page<OrderDto> findByUser(Long userId, Pageable pageable) {
        return orderRepo.findByUserId(userId, pageable).map(OrderDto::from);
    }
}
```

```java
// Fires after OrderService.create() commits — safe to send email now
@Component
public class OrderEventListener {

    private final EmailService emailService;

    @EventListener
    @Async
    public void onOrderPlaced(OrderPlacedEvent event) {
        emailService.sendConfirmation(event.customerEmail(), event.orderId());
    }
}
```

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final OrderService service;

    @GetMapping("/{id}")
    public ResponseEntity<OrderDto> getById(@PathVariable Long id) {
        return service.findById(id).map(ResponseEntity::ok)
                      .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<OrderDto> create(@RequestBody @Valid CreateOrderRequest req) {
        return ResponseEntity.status(HttpStatus.CREATED).body(service.create(req));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> cancel(@PathVariable Long id) {
        service.cancel(id);
        return ResponseEntity.noContent().build();
    }

    @GetMapping("/user/{userId}")
    public ResponseEntity<Page<OrderDto>> listByUser(
            @PathVariable Long userId, Pageable pageable) {
        return ResponseEntity.ok(service.findByUser(userId, pageable));
    }
}
```

```yaml
# order-service/application.yml
spring:
  application:
    name: order-service
  config:
    import: configserver:http://config-server:8888
  datasource:
    url: jdbc:postgresql://postgres-order:5432/orderdb
    username: ${DB_USER}
    password: ${DB_PASS}
  cache:
    type: redis
  data:
    redis:
      host: redis
      port: 6379
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        default_batch_fetch_size: 50
  flyway:
    enabled: true
server:
  port: 8081
eureka:
  client:
    service-url:
      defaultZone: http://eureka-server:8761/eureka/
resilience4j:
  circuitbreaker:
    instances:
      user-service:
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10s
        sliding-window-size: 10
      payment-service:
        failure-rate-threshold: 50
        wait-duration-in-open-state: 15s
        sliding-window-size: 10
management:
  tracing:
    sampling:
      probability: 1.0
  zipkin:
    tracing:
      endpoint: http://zipkin:9411/api/v2/spans
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
```

---

### Payment Service

```sql
-- V1__create_payments.sql
CREATE TABLE payments (
    id           BIGSERIAL     PRIMARY KEY,
    order_id     BIGINT        NOT NULL UNIQUE,
    amount       NUMERIC(10,2) NOT NULL,
    status       VARCHAR(20)   NOT NULL,
    processed_at TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);
```

```java
@Service
@Transactional
public class PaymentService {

    private final PaymentRepository repo;

    public PaymentDto charge(ChargeRequest req) {
        // production: call Stripe/PayPal SDK here; this is the stub
        Payment payment = Payment.builder()
            .orderId(req.orderId())
            .amount(req.amount())
            .status(PaymentStatus.COMPLETED)
            .processedAt(Instant.now())
            .build();
        return PaymentDto.from(repo.save(payment));
    }
}
```

```java
@RestController
@RequestMapping("/api/payments")
public class PaymentController {

    private final PaymentService service;

    @PostMapping
    public ResponseEntity<PaymentDto> charge(@RequestBody @Valid ChargeRequest req) {
        return ResponseEntity.status(HttpStatus.CREATED).body(service.charge(req));
    }
}
```

---

### API Gateway

```yaml
spring:
  application:
    name: api-gateway
  config:
    import: configserver:http://config-server:8888
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service          # lb:// = load-balanced through Eureka
          predicates:
            - Path=/api/users/**

        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**

        - id: payment-service
          uri: lb://payment-service
          predicates:
            - Path=/api/payments/**

      default-filters:
        - AddResponseHeader=X-Gateway, shop-gateway

server:
  port: 8080
eureka:
  client:
    service-url:
      defaultZone: http://eureka-server:8761/eureka/
```

---

### Frontend (React + Nginx)

```nginx
# frontend/nginx.conf
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    # SPA routing — unknown paths fall back to index.html
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Proxy API calls to the gateway — browser never sees a different origin, so no CORS
    location /api/ {
        proxy_pass         http://api-gateway:8080;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}
```

```js
// All API calls use /api prefix — Nginx forwards to the gateway, gateway routes to services
// No hardcoded service hostnames anywhere in the frontend

const api = {
    getOrder:   (id)   => fetch(`/api/orders/${id}`, { headers: authHeader() }).then(r => r.json()),
    createOrder: (body) => fetch('/api/orders', {
        method:  'POST',
        headers: { 'Content-Type': 'application/json', ...authHeader() },
        body:    JSON.stringify(body)
    }).then(r => r.json()),
    getUser: (id) => fetch(`/api/users/${id}`, { headers: authHeader() }).then(r => r.json()),
};

const authHeader = () => ({ Authorization: `Bearer ${localStorage.getItem('token')}` });
```

---

### Dockerfiles

All services use the same multi-stage pattern — Maven builds in a JDK image, the final image is lean JRE-only.

```dockerfile
# Dockerfile (same pattern for every Spring Boot service — change port + jar name)
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline -q
COPY src ./src
RUN ./mvnw package -DskipTests -q

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-jar", "app.jar"]
```

```dockerfile
# frontend/Dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --silent
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

---

### Docker Compose (local dev)

```yaml
# docker-compose.yml
services:

  # ── Infrastructure ──────────────────────────────────────────────────────────

  postgres-user:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: userdb
      POSTGRES_USER: shop
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata-user:/var/lib/postgresql/data

  postgres-order:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: orderdb
      POSTGRES_USER: shop
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata-order:/var/lib/postgresql/data

  postgres-payment:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: paymentdb
      POSTGRES_USER: shop
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata-payment:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

  zipkin:
    image: openzipkin/zipkin:latest
    ports:
      - "9411:9411"

  # ── Platform services ────────────────────────────────────────────────────────

  eureka-server:
    build: ./eureka-server
    ports:
      - "8761:8761"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8761/actuator/health"]
      interval: 10s
      retries: 5

  config-server:
    build: ./config-server
    ports:
      - "8888:8888"
    depends_on:
      eureka-server:
        condition: service_healthy

  # ── Domain services ──────────────────────────────────────────────────────────

  user-service:
    build: ./user-service
    ports:
      - "8082:8082"
    environment:
      DB_USER: shop
      DB_PASS: secret
    depends_on:
      eureka-server:
        condition: service_healthy
      postgres-user:
        condition: service_started
      config-server:
        condition: service_started

  payment-service:
    build: ./payment-service
    ports:
      - "8083:8083"
    environment:
      DB_USER: shop
      DB_PASS: secret
    depends_on:
      eureka-server:
        condition: service_healthy
      postgres-payment:
        condition: service_started
      config-server:
        condition: service_started

  order-service:
    build: ./order-service
    ports:
      - "8081:8081"
    environment:
      DB_USER: shop
      DB_PASS: secret
    depends_on:
      eureka-server:
        condition: service_healthy
      postgres-order:
        condition: service_started
      config-server:
        condition: service_started
      redis:
        condition: service_started
      user-service:
        condition: service_started
      payment-service:
        condition: service_started

  api-gateway:
    build: ./api-gateway
    ports:
      - "8080:8080"
    depends_on:
      eureka-server:
        condition: service_healthy
      config-server:
        condition: service_started

  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    depends_on:
      - api-gateway

volumes:
  pgdata-user:
  pgdata-order:
  pgdata-payment:
```

Start everything:
```bash
docker compose up --build
# Frontend:  http://localhost:3000
# Gateway:   http://localhost:8080
# Eureka UI: http://localhost:8761
# Zipkin:    http://localhost:9411
```

---

### Kubernetes

#### namespace + secrets

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: shop
```

```yaml
# k8s/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
  namespace: shop
type: Opaque
stringData:
  username: shop
  password: super-secret-prod-password   # use Sealed Secrets or Vault in real prod

---
apiVersion: v1
kind: Secret
metadata:
  name: jwt-secret
  namespace: shop
type: Opaque
stringData:
  value: your-256-bit-base64-encoded-secret
```

#### Infrastructure — one PostgreSQL StatefulSet per service

Each service gets its own database — schema isolation, independent scaling, independent backups.

```yaml
# k8s/infra/postgres-order.yaml  (repeat pattern for postgres-user, postgres-payment)
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-order
  namespace: shop
spec:
  serviceName: postgres-order
  replicas: 1
  selector:
    matchLabels:
      app: postgres-order
  template:
    metadata:
      labels:
        app: postgres-order
    spec:
      containers:
        - name: postgres
          image: postgres:16-alpine
          env:
            - name: POSTGRES_DB
              value: orderdb
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef: { name: db-credentials, key: username }
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef: { name: db-credentials, key: password }
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: pgdata
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: pgdata
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 20Gi

---
apiVersion: v1
kind: Service
metadata:
  name: postgres-order
  namespace: shop
spec:
  selector:
    app: postgres-order
  ports:
    - port: 5432
  clusterIP: None     # headless — pods addressed by DNS: postgres-order.shop.svc.cluster.local
```

```yaml
# k8s/infra/redis.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: shop
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:7-alpine
          ports:
            - containerPort: 6379

---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: shop
spec:
  selector:
    app: redis
  ports:
    - port: 6379
```

#### Domain service — Deployment + Service

One manifest shown; the pattern is identical for every Spring Boot service (change name, image, port).

```yaml
# k8s/order-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  namespace: shop
spec:
  replicas: 2                  # two pods — Gateway load-balances across them via Eureka
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
        - name: order-service
          image: shop/order-service:latest
          ports:
            - containerPort: 8081
          env:
            - name: DB_USER
              valueFrom:
                secretKeyRef: { name: db-credentials, key: username }
            - name: DB_PASS
              valueFrom:
                secretKeyRef: { name: db-credentials, key: password }
            - name: SPRING_PROFILES_ACTIVE
              value: prod
          # K8s uses Actuator health endpoints to manage traffic
          readinessProbe:      # pod receives traffic only when this returns 200
            httpGet:
              path: /actuator/health/readiness
              port: 8081
            initialDelaySeconds: 30
            periodSeconds: 10
          livenessProbe:       # pod is restarted if this fails
            httpGet:
              path: /actuator/health/liveness
              port: 8081
            initialDelaySeconds: 60
            periodSeconds: 20
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"
              cpu: "500m"

---
apiVersion: v1
kind: Service
metadata:
  name: order-service
  namespace: shop
spec:
  selector:
    app: order-service
  ports:
    - port: 8081
      targetPort: 8081
```

#### Ingress — single entry point

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: shop-ingress
  namespace: shop
  annotations:
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
spec:
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-gateway
                port:
                  number: 8080
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

#### Deploy

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/secrets.yaml
kubectl apply -f k8s/infra/              # postgres x3, redis, zipkin
kubectl apply -f k8s/eureka-server.yaml
kubectl apply -f k8s/config-server.yaml
kubectl apply -f k8s/user-service.yaml
kubectl apply -f k8s/payment-service.yaml
kubectl apply -f k8s/order-service.yaml
kubectl apply -f k8s/api-gateway.yaml
kubectl apply -f k8s/frontend.yaml
kubectl apply -f k8s/ingress.yaml

# Watch pods come up
kubectl get pods -n shop -w
```

### How a Request Flows End-to-End

```
1. User opens browser → hits http://shop.example.com/
   Ingress → frontend pod (Nginx serves React SPA)

2. User submits order form → POST /api/orders
   Nginx (nginx.conf proxy_pass) → api-gateway pod

3. Gateway matches route /api/orders/** → lb://order-service
   Eureka returns available order-service pod IPs → Gateway picks one

4. order-service.OrderService.create():
   a. Feign → GET user-service/api/users/{id}   (Circuit Breaker wraps this)
   b. Build Order entity, persist to postgres-order
   c. Feign → POST payment-service/api/payments  (charges card)
   d. Update status → CONFIRMED, save
   e. Publish OrderPlacedEvent

5. OrderEventListener (async thread):
   → EmailService sends confirmation to customer

6. Response: 201 Created + OrderDto flows back through Gateway → Nginx → browser

7. Zipkin collected traceId across steps 3-5 → full trace visible at :9411
   Prometheus scraped /actuator/prometheus → latency/error dashboards in Grafana
```

---

## JPA Auditing

Automatically populate `createdAt` / `updatedAt` / `createdBy` on every save without touching service code.

```java
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaConfig {

    @Bean
    public AuditorAware<String> auditorProvider() {
        // pulls the authenticated username from SecurityContext
        return () -> Optional.ofNullable(SecurityContextHolder.getContext().getAuthentication())
                             .map(Authentication::getName);
    }
}
```

```java
@MappedSuperclass   // not an entity itself — just a base class
@EntityListeners(AuditingEntityListener.class)
@Getter
public abstract class Auditable {

    @CreatedDate
    @Column(updatable = false)
    private Instant createdAt;

    @LastModifiedDate
    private Instant updatedAt;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String updatedBy;
}
```

```java
@Entity
public class Order extends Auditable {   // inherits all four audit columns
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    // ... other fields
}
```

---

## Optimistic Locking

Prevents lost updates when two transactions read and then write the same row concurrently. No database locks held — the check happens at commit time.

```java
@Entity
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Version          // Hibernate increments this on every UPDATE; throws if it changed since you read
    private Long version;

    private OrderStatus status;
}
```

```
Thread A reads Order(id=1, version=0, status=PENDING)
Thread B reads Order(id=1, version=0, status=PENDING)

Thread A saves → UPDATE orders SET status='CONFIRMED', version=1 WHERE id=1 AND version=0  ✓
Thread B saves → UPDATE orders SET status='CANCELLED', version=1 WHERE id=1 AND version=0  ✗ (0 rows affected)
                 → Spring throws OptimisticLockingFailureException
```

Handle the exception at the controller level:

```java
@ExceptionHandler(OptimisticLockingFailureException.class)
public ProblemDetail handleConflict(OptimisticLockingFailureException ex) {
    return ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT,
        "Order was modified by another request — please retry");
}
```

> Use optimistic locking for high-read, low-conflict scenarios. Use pessimistic locking (`@Lock(LockModeType.PESSIMISTIC_WRITE)`) only when conflicts are frequent and retries are unacceptable.

---

## Spring Data Projections

Fetch only the columns you need instead of full entities — reduces data transfer and avoids exposing internal fields.

### Interface Projection

Spring generates a proxy that maps query results to the interface at runtime.

```java
// Only name and email — no password, no audit fields
public interface UserSummary {
    Long getId();
    String getName();
    String getEmail();
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<UserSummary> findByRole(Role role);   // Spring returns only those two columns
}
```

### DTO Projection (Class-based)

More explicit — use when you need computed fields or constructor logic.

```java
public record OrderSummary(Long id, String status, BigDecimal total) {
    // JPQL constructor expression maps directly to this record
}

@Query("SELECT new com.shop.order.dto.OrderSummary(o.id, o.status, o.total) " +
       "FROM Order o WHERE o.userId = :userId")
List<OrderSummary> findSummariesByUser(@Param("userId") Long userId);
```

---

## Spring Data Specifications

Dynamic queries — build `WHERE` clauses at runtime based on which filters are present. Avoids a combinatorial explosion of repository methods.

```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long>,
                                          JpaSpecificationExecutor<Order> { }
```

```java
public class OrderSpecs {

    public static Specification<Order> hasStatus(OrderStatus status) {
        return (root, query, cb) ->
            status == null ? cb.conjunction()              // no filter if null
                           : cb.equal(root.get("status"), status);
    }

    public static Specification<Order> forUser(Long userId) {
        return (root, query, cb) ->
            userId == null ? cb.conjunction()
                           : cb.equal(root.get("userId"), userId);
    }

    public static Specification<Order> createdAfter(Instant from) {
        return (root, query, cb) ->
            from == null ? cb.conjunction()
                         : cb.greaterThanOrEqualTo(root.get("createdAt"), from);
    }
}
```

```java
@Service
public class OrderService {

    public Page<OrderDto> search(OrderStatus status, Long userId, Instant from, Pageable pageable) {
        Specification<Order> spec = Specification
            .where(OrderSpecs.hasStatus(status))
            .and(OrderSpecs.forUser(userId))
            .and(OrderSpecs.createdAfter(from));

        return orderRepo.findAll(spec, pageable).map(OrderDto::from);
    }
}
```

```java
// Controller — all params optional
@GetMapping("/search")
public ResponseEntity<Page<OrderDto>> search(
        @RequestParam(required = false) OrderStatus status,
        @RequestParam(required = false) Long userId,
        @RequestParam(required = false) @DateTimeFormat(iso = ISO.DATE_TIME) Instant from,
        Pageable pageable) {
    return ResponseEntity.ok(service.search(status, userId, from, pageable));
}
// GET /api/orders/search?status=PENDING&page=0&size=20
// GET /api/orders/search?userId=5&from=2026-01-01T00:00:00Z
```

---

## Caching

Spring's cache abstraction works with Redis, Caffeine, EhCache — swap the backend without changing annotations.

```yaml
spring:
  cache:
    type: redis
  data:
    redis:
      host: redis
      port: 6379
      timeout: 2s        # connection timeout — fail fast if Redis is down
  cache:
    redis:
      time-to-live: 600s   # default TTL for all caches
```

```java
@SpringBootApplication
@EnableCaching
public class Application { ... }
```

| Annotation | When to use |
|---|---|
| `@Cacheable` | Cache the return value; skip method if cache hit |
| `@CachePut` | Always execute method AND update cache — good for writes |
| `@CacheEvict` | Remove one or all entries — call on delete/update |

```java
@Service
public class UserService {

    @Cacheable(value = "users", key = "#id")
    public Optional<UserDto> findById(Long id) {
        return repo.findById(id).map(UserDto::from);
        // second call with same id hits Redis — DB never queried
    }

    @CachePut(value = "users", key = "#result.id")
    public UserDto update(Long id, UpdateUserRequest req) {
        User user = repo.findById(id).orElseThrow();
        user.setName(req.name());
        return UserDto.from(repo.save(user));
        // executes AND writes the new value to cache
    }

    @CacheEvict(value = "users", key = "#id")
    public void delete(Long id) {
        repo.deleteById(id);
        // removes this entry from cache after deletion
    }

    @CacheEvict(value = "users", allEntries = true)
    @Scheduled(cron = "0 0 3 * * *")     // nightly cache flush at 03:00
    public void evictAllUsers() { }
}
```

**Conditional caching** — skip cache for unauthenticated or admin requests:

```java
@Cacheable(value = "orders", key = "#id", condition = "#id > 0",
           unless = "#result == null")   // don't cache null (not-found) results
public Optional<OrderDto> findById(Long id) { ... }
```

---

## OAuth2 Resource Server

The production JWT approach. Instead of a hand-rolled `JwtAuthFilter`, configure Spring to validate tokens natively — it handles signature verification, expiry, and claims extraction automatically.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          # Option A — your own secret (symmetric, HS256)
          # Spring auto-validates signature and expiry
          secret: ${JWT_SECRET}

          # Option B — external identity provider (Keycloak, Auth0, Okta)
          # Spring fetches the public keys from the jwks-uri automatically
          issuer-uri: https://auth.example.com/realms/shop
          # jwks-uri: https://auth.example.com/realms/shop/protocol/openid-connect/certs
```

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)
            .sessionManagement(sm -> sm.sessionCreationPolicy(STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**", "/actuator/health").permitAll()
                .anyRequest().authenticated())
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtConverter())))
            .build();
    }

    // Maps JWT claims → Spring GrantedAuthority list
    @Bean
    public JwtAuthenticationConverter jwtConverter() {
        JwtGrantedAuthoritiesConverter converter = new JwtGrantedAuthoritiesConverter();
        converter.setAuthoritiesClaimName("roles");     // claim name in your token
        converter.setAuthorityPrefix("ROLE_");

        JwtAuthenticationConverter jwtConverter = new JwtAuthenticationConverter();
        jwtConverter.setJwtGrantedAuthoritiesConverter(converter);
        return jwtConverter;
    }
}
```

**Auth server issues a token:**
```json
{
  "sub": "user-42",
  "email": "alice@example.com",
  "roles": ["CUSTOMER", "ADMIN"],
  "iat": 1719600000,
  "exp": 1719686400
}
```

**Reading JWT claims inside a controller/service:**

```java
@GetMapping("/me")
public ResponseEntity<UserDto> me(@AuthenticationPrincipal Jwt jwt) {
    String userId = jwt.getSubject();
    String email  = jwt.getClaimAsString("email");
    List<String> roles = jwt.getClaimAsStringList("roles");
    return ResponseEntity.ok(service.findById(Long.parseLong(userId)).orElseThrow());
}
```

**Auth controller — issue tokens:**

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final AuthenticationManager authManager;
    private final JwtEncoder jwtEncoder;

    @PostMapping("/login")
    public ResponseEntity<TokenResponse> login(@RequestBody LoginRequest req) {
        Authentication auth = authManager.authenticate(
            new UsernamePasswordAuthenticationToken(req.email(), req.password()));

        User user = (User) auth.getPrincipal();

        JwtClaimsSet claims = JwtClaimsSet.builder()
            .subject(user.getId().toString())
            .claim("email", user.getEmail())
            .claim("roles", user.getRoles())
            .issuedAt(Instant.now())
            .expiresAt(Instant.now().plusSeconds(86400))
            .build();

        String token = jwtEncoder.encode(JwtEncoderParameters.from(claims)).getTokenValue();
        return ResponseEntity.ok(new TokenResponse(token));
    }
}
```

> The hand-rolled `JwtAuthFilter` from earlier in these notes is educational but avoid it in production. Spring's resource server handles edge cases (clock skew, malformed tokens, JWKS rotation) that a manual implementation will miss.

---

## MapStruct — DTO Mapping

Compile-time code generator — zero reflection, no runtime overhead. Converts entities ↔ DTOs automatically.

```java
// Source
@Entity
public class Order {
    private Long id;
    private Long userId;
    private OrderStatus status;
    private BigDecimal total;
    private Instant createdAt;
    private List<OrderItem> items;
}

// Target
public record OrderDto(
    Long id,
    Long userId,
    String status,       // enum → String
    BigDecimal total,
    Instant createdAt,
    List<OrderItemDto> items
) {}
```

```java
@Mapper(componentModel = "spring",   // makes it a Spring @Component — inject normally
        uses = OrderItemMapper.class) // delegate item mapping to another mapper
public interface OrderMapper {

    // field names match → mapped automatically
    // enum → String → handled automatically
    OrderDto toDto(Order order);

    List<OrderDto> toDtoList(List<Order> orders);

    // Explicit mapping when names differ
    @Mapping(source = "req.customerId", target = "userId")
    @Mapping(target = "id",        ignore = true)   // DB generates this
    @Mapping(target = "createdAt", ignore = true)   // set by JPA auditing
    @Mapping(target = "status",    constant = "PENDING")
    Order toEntity(CreateOrderRequest req);
}

@Mapper(componentModel = "spring")
public interface OrderItemMapper {
    OrderItemDto toDto(OrderItem item);
}
```

```java
// Usage in service — no manual field copying
@Service
public class OrderService {
    private final OrderRepository repo;
    private final OrderMapper mapper;

    public OrderDto create(CreateOrderRequest req) {
        Order order = mapper.toEntity(req);   // MapStruct generated code
        return mapper.toDto(repo.save(order));
    }
}
```

MapStruct generates the implementation class at compile time — check `target/generated-sources` to see it. If a mapping is ambiguous or missing, it fails at **compile time**, not runtime.

---

## Kafka

Asynchronous, durable messaging. Use when services should not wait for each other (fire-and-forget) or when you need event replay.

```yaml
spring:
  kafka:
    bootstrap-servers: kafka:9092
    producer:
      key-serializer:   org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      group-id: order-service
      key-deserializer:   org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "com.shop.*"
      auto-offset-reset: earliest   # start from beginning if no committed offset
```

### Producer

```java
@Service
public class OrderEventPublisher {

    private final KafkaTemplate<String, OrderPlacedEvent> kafka;

    public void publishOrderPlaced(Order order) {
        OrderPlacedEvent event = new OrderPlacedEvent(order.getId(), order.getUserId(),
                                                       order.getTotal(), Instant.now());
        kafka.send("order-placed", order.getId().toString(), event);
        // key = orderId → ensures all events for the same order go to the same partition (ordered)
    }
}
```

### Consumer

```java
@Component
public class OrderEventConsumer {

    private final NotificationService notificationService;
    private final InventoryService inventoryService;

    // Each @KafkaListener method is its own consumer — both run in the same consumer group
    @KafkaListener(topics = "order-placed", groupId = "notification-service")
    public void sendConfirmation(OrderPlacedEvent event) {
        notificationService.sendOrderConfirmation(event.userId(), event.orderId());
    }

    @KafkaListener(topics = "order-placed", groupId = "inventory-service")
    public void reserveStock(OrderPlacedEvent event) {
        inventoryService.reserve(event.orderId());
    }
}
```

### Error Handling — Dead Letter Topic

Failed messages go to a `-dlt` topic instead of blocking the partition.

```java
@Configuration
public class KafkaConfig {

    @Bean
    public DefaultErrorHandler errorHandler(KafkaTemplate<?, ?> template) {
        DeadLetterPublishingRecoverer recoverer =
            new DeadLetterPublishingRecoverer(template,
                (record, ex) -> new TopicPartition(record.topic() + "-dlt", record.partition()));

        // retry 3 times with 1s backoff before sending to DLT
        return new DefaultErrorHandler(recoverer,
            new FixedBackOff(1000L, 3));
    }
}
```

**Kafka vs Spring Events:**

| | Spring Events | Kafka |
|---|---|---|
| Scope | In-process only | Cross-service, persistent |
| Durability | Lost on crash | Replayed from log |
| Ordering | Guaranteed | Per-partition |
| Use when | Internal decoupling within one service | Cross-service async communication |

---

## OAuth2 — CORS Configuration

Required when the frontend is on a different origin (domain/port) from the API.

```java
@Configuration
public class CorsConfig {

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("https://shop.example.com"));
        config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        config.setAllowedHeaders(List.of("Authorization", "Content-Type"));
        config.setAllowCredentials(true);
        config.setMaxAge(3600L);   // preflight cache duration in seconds

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", config);
        return source;
    }
}
```

```java
// Wire it into the security filter chain
.cors(cors -> cors.configurationSource(corsConfigurationSource()))
```

> If you proxy `/api` through Nginx (as in the full example above), CORS is not needed — the browser sees one origin for everything. Only configure CORS when the frontend and API are on genuinely different origins.

---

## OpenAPI / Swagger

Auto-generates interactive API docs from your controllers. Available at `/swagger-ui.html` and `/v3/api-docs`.

```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI openAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("Shop API")
                .version("1.0.0")
                .description("E-commerce platform API"))
            .addSecurityItem(new SecurityRequirement().addList("bearerAuth"))
            .components(new Components()
                .addSecuritySchemes("bearerAuth",
                    new SecurityScheme()
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")));
    }
}
```

Annotate controllers for richer docs:

```java
@RestController
@RequestMapping("/api/orders")
@Tag(name = "Orders", description = "Order management")
public class OrderController {

    @Operation(summary = "Get order by ID")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Order found"),
        @ApiResponse(responseCode = "404", description = "Order not found",
                     content = @Content(schema = @Schema(implementation = ProblemDetail.class)))
    })
    @GetMapping("/{id}")
    public ResponseEntity<OrderDto> getById(
            @Parameter(description = "Order ID") @PathVariable Long id) {
        return service.findById(id).map(ResponseEntity::ok)
                      .orElse(ResponseEntity.notFound().build());
    }
}
```

Disable in production if you don't want to expose the schema:

```yaml
springdoc:
  api-docs:
    enabled: ${SWAGGER_ENABLED:false}   # false by default; set true in dev/staging
  swagger-ui:
    enabled: ${SWAGGER_ENABLED:false}
```

---

## Production Tuning

### HikariCP — Connection Pool

`spring-boot-starter-data-jpa` includes HikariCP. Default pool size is 10 — almost always wrong for production.

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20          # max connections to the DB
      minimum-idle: 5                # keep 5 warm at all times
      connection-timeout: 3000       # ms to wait for a connection before throwing
      idle-timeout: 600000           # ms before an idle connection is closed (10 min)
      max-lifetime: 1800000          # ms before a connection is retired regardless (30 min)
      keepalive-time: 30000          # send keepalive ping every 30s (prevents firewall drops)
      pool-name: OrderServicePool
      leak-detection-threshold: 5000 # warn if a connection is held longer than 5s
```

Rule of thumb for `maximum-pool-size`: `(core_count * 2) + effective_spindle_count`. For a 4-core app server: ~10. Larger pools do not always mean more throughput — they cause context switching and DB pressure.

### Graceful Shutdown

Lets in-flight requests finish before the JVM exits. Essential in Kubernetes — pods are killed during rolling deploys.

```yaml
server:
  shutdown: graceful          # wait for requests to finish (default: immediate)

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s  # max wait time before force-kill
```

K8s sends `SIGTERM` → Spring stops accepting new requests → existing requests drain → JVM exits. Match `timeout-per-shutdown-phase` to your K8s `terminationGracePeriodSeconds`.

```yaml
# k8s/order-service.yaml
spec:
  template:
    spec:
      terminationGracePeriodSeconds: 45   # slightly longer than Spring's timeout
      containers:
        - name: order-service
          lifecycle:
            preStop:
              exec:
                command: ["sh", "-c", "sleep 5"]  # wait for load balancer to deregister
```

### JVM Flags for Containers

```dockerfile
ENTRYPOINT ["java",
  "-XX:+UseContainerSupport",          # read CPU/memory limits from cgroups (not host)
  "-XX:MaxRAMPercentage=75.0",         # use 75% of container memory for heap
  "-XX:+UseG1GC",                      # G1 GC — good default for most services
  "-XX:+ExitOnOutOfMemoryError",       # crash hard instead of limping along
  "-Djava.security.egd=file:/dev/./urandom",  # faster random number generation
  "-jar", "app.jar"]
```

> Without `-XX:+UseContainerSupport`, the JVM reads the host machine's memory, not the container limit — the heap may be set to 4 GB when the container only has 512 MB, causing OOM kills.

### Horizontal Pod Autoscaler (K8s)

Scale pods automatically based on CPU or custom metrics.

```yaml
# k8s/order-service-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
  namespace: shop
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70    # add a pod when average CPU exceeds 70%
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

---

## Structured Logging

In production you ship logs to Elasticsearch or Loki. JSON format makes them machine-parseable and correlates `traceId` automatically across services.

**Logback JSON encoder** (`pom.xml`):

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

```xml
<!-- src/main/resources/logback-spring.xml -->
<configuration>
    <springProfile name="prod">
        <appender name="JSON_CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="net.logstash.logback.encoder.LogstashEncoder">
                <!-- Micrometer Tracing auto-injects traceId + spanId into MDC -->
                <includeMdcKeyName>traceId</includeMdcKeyName>
                <includeMdcKeyName>spanId</includeMdcKeyName>
            </encoder>
        </appender>
        <root level="INFO">
            <appender-ref ref="JSON_CONSOLE"/>
        </root>
    </springProfile>

    <springProfile name="!prod">
        <!-- Human-readable in dev -->
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss} [%thread] %-5level %logger{36} traceId=%X{traceId} - %msg%n</pattern>
            </encoder>
        </appender>
        <root level="DEBUG">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
</configuration>
```

A single log line in prod looks like:

```json
{
  "timestamp": "2026-06-29T10:32:11.420Z",
  "level": "INFO",
  "logger": "com.shop.order.service.OrderService",
  "message": "Order 42 confirmed for user 7",
  "traceId": "4bf92f3577b34da6",
  "spanId": "00f067aa0ba902b7",
  "service": "order-service"
}
```

That `traceId` is the same one Zipkin recorded — paste it into Zipkin and see the full distributed trace.

**Avoid logging secrets:**

```java
// Bad — logs the full request which may contain payment tokens
log.info("Processing request: {}", request);

// Good — log only safe identifiers
log.info("Processing order for userId={}, amount={}", req.userId(), req.total());
```

---

## JPA — @Embeddable / @Embedded

Value objects: logically grouped fields that belong to an entity but live in the same table row. No separate table, no foreign key, no `@Entity`.

```java
@Embeddable
public class Address {
    private String street;
    private String city;
    private String country;
    @Column(name = "postal_code")
    private String postalCode;
}

@Entity
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;

    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "street",     column = @Column(name = "billing_street")),
        @AttributeOverride(name = "city",       column = @Column(name = "billing_city")),
        @AttributeOverride(name = "country",    column = @Column(name = "billing_country")),
        @AttributeOverride(name = "postalCode", column = @Column(name = "billing_postal_code"))
    })
    private Address billingAddress;

    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "street",  column = @Column(name = "shipping_street")),
        @AttributeOverride(name = "city",    column = @Column(name = "shipping_city")),
        @AttributeOverride(name = "country", column = @Column(name = "shipping_country")),
        @AttributeOverride(name = "postalCode", column = @Column(name = "shipping_postal_code"))
    })
    private Address shippingAddress;
    // `@AttributeOverrides` required when the same @Embeddable is used twice — avoids column name clash
}
```

The `users` table ends up with flat columns: `billing_street`, `billing_city`, `shipping_street`, etc. The Java model stays clean and grouped.

---

## JPA — Inheritance Strategies

When a class hierarchy needs to be persisted. Three strategies; pick based on query patterns and schema constraints.

### SINGLE_TABLE (default)

All subclasses in one table. A `dtype` discriminator column identifies the type. Fastest queries (no JOIN), but nullable columns for subclass-specific fields.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "payment_type", discriminatorType = DiscriminatorType.STRING)
public abstract class Payment {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private BigDecimal amount;
    private Instant processedAt;
}

@Entity
@DiscriminatorValue("CARD")
public class CardPayment extends Payment {
    private String last4Digits;
    private String cardBrand;   // nullable for non-card payments
}

@Entity
@DiscriminatorValue("BANK")
public class BankPayment extends Payment {
    private String iban;        // nullable for non-bank payments
}
```

### JOINED

Each class maps to its own table. Subclass tables share the same PK. Normalized schema, but queries use JOINs.

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Notification {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private Long userId;
    private Instant sentAt;
}

@Entity
@Table(name = "email_notifications")
public class EmailNotification extends Notification {
    private String toAddress;
    private String subject;
}

@Entity
@Table(name = "sms_notifications")
public class SmsNotification extends Notification {
    private String phoneNumber;
}
// SELECT * FROM notification n JOIN email_notification e ON n.id = e.id WHERE ...
```

### TABLE_PER_CLASS

Each concrete class gets its own fully independent table — no shared parent table, no JOINs, but polymorphic queries (`findAll()`) become a UNION across all tables. Rarely used.

**Choosing a strategy:**

| Strategy | Schema | Query perf | Use when |
|---|---|---|---|
| `SINGLE_TABLE` | Denormalized, nullable cols | Fastest — no JOIN | Subclasses share most fields |
| `JOINED` | Normalized | JOIN per query | Subclasses have distinct fields |
| `TABLE_PER_CLASS` | Fully separate tables | UNION for polymorphic | Never query the base type |

---

## Bean Lifecycle

Spring manages the full lifecycle: instantiate → inject → init → use → destroy.

```java
@Component
public class CacheWarmup {

    private final ProductRepository repo;
    private Map<Long, Product> cache;

    // 1. Constructor runs — dependencies injected
    public CacheWarmup(ProductRepository repo) {
        this.repo = repo;
    }

    // 2. @PostConstruct runs after all dependencies are injected
    @PostConstruct
    public void init() {
        cache = repo.findAll().stream()
                    .collect(Collectors.toMap(Product::getId, p -> p));
        log.info("Cache warmed with {} products", cache.size());
    }

    // 3. Bean is used normally...

    // 4. @PreDestroy runs when the application context closes (graceful shutdown)
    @PreDestroy
    public void cleanup() {
        cache.clear();
        log.info("Cache cleared on shutdown");
    }
}
```

**Common use cases:**

| Hook | Use for |
|---|---|
| `@PostConstruct` | Cache warming, resource initialisation, validation of config |
| `@PreDestroy` | Closing connections, flushing buffers, releasing resources |

> `@PostConstruct` is also a good place to validate `@ConfigurationProperties` beyond what `@Validated` can express — e.g. checking that two related values are consistent.

---

## Spring Boot Auto-configuration

How starters wire themselves up without any `@Bean` declarations from you. Understanding this lets you debug "why is this bean appearing?" and write your own starters.

```
spring-boot-starter-data-jpa
  └── spring-boot-autoconfigure
        └── META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
              └── JpaRepositoriesAutoConfiguration
                    └── @ConditionalOnClass(JpaRepository.class)   ← is JPA on the classpath?
                    └── @ConditionalOnMissingBean(JpaRepositoryFactoryBean.class)  ← did you define your own?
                    └── @EnableJpaRepositories  ← wires Spring Data if both conditions pass
```

**Key condition annotations:**

| Annotation | Registers bean only when |
|---|---|
| `@ConditionalOnClass` | A class is present on the classpath |
| `@ConditionalOnMissingBean` | No bean of that type was defined by the user |
| `@ConditionalOnProperty` | A config property is set (and optionally equals a value) |
| `@ConditionalOnMissingClass` | A class is absent |

```java
// How a starter auto-configures a client
@AutoConfiguration
@ConditionalOnClass(RedisClient.class)           // only if Lettuce is on classpath
@ConditionalOnProperty(prefix = "spring.data.redis", name = "host")  // only if host is set
public class RedisAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean                    // only if user hasn't defined their own
    public RedisConnectionFactory redisConnectionFactory(RedisProperties props) {
        return new LettuceConnectionFactory(props.getHost(), props.getPort());
    }
}
```

**Debugging auto-configuration** — see what was applied and why it was skipped:

```bash
java -jar app.jar --debug 2>&1 | grep -A2 "Did not match\|Matched"
# or hit /actuator/conditions in a running app
```

---

## Security Testing

Test secured controllers without running a full auth server.

```java
@WebMvcTest(OrderController.class)
@Import(SecurityConfig.class)   // import your security config into the slice
class OrderControllerSecurityTest {

    @Autowired MockMvc mvc;
    @MockBean  OrderService service;

    @Test
    void getOrder_returns401WhenNoToken() throws Exception {
        mvc.perform(get("/api/orders/1"))
            .andExpect(status().isUnauthorized());
    }

    @Test
    @WithMockUser(roles = "CUSTOMER")      // injects a mock user into SecurityContext
    void getOrder_returns200ForAuthenticatedUser() throws Exception {
        given(service.findById(1L)).willReturn(Optional.of(OrderDto.stub()));

        mvc.perform(get("/api/orders/1"))
            .andExpect(status().isOk());
    }

    @Test
    @WithMockUser(roles = "CUSTOMER")
    void deleteOrder_returns403ForNonAdmin() throws Exception {
        mvc.perform(delete("/api/orders/1"))
            .andExpect(status().isForbidden());
    }

    @Test
    @WithMockUser(roles = "ADMIN")
    void deleteOrder_returns204ForAdmin() throws Exception {
        mvc.perform(delete("/api/orders/1"))
            .andExpect(status().isNoContent());
    }
}
```

**Custom principal** — when you need JWT claims in the test:

```java
@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = WithMockJwtFactory.class)
public @interface WithMockJwt {
    long userId() default 1L;
    String[] roles() default {"CUSTOMER"};
}

public class WithMockJwtFactory implements WithSecurityContextFactory<WithMockJwt> {

    @Override
    public SecurityContext createSecurityContext(WithMockJwt annotation) {
        Jwt jwt = Jwt.withTokenValue("mock-token")
            .header("alg", "RS256")
            .subject(String.valueOf(annotation.userId()))
            .claim("roles", List.of(annotation.roles()))
            .issuedAt(Instant.now())
            .expiresAt(Instant.now().plusSeconds(3600))
            .build();

        JwtAuthenticationToken auth = new JwtAuthenticationToken(jwt,
            Arrays.stream(annotation.roles())
                  .map(r -> new SimpleGrantedAuthority("ROLE_" + r))
                  .toList());

        SecurityContext ctx = SecurityContextHolder.createEmptyContext();
        ctx.setAuthentication(auth);
        return ctx;
    }
}

// Usage
@Test
@WithMockJwt(userId = 42L, roles = {"ADMIN"})
void deleteOrder_adminCanDelete() throws Exception { ... }
```

---

## API Versioning

No single correct answer — pick a strategy and apply it consistently.

### URL Path Versioning (most common)

```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderControllerV1 { ... }

@RestController
@RequestMapping("/api/v2/orders")
public class OrderControllerV2 { ... }   // new response shape, breaking change
```

Pros: obvious, cacheable, easy to route at the gateway. Cons: pollutes URLs, forces clients to update paths on major versions.

### Header Versioning

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @GetMapping(headers = "API-Version=1")
    public ResponseEntity<OrderDtoV1> getV1(@PathVariable Long id) { ... }

    @GetMapping(headers = "API-Version=2")
    public ResponseEntity<OrderDtoV2> getV2(@PathVariable Long id) { ... }
}
```

Pros: clean URLs. Cons: not cacheable by default, harder to test in a browser.

### Content Negotiation

```java
@GetMapping(produces = "application/vnd.shop.order-v1+json")
public OrderDtoV1 getV1() { ... }

@GetMapping(produces = "application/vnd.shop.order-v2+json")
public OrderDtoV2 getV2() { ... }
// Client sends: Accept: application/vnd.shop.order-v2+json
```

**Deprecation strategy** — tell clients which version is going away:

```java
@GetMapping("/api/v1/orders/{id}")
@Deprecated
public ResponseEntity<OrderDtoV1> getV1(@PathVariable Long id) {
    HttpHeaders headers = new HttpHeaders();
    headers.add("Deprecation", "true");
    headers.add("Sunset", "Sat, 31 Dec 2026 23:59:59 GMT");
    headers.add("Link", "</api/v2/orders/" + id + ">; rel=\"successor-version\"");
    return ResponseEntity.ok().headers(headers).body(service.getV1(id));
}
```

---

## Idempotency

Guarantees that a duplicate request (retry, network timeout) has the same effect as the original — critical for payments, order creation, anything financial.

```java
// Client sends a unique key with every request
// POST /api/orders
// Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
```

```java
@Entity
public class IdempotencyRecord {
    @Id
    private String key;              // the UUID from the header
    private String responseBody;     // cached JSON response
    private int    statusCode;
    private Instant createdAt;
}
```

```java
@Service
public class IdempotencyService {

    private final IdempotencyRepository repo;

    public Optional<CachedResponse> findExisting(String key) {
        return repo.findById(key).map(r -> new CachedResponse(r.getStatusCode(), r.getResponseBody()));
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)   // save independently of outer tx
    public void save(String key, int status, String body) {
        repo.save(new IdempotencyRecord(key, body, status, Instant.now()));
    }
}
```

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final OrderService service;
    private final IdempotencyService idempotency;
    private final ObjectMapper mapper;

    @PostMapping
    public ResponseEntity<OrderDto> create(
            @RequestHeader("Idempotency-Key") String key,
            @RequestBody @Valid CreateOrderRequest req) throws JsonProcessingException {

        // Return cached response if this key was seen before
        Optional<CachedResponse> cached = idempotency.findExisting(key);
        if (cached.isPresent()) {
            return ResponseEntity.status(cached.get().status())
                                 .body(mapper.readValue(cached.get().body(), OrderDto.class));
        }

        OrderDto created = service.create(req);
        String body = mapper.writeValueAsString(created);
        idempotency.save(key, 201, body);

        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
}
```

> Clean up old idempotency records with a scheduled job. Keys are typically valid for 24 hours.

---

## Rate Limiting — API Gateway

Protects services from being overwhelmed. Configure at the gateway so individual services don't need to handle it.

```xml
<!-- pom.xml — gateway service -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

```yaml
# api-gateway/application.yml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 20    # tokens added per second
                redis-rate-limiter.burstCapacity: 40    # max tokens in the bucket
                redis-rate-limiter.requestedTokens: 1   # cost per request
                key-resolver: "#{@ipKeyResolver}"       # rate limit per IP
```

```java
@Configuration
public class RateLimiterConfig {

    // Rate limit per client IP
    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> Mono.just(
            Objects.requireNonNull(exchange.getRequest().getRemoteAddress())
                   .getAddress().getHostAddress());
    }

    // Rate limit per authenticated user (fairer — power users don't starve others)
    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> exchange.getPrincipal()
                                   .map(Principal::getName)
                                   .defaultIfEmpty("anonymous");
    }
}
```

When the limit is exceeded the gateway returns `429 Too Many Requests` with headers:

```
X-RateLimit-Remaining: 0
X-RateLimit-Replenish-Rate: 20
X-RateLimit-Burst-Capacity: 40
```

---

## Saga Pattern — Distributed Transactions

The question every senior gets: *"How do you maintain data consistency across microservices without distributed transactions?"*

2PC (two-phase commit) is theoretically correct but brittle in practice — one coordinator failure blocks all participants. Sagas replace it with a sequence of local transactions, each publishing an event. On failure, compensating transactions undo the work.

### Choreography-based Saga (event-driven)

No central coordinator. Each service listens to events and decides what to do next.

```
Order Service          Payment Service         Inventory Service
     │                      │                        │
     ├─ save Order(PENDING) ─┤                        │
     ├─ emit OrderCreated ──►│                        │
     │                      ├─ charge card            │
     │                      ├─ emit PaymentCompleted ►│
     │                      │                        ├─ reserve stock
     │                      │                        ├─ emit StockReserved
     │◄─────────────────────┼────────────────────────┘
     ├─ update Order(CONFIRMED)

── on failure ──────────────────────────────────────────────────────
Payment fails:
     │                      │
     │                      ├─ emit PaymentFailed ───►Order Service
     │◄────────────────────────────────────────────────
     ├─ update Order(CANCELLED)   ← compensating transaction
```

```java
// Order Service — reacts to payment outcome
@Component
public class SagaEventListener {

    private final OrderRepository orderRepo;

    @KafkaListener(topics = "payment-completed")
    public void onPaymentCompleted(PaymentCompletedEvent event) {
        orderRepo.findById(event.orderId()).ifPresent(order -> {
            order.setStatus(OrderStatus.CONFIRMED);
            orderRepo.save(order);
        });
    }

    @KafkaListener(topics = "payment-failed")
    public void onPaymentFailed(PaymentFailedEvent event) {
        orderRepo.findById(event.orderId()).ifPresent(order -> {
            order.setStatus(OrderStatus.CANCELLED);   // compensating transaction
            orderRepo.save(order);
        });
    }
}
```

### Orchestration-based Saga

A central saga orchestrator drives the sequence, making the flow explicit and easier to reason about.

```java
@Component
public class PlaceOrderSaga {

    private final PaymentClient paymentClient;
    private final InventoryClient inventoryClient;
    private final OrderRepository orderRepo;

    public void execute(Order order) {
        try {
            paymentClient.charge(order.getId(), order.getTotal());
            inventoryClient.reserve(order.getId(), order.getItems());
            order.setStatus(OrderStatus.CONFIRMED);
        } catch (PaymentException ex) {
            order.setStatus(OrderStatus.CANCELLED);   // nothing to compensate yet
        } catch (InventoryException ex) {
            paymentClient.refund(order.getId());      // compensate the payment
            order.setStatus(OrderStatus.CANCELLED);
        } finally {
            orderRepo.save(order);
        }
    }
}
```

**Choreography vs Orchestration:**

| | Choreography | Orchestration |
|---|---|---|
| Coupling | Services only know about events | Services know about the orchestrator |
| Visibility | Hard to see the full flow | Flow is explicit in one place |
| Failure handling | Distributed across services | Centralised |
| Use when | Simple, few steps | Complex flows with many compensations |
```
