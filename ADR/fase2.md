| # | Descripción del problema | Archivo | Línea aprox. | Principio violado | Riesgo |
|---|-------------------------|---------|--------------|-------------------|--------|
| 1 | El login/registro recibe la contraseña por query params (`@RequestParam p`), lo que expone credenciales en URL, logs e historial del navegador. | AuthController.java | 18–27 | Seguridad básica (mínima exposición) | Alto |
| 2 | No se usan DTOs ni `@RequestBody`; todo entra por parámetros sueltos, lo que hace más difícil validar los datos y mantener el código. | AuthController.java | 18–27 | Clean Code (contratos claros) | Medio |
| 3 | Las respuestas usan `Map<String,Object>` en lugar de un DTO de respuesta, lo que permite estructuras inconsistentes y posible exposición de datos. | AuthController.java | 16–28 | Clean Code (tipado/claridad) | Medio |
| 4 | Los endpoints propagan `throws Exception`, lo que genera manejo de errores poco controlado hacia el cliente. | AuthController.java | 18 y 25 | Clean Code (manejo de errores) | Medio |
| 5 | No hay validaciones básicas de entrada (usuario vacío, email inválido, contraseña débil) en el controller o mediante DTOs. | AuthController.java | 18–28 | Clean Code / Seguridad | Medio |
| 6 | No se evidencia ningún control de seguridad adicional (rate limit o bloqueo de intentos) en endpoints sensibles de autenticación. | AuthController.java | 16–30 | Seguridad básica | Medio |


FASE3

¿Qué datos sensibles aparecen en la respuesta? ¿Debería retornarse eso?

-En la respuesta del endpoint se devuelve el hash de la contraseña ingresada (MD5 del password), tanto cuando el login es correcto como cuando falla. Además, cuando la autenticación es exitosa, también se retorna el nombre de usuario (admin).

-No. Exponer hashes de contraseñas en una API es una mala práctica de seguridad, ya que facilita ataques offline y la posible filtración de credenciales derivadas. En un proceso de autenticación, lo correcto es devolver un token de sesión (por ejemplo, JWT) y únicamente la información mínima necesaria del usuario, nunca datos relacionados con contraseñas, ni siquiera en forma de hash.

¿Qué ocurrió? ¿Por qué es peligroso en producción?
-En el repositorio analizado, el backend construye la consulta SQL concatenando directamente el valor del parámetro u en la cadena de la consulta. Esto permite que un usuario malintencionado modifique la estructura del SQL mediante entradas manipuladas, lo que podría alterar la lógica de autenticación o acceder a información no autorizada, dependiendo del motor de base de datos y de la consulta final generada.

-Este tipo de vulnerabilidad permite ejecutar instrucciones SQL no previstas por el desarrollador, lo que puede derivar en accesos no autorizados, lectura de datos sensibles, modificación o incluso eliminación de información. En un entorno productivo, este riesgo puede comprometer completamente la seguridad del sistema y de los datos de los usuarios.

¿Cuál fue rechazado? ¿Es esa una validación suficiente?

-La contraseña 123 fue rechazada porque no cumple con la longitud mínima requerida (p.length() > 3). En cambio, la contraseña 1234 fue aceptada, ya que cumple con el requisito de longitud.

-No. Validar únicamente la longitud mínima de la contraseña no garantiza seguridad. Contraseñas como “1234” siguen siendo muy débiles y fáciles de adivinar. Además, el uso de MD5 para el hash de contraseñas es inseguro, ya que es un algoritmo obsoleto y vulnerable a ataques de fuerza bruta. Lo recomendable es usar algoritmos diseñados para contraseñas como bcrypt, scrypt o Argon2, junto con políticas de contraseñas más fuertes.

**¿Cuál es la situación que obliga a tomar una decisión?**
-El módulo de login tiene problemas serios de seguridad y diseño. Se están armando consultas SQL concatenando datos del usuario (riesgo de SQL Injection), se devuelven hashes de contraseñas en la respuesta y la validación de contraseñas es muy débil. Esto pone en riesgo los datos de los usuarios y deja una base mala para seguir creciendo el sistema


**¿Qué vas a cambiar y por qué lo elegiste?**
-Voy a refactorizar el login usando consultas parametrizadas u ORM para evitar inyecciones SQL, dejar de devolver cualquier dato sensible en las respuestas y cambiar MD5 por un hash seguro (como bcrypt). Elegí esto porque ataca directamente los problemas más críticos de seguridad y deja el módulo listo para crecer


**¿Qué ganas y qué pierdes con esta decisión?**
-Gano: más seguridad, código más limpio y una base más sólida para escalar.
Pierdo: tiempo de desarrollo y algunos ajustes en el frontend por cambios en la API.