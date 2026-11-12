# Arquitectura de Spring Security

## 1. ¿Qué es Spring Security?

Spring Security es un framework que permite proteger aplicaciones Spring Boot mediante mecanismos de autenticación y autorización.

Su funcionamiento se basa principalmente en una cadena de filtros que intercepta las solicitudes HTTP antes de que lleguen a los controladores.

Sus dos objetivos principales son:

- **Autenticación:** verificar quién está realizando la solicitud.
- **Autorización:** determinar qué acciones puede realizar ese usuario.

## 2. SecurityFilterChain

`SecurityFilterChain` representa una cadena de filtros de seguridad que se aplican a determinadas solicitudes HTTP.

Antes de que una solicitud llegue al controlador, atraviesa los filtros configurados en esta cadena.

Cada filtro puede realizar una tarea específica, como:

- Procesar las credenciales del usuario.
- Validar un token JWT.
- Gestionar la autenticación.
- Aplicar reglas de autorización.
- Interceptar solicitudes no permitidas.

Spring Security permite definir una o varias cadenas `SecurityFilterChain`, dependiendo de las necesidades de la aplicación.

## 3. ¿Qué es un Filter?

Un `Filter` es un componente de Java que permite interceptar solicitudes y respuestas HTTP durante su procesamiento.

Puede ejecutar diferentes acciones:

- Examinar la solicitud recibida.
- Validar información.
- Modificar datos de la solicitud o respuesta.
- Permitir que la solicitud continúe.
- Detener el procesamiento y generar una respuesta.

Para permitir que la solicitud avance al siguiente componente de la cadena se utiliza:

```java
chain.doFilter(request, response);
```

Si no se invoca este método, la solicitud no continúa por la cadena de filtros.

Un filtro también puede ejecutar instrucciones después de `chain.doFilter()`, cuando el procesamiento regresa desde los componentes posteriores.

## 4. FilterChainProxy

`FilterChainProxy` es un filtro interno de Spring Security que actúa como intermediario entre las solicitudes HTTP y las cadenas `SecurityFilterChain`.

Su responsabilidad principal es identificar qué cadena de seguridad debe utilizarse para procesar cada solicitud.

Cuando existen varias cadenas, `FilterChainProxy` evalúa sus condiciones de coincidencia y utiliza la primera `SecurityFilterChain` cuyo criterio coincide con la solicitud.

Por ejemplo, una aplicación podría tener:

- Una cadena para las rutas `/api/**`.
- Otra cadena para las rutas `/admin/**`.

Cada cadena puede tener sus propios filtros y reglas de seguridad.

## 5. RequestMatcher y securityMatcher()

Spring Security utiliza criterios de coincidencia para determinar qué solicitudes deben ser atendidas por una cadena de seguridad.

El método `securityMatcher()` permite establecer el alcance de una `SecurityFilterChain`.

Por ejemplo:

```java
http.securityMatcher("/api/**");
```

Esta configuración indica que la cadena atenderá las solicitudes cuya ruta coincida con `/api/**`.

Es importante diferenciar dos conceptos:

- **`securityMatcher()`:** determina qué solicitudes procesa una cadena de seguridad.
- **`requestMatchers()`:** establece reglas de autorización para las solicitudes que ya pertenecen a esa cadena.

## 6. HttpSecurity

`HttpSecurity` es el objeto que proporciona Spring Security para configurar los mecanismos de protección de una aplicación.

A través de este objeto se pueden establecer:

- Las rutas que serán protegidas.
- Las reglas de autorización.
- Los mecanismos de autenticación.
- La protección CSRF.
- La incorporación y ubicación de filtros.

Spring proporciona este objeto mediante inyección de dependencias, por lo que no es necesario crearlo manualmente.

### Método build()

Cuando se termina de configurar `HttpSecurity`, se utiliza:

```java
return http.build();
```

Este método construye y devuelve un objeto `SecurityFilterChain` con la configuración establecida.

Para que Spring registre esa cadena como un componente administrado por el contenedor, el método de configuración debe estar anotado con `@Bean`.

## 7. Filtros incorporados de Spring Security

Spring Security incorpora diferentes filtros según los mecanismos de seguridad que se configuren.

Algunos ejemplos son:

| Filtro | Responsabilidad |
|---|---|
| `BasicAuthenticationFilter` | Procesa la autenticación mediante HTTP Basic. |
| `UsernamePasswordAuthenticationFilter` | Procesa credenciales de usuario y contraseña en los mecanismos que utilizan este filtro. |
| `ExceptionTranslationFilter` | Gestiona la traducción de excepciones de seguridad en respuestas HTTP o acciones de autenticación. |
| `AuthorizationFilter` | Aplica las reglas de autorización configuradas. |

No todos los filtros están presentes en todas las aplicaciones. Su incorporación depende de la configuración de seguridad.

Spring Security también administra el orden en el que deben ejecutarse estos filtros.

### Incorporar filtros personalizados

Cuando una aplicación necesita implementar un mecanismo propio, es posible agregar filtros personalizados a la cadena.

Para ello se utilizan métodos como:

- `addFilterBefore()`: agrega un filtro antes de otro.
- `addFilterAfter()`: agrega un filtro después de otro.
- `addFilterAt()`: intenta ubicar un filtro en la misma posición que otro.

Estos mecanismos permiten integrar funcionalidades adicionales, como la validación de tokens JWT.

## 8. Autenticación y autorización

Aunque están relacionadas, son responsabilidades diferentes.

### Autenticación

Responde a la pregunta:

**¿Quién eres?**

Consiste en verificar la identidad del usuario mediante credenciales u otros mecanismos de autenticación.

Por ejemplo, Spring Security puede utilizar un nombre de usuario y una contraseña para comprobar la identidad de quien realiza una solicitud.

### Autorización

Responde a la pregunta:

**¿Qué puedes hacer?**

Una vez identificado el usuario, Spring Security determina qué recursos o acciones tiene permitidos.

Para ello se pueden establecer reglas basadas en roles o autoridades.

Por ejemplo:

- `USER`: puede consultar productos.
- `ADMIN`: puede consultar, crear, actualizar y eliminar productos.

### hasRole() y hasAnyRole()

Spring Security proporciona métodos para definir permisos basados en roles:

```java
.hasRole("ADMIN")
```

Permite el acceso únicamente a usuarios que tengan el rol `ADMIN`.

```java
.hasAnyRole("USER", "ADMIN")
```

Permite el acceso a usuarios que tengan cualquiera de los roles indicados.

Cuando se utiliza `.roles("ADMIN")` al crear un usuario, Spring Security registra internamente la autoridad `ROLE_ADMIN`.

Por esta razón, al utilizar `hasRole("ADMIN")` no se debe agregar manualmente el prefijo `ROLE_`.

## 9. Respuestas de seguridad HTTP

Durante el proceso de autenticación y autorización pueden presentarse diferentes respuestas:

| Código HTTP | Significado |
|---|---|
| `200 OK` | La solicitud fue procesada correctamente. |
| `401 Unauthorized` | No se proporcionaron credenciales válidas o no se pudo autenticar al usuario. |
| `403 Forbidden` | El usuario está autenticado, pero no tiene permisos para realizar la acción solicitada. |

La diferencia entre `401` y `403` es fundamental: el primero está relacionado con la autenticación y el segundo con la autorización.

## 10. Resumen de los componentes principales

| Componente | Función principal |
|---|---|
| `Filter` | Intercepta y procesa solicitudes y respuestas HTTP. |
| `FilterChain` | Permite continuar el procesamiento entre filtros. |
| `SecurityFilterChain` | Define los filtros y mecanismos de seguridad de una cadena. |
| `FilterChainProxy` | Selecciona la cadena de seguridad correspondiente a cada solicitud. |
| `RequestMatcher` | Evalúa si una solicitud cumple determinados criterios. |
| `securityMatcher()` | Define qué solicitudes atiende una cadena. |
| `HttpSecurity` | Permite configurar los mecanismos de seguridad. |
| `build()` | Construye la cadena de filtros configurada. |
| `requestMatchers()` | Permite establecer reglas de autorización para determinadas rutas. |

**Idea fundamental:** Spring Security intercepta las solicitudes HTTP mediante filtros, selecciona la cadena de seguridad correspondiente y aplica los mecanismos de autenticación y autorización antes de permitir el acceso a los recursos de la aplicación.