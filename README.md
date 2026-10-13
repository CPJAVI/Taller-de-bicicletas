# Taller de bicicletas

Prueba técnica: prototipo de gestión de reparaciones para un taller de bicicletas.

Todo el proyecto vive en un único archivo `taller.html`. No necesita servidor, ni instalación, ni dependencias. Los datos se guardan en el `localStorage` del navegador.

---

## Cómo ejecutarlo

1. Descarga o clona el repositorio. Hazlo ejecutando el comando: `git clone https://github.com/CPJAVI/Taller-de-bicicletas.git`
2. Abre `taller.html` haciendo clic.
3. Ya puedes usar la aplicación.

No hace falta instalar nada por ahora dado que es un ejercicio sencillo. Pero de cara al futuro estaría dispuesto a trabajar para desarrollar un backend (ver *Mejoras futuras*).

---

## Funcionamiento

### Roles

Hay dos roles: **cliente** y **administrador**.

- **Administrador**: usuario `admin`, contraseña `admin`. Es un usuario fijo definido en el código, no se registra ni se guarda en `localStorage`.
- **Cliente**: se registra él mismo desde la pantalla de inicio (si no hay cliente debes crear cuenta).

Según con qué rol entres, la aplicación te lleva a un panel distinto.

### Registro de cliente

Al crear una cuenta se pide:

- **Usuario**: identificador de acceso. No puede repetirse ni ser `admin`. Mínimo 3 caracteres.
- **Contraseña**: mínimo 4 caracteres. Se pide dos veces para evitar erratas.
- **Teléfono de contacto**: solo dígitos, mínimo 9.

Al confirmar, el sistema asigna automáticamente al cliente un **código de cliente** de 6 dígitos, único entre todos los clientes registrados.

### Panel del cliente

- En la barra superior se muestra su **código de cliente**.
- Puede **crear una reparación** indicando la descripción de la bicicleta y el problema comunicado. Al crearla, se le asigna un **código de reparación** de 6 dígitos, único entre todas las órdenes.
- Ve el **historial** de sus reparaciones con su código de reparación, su código de cliente, la bicicleta, el problema, el estado y la fecha de creación.
- Cada reparación tiene una columna **Historial** desplegable donde puede consultar **todos los cambios de estado** que ha sufrido: estado, fecha y quién lo cambió (para este caso saldrá al principio por defecto pendiente de nombre de cliente y los demás cambios aparecerá admin, se espera que esto evolucione en caso de llegar a crearse los usuarios mecánicos para que en vez de admin ponga nombre del mecánico que lo cambió).
- Puede **cerrar sesión**.
- Puede **eliminar su cuenta**. Al hacerlo se borran también todas sus reparaciones.

Si el admin cambia el estado de una de sus reparaciones, el cliente lo verá la próxima vez que inicie sesión, tanto en la columna Estado como en su historial.

### Panel del administrador

- Ve **todas las órdenes de reparación**, con su código de reparación, el código del cliente que la creó, la bicicleta, el problema, el estado y la fecha.
- Puede **cambiar el estado** de cualquier reparación entre `Pendiente`, `En reparación` y `Terminada`. Cada cambio queda registrado automáticamente en el historial de esa reparación.
- Cada reparación tiene una columna **Historial** desplegable con todos los cambios de estado registrados (estado, fecha y usuario que lo hizo; para esto último verá él en todos los cambios salvo el "Pendiente" inicial que será atribuido al usuario que añadió la orden de reparación).
- Ve la lista de **clientes registrados**, con su código de cliente, usuario, teléfono de contacto y número de reparaciones.
- Puede **eliminar la cuenta de un cliente**. Al hacerlo se borran también todas sus reparaciones.
- Puede **cerrar sesión**.

---

## Estados de una reparación

Solo el administrador puede cambiar el estado. Los valores posibles son:

- `Pendiente` (estado inicial al crear la orden)
- `En reparación`
- `Terminada`

Cada cambio queda registrado en el **historial de la reparación**, con:

- El **estado** al que se ha cambiado.
- La **fecha** del cambio.
- El **usuario** que lo ha hecho (el cliente cuando la crea, el admin cuando la actualiza).

---

## Almacenamiento

Todo se guarda en el `localStorage` del navegador bajo tres claves:

- `taller_usuarios_v1` — clientes registrados (usuario, contraseña, teléfono, código de cliente).
- `taller_ordenes_v1` — órdenes de reparación (código, usuario interno del cliente, código de cliente, bicicleta, problema, estado, fecha de creación e historial de cambios).
- `taller_sesion_v1` — sesión activa (usuario y rol).

Cada orden incluye un array `historial` con entradas del tipo:

    { "estado": "En reparación", "fecha": "2026-10-13T11:30:00.000Z", "usuario": "admin" }

Si quieres empezar de cero, abre las herramientas de desarrollo del navegador → Application → Local Storage y borra esas claves.

---

## Privacidad y datos

- **No se usan datos personales reales**. Todo lo que se introduce en el prototipo es ficticio.
- El **teléfono de contacto** se guarda en la ficha del cliente, no se copia en cada orden. En las órdenes se propaga únicamente el **código de cliente**.
- El **código de cliente** y el **código de reparación** son identificadores internos del taller, no datos personales.
- En este prototipo el administrador (el taller) es quien ve el teléfono. En un producto real, solo el personal con rol de recepción o administración debería poder consultar datos de contacto; los técnicos solo necesitarían el código, la bicicleta, el problema y el estado.
- Las contraseñas se guardan **en claro** en `localStorage` porque es un prototipo sin backend. En producción habría que usar un servidor con base de datos y guardar solo el hash de la contraseña (por ejemplo bcrypt) con su sal.

### Cómo relacionaría las reparaciones con los clientes si hubiera datos de contacto

Si el producto necesitara guardar datos de contacto reales (nombre, teléfono, email), el modelo sería:

- Una tabla **clientes** con: `cliente_id` (el código de 6 dígitos que ya se usa), datos de contacto y fecha de alta.
- Una tabla **órdenes** que guarda solo `cliente_id`, más los datos de la reparación.
- Las órdenes **nunca** duplicarían el teléfono ni el nombre; siempre se resolvería la relación a través del `cliente_id`.

Acceso a esos datos:

- **Recepción / administración**: puede ver y editar los datos de contacto, porque los necesita para avisar al cliente.
- **Mecánicos / taller**: solo ven el código de cliente, la bicicleta, el problema y el estado. No necesitan el teléfono ni el nombre para reparar.
- **Cliente**: solo ve sus propios datos y sus propias reparaciones.

Medidas de protección que aplicaría:

- Cifrado en tránsito (HTTPS) y en reposo.
- Registro de accesos a datos de contacto (quién los consultó y cuándo).
- Principio de mínimo privilegio: cada rol accede solo a lo que necesita.
- Política de retención y borrado: al eliminar un cliente, se eliminan también sus datos de contacto y sus reparaciones.
- Consentimiento explícito del cliente para el uso de sus datos de contacto.

---

## Limitaciones conocidas

- Sin backend: los datos viven en el navegador y no se comparten entre dispositivos ni usuarios.
- Contraseñas en claro en `localStorage`.
- El usuario `admin` está fijado en el código.
- No hay filtros ni búsqueda en las tablas.

---

## Mejoras futuras

- Backend con API REST y base de datos real.
- Posibilidad de sustituir el admin y en su lugar crear dos tipos de usuarios: clientes (con las funciones de clientes que tienen en esta ocasión) y mecánicos (mismas funciones que admin para reparar bicicletas y consultar datos de clientes).
- Hash de contraseñas con bcrypt o similar.
- Filtros por estado, cliente o fecha en el panel del admin.
- Búsqueda por código de reparación o de cliente.
- Exportar órdenes a CSV.
- Tests automatizados.
- Gestión de varios administradores con roles diferenciados.

---

## Uso de IA

Se ha utilizado la IA de DeepSeek como apoyo para estructurar la solución, revisar el código y redactar esta documentación. No obstante, las decisiones finales y las comprobaciones se han hecho manualmente.

Pruebas realizadas: creación de cuentas, creación de órdenes, cambio de estados desde el admin, persistencia tras recargar, borrado en cascada al eliminar un cliente, validación de teléfono y contraseñas, comportamiento del registro frente a usuarios duplicados, y registro correcto de cada cambio de estado en el historial de la reparación.

No se han introducido datos personales reales ni información confidencial. El tiempo empleado ha sido aproximadamente 1 hora.
