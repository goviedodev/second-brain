---
created: 2026-09-22
tags: [project, entrevista, java]
source: /home/goviedo/proyectos/entrevistas/preparacion-java/guia-entrevista.md
---

# 8. Seguridad: JWT, OAuth2 y Spring Security

Parte de [[Preparación entrevista Java FullStack]]. Lo marcado con 🎯 es lo que más probablemente pregunten.

> El temario dice **"JWT / OAuth2 — esto sí o sí"**. Es el tema más probable de la entrevista. Prepáralo hasta poder dibujarlo.

## 8.1 Autenticación vs Autorización 🎯
- **Autenticación (AuthN):** *¿quién eres?* → login, credenciales, token.
- **Autorización (AuthZ):** *¿qué puedes hacer?* → roles, scopes, permisos.

## 8.2 JWT — estructura

```
header.payload.signature
```
```json
// header
{ "alg": "RS256", "typ": "JWT" }
// payload (claims)
{ "sub": "user-123", "iss": "https://auth.miapp.com", "aud": "api-pedidos",
  "exp": 1735689600, "iat": 1735686000, "roles": ["ADMIN"] }
```
- **Firmado ≠ cifrado.** El payload es Base64URL: **cualquiera puede leerlo**. Nunca pongas datos sensibles dentro.
- La firma garantiza **integridad y autenticidad**, no confidencialidad.
- **HS256** (secreto compartido, simétrico) vs **RS256** (clave privada firma / pública verifica, asimétrico). En microservicios se prefiere RS256: los servicios solo necesitan la clave pública (JWKS).

## 8.3 Sesión vs JWT 🎯

| | Sesión en servidor | JWT |
|---|---|---|
| Estado | Guardado en servidor/Redis | Stateless, va en el token |
| Escalado | Requiere sticky sessions o store compartido | Escala horizontal sin estado |
| Revocación | Inmediata (borras la sesión) | **Difícil**: el token vale hasta que expira |
| Tamaño | Cookie pequeña | Token grande en cada request |

**Mitigación de la revocación:** access tokens cortos (5–15 min) + **refresh token** largo, almacenado y revocable en servidor; lista de revocación (denylist) para casos críticos.

## 8.4 Dónde guardar el token en el frontend 🎯
- `localStorage`: simple, pero **vulnerable a XSS** (cualquier script lo lee).
- **Cookie `HttpOnly` + `Secure` + `SameSite=Strict/Lax`**: inaccesible a JS, mitiga XSS, pero requiere protección **CSRF**.
- Respuesta madura: *"Para apps con backend propio prefiero cookie HttpOnly con SameSite y CSRF token; si es un SPA contra API de terceros, access token en memoria y refresh en cookie HttpOnly."*

## 8.5 OAuth2 — roles y flujos 🎯

**Roles:** Resource Owner (usuario), Client (la app), Authorization Server (quien emite tokens), Resource Server (la API protegida).

| Flujo | Cuándo usarlo |
|---|---|
| **Authorization Code + PKCE** | SPAs y apps móviles. **Es el estándar actual.** |
| **Client Credentials** | Máquina a máquina (servicio → servicio), sin usuario |
| **Refresh Token** | Renovar el access token sin re-login |
| Implicit / Password (ROPC) | **Obsoletos**, no recomendarlos |

**OAuth2 vs OpenID Connect:** OAuth2 es **autorización** (dar acceso a un recurso). **OIDC** es una capa sobre OAuth2 que agrega **autenticación** e introduce el `id_token` (un JWT con la identidad del usuario). Si te preguntan "login con Google", la respuesta correcta es **OIDC**.

## 8.6 Flujo Authorization Code + PKCE (dibújalo)

```
1. Angular genera code_verifier + code_challenge
2. Redirige al Authorization Server con code_challenge
3. Usuario se autentica y consiente
4. AS redirige de vuelta con ?code=XYZ
5. Angular canjea code + code_verifier → access_token (+ refresh + id_token)
6. Angular llama a la API con Authorization: Bearer <access_token>
7. La API valida firma, iss, aud y exp contra el JWKS del AS
```
PKCE evita que un atacante que intercepte el `code` pueda canjearlo sin el `code_verifier`.

## 8.7 Spring Security — configuración moderna (Spring Boot 3, sin `WebSecurityConfigurerAdapter`) 🎯

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())                     // API stateless con Bearer
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**", "/actuator/health").permitAll()
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated())
            .oauth2ResourceServer(oauth -> oauth.jwt(Customizer.withDefaults()))
            .exceptionHandling(e -> e
                .authenticationEntryPoint((req, res, ex) -> res.sendError(401))
                .accessDeniedHandler((req, res, ex) -> res.sendError(403)))
            .build();
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();   // nunca MD5/SHA1 para contraseñas
    }
}
```

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://auth.miapp.com   # descarga el JWKS solo
```

**Filtro JWT propio** (cuando emites tus propios tokens, sin Authorization Server externo):

```java
public class JwtAuthFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res,
                                    FilterChain chain) throws ServletException, IOException {
        String header = req.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            if (jwtService.esValido(token)) {
                var auth = new UsernamePasswordAuthenticationToken(
                        jwtService.getSubject(token), null, jwtService.getAuthorities(token));
                SecurityContextHolder.getContext().setAuthentication(auth);
            }
        }
        chain.doFilter(req, res);
    }
}
```

## 8.8 Cadena de filtros de Spring Security 🎯
La petición pasa por una cadena: `SecurityContextPersistenceFilter` → filtros de autenticación (`UsernamePasswordAuthenticationFilter`, `BearerTokenAuthenticationFilter`) → `ExceptionTranslationFilter` → `FilterSecurityInterceptor/AuthorizationFilter`.
Piezas clave: `AuthenticationManager` → `AuthenticationProvider` → `UserDetailsService` → `SecurityContextHolder`.

## 8.9 Autorización a nivel de método
```java
@PreAuthorize("hasRole('ADMIN')")
public void borrar(Long id) { ... }

@PreAuthorize("#id == authentication.name or hasRole('ADMIN')")
public Usuario ver(String id) { ... }
```

## 8.10 Validaciones que un revisor espera escuchar
Al validar un JWT hay que comprobar: **firma**, **`exp`** (expiración), **`iss`** (emisor), **`aud`** (destinatario), algoritmo esperado (rechazar `alg: none` y evitar confusión HS/RS), y que el usuario siga activo.

## 8.11 OWASP Top 10 aplicado
Broken Access Control (validar autorización en el servidor, no ocultar botones), Inyección (consultas parametrizadas / JPA), XSS (sanitizar; Angular escapa por defecto), CSRF (tokens en flujos con cookie), configuración insegura (CORS abierto, actuator expuesto), secretos en el repo, dependencias vulnerables (`mvn dependency-check`, Dependabot).
