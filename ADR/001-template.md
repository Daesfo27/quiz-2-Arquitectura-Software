# ADR-001: [Título de la decisión]
## Contexto
## Decisión
## Consecuencias
## Alternativas consideradas


-------------------------------------------------------------------------------------------------

---

# 📄 `ADR/001-template.md` (ADR COMPLETO)

```md

Fase4

# ADR-001: Refactor del módulo de autenticación para corregir fallas de seguridad y Clean Code

## Contexto
El sistema actual expone endpoints de autenticación (`/login` y `/register`) recibiendo credenciales por query params, lo que deja usuario y contraseña visibles en URL, logs del servidor y herramientas de monitoreo. Además, el controller devuelve respuestas como `Map<String,Object>` sin un contrato claro (DTO), propaga errores con `throws Exception` y no aplica validaciones básicas de entrada.  
En las pruebas funcionales se observó que el registro acepta contraseñas débiles como `1234` y que, ante un intento de SQL Injection en el login, la API expone información interna como un `hash` en la respuesta. Estos problemas son críticos porque el módulo de autenticación es una puerta de entrada al sistema; una falla aquí afecta directamente a los usuarios, al equipo de desarrollo (mantenibilidad y bugs) y al negocio (riesgo de seguridad y reputación).

## Decisión
1) **Cambiar entrada por DTOs con RequestBody:** reemplazar `@RequestParam` por `@RequestBody` usando `LoginRequest` y `RegisterRequest`, habilitando validaciones con anotaciones (`@NotBlank`, `@Email`, `@Size`).  
2) **Eliminar riesgos de SQL Injection:** prohibir SQL concatenado en repositorios y usar consultas parametrizadas (PreparedStatement) o Spring Data JPA para separar datos de la consulta.  
3) **Proteger contraseñas:** almacenar contraseñas únicamente con hash seguro (BCrypt/Argon2) y nunca retornar password ni hash en las respuestas.  
4) **Estandarizar manejo de errores:** implementar manejo centralizado de excepciones (`@ControllerAdvice`) para respuestas HTTP consistentes y sin filtrar detalles internos.  
5) **Respuestas mínimas y seguras:** devolver solo la información necesaria en login (por ejemplo `userId`/`username` o token), evitando exponer datos internos.

## Consecuencias
**Positivas:**  
- Reducción significativa del riesgo de exposición de credenciales y ataques de inyección.  
- Código más mantenible y legible con DTOs y validaciones claras.  
- Manejo de errores consistente y profesional.  
- Facilita pruebas unitarias e integración.

**Negativas / riesgos:**  
- Tiempo adicional de refactor y necesidad de pruebas para evitar regresiones.  
- Cambios en el contrato del API pueden afectar clientes existentes.  
- Posible necesidad de migrar contraseñas existentes a hash seguro.

## Alternativas consideradas
1) **Corregir solo la seguridad (SQL Injection y contraseñas) sin refactor de Clean Code:** descartado porque mantiene problemas de mantenibilidad y riesgo de reintroducir fallas.  
2) **Reescribir el módulo desde cero:** descartado por tiempo y riesgo de no cumplir la entrega; se prefiere refactor incremental.





