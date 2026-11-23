# Trabajo Práctico — Base de Datos y Normalización

## Descripción del proyecto

**AutoMarket** es una aplicación web para consultar vehículos publicados, gestionar usuarios y administrar favoritos. Este trabajo aplica los principios de normalización y crea el esquema relacional de MySQL utilizado por el proyecto.

## Parte 1: Normalización aplicada

### Registro hipotético sin normalizar

Antes de normalizar, un vehículo podría registrarse junto con el usuario y sus datos dentro de una única fila:

| vehiculo_id | usuario | email | rol | titulo | precio | anio | categoria | transmision | combustible | favorito_por |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | Bautista Salto | bautista@mail.com | administrador | Toyota Corolla | 8500 | 2022 | Sedan | Automática | Nafta | Ana, Lucas, Sofía |

El campo `favorito_por` contiene múltiples valores y genera información repetida. También se repiten datos de usuario y categoría en varias publicaciones.

### Primera Forma Normal — 1FN

Se eliminan los grupos repetitivos y se deja un único valor en cada campo. La relación de favoritos se representa mediante registros separados:

| vehiculo_id | usuario_id | titulo | precio | anio | categoria_nombre | transmision | combustible |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 1 | Toyota Corolla | 8500 | 2022 | Sedán y Hatchback | Automática | Nafta |

| usuario_id | vehiculo_id |
| --- | --- |
| 2 | 1 |
| 3 | 1 |
| 4 | 1 |

### Segunda Forma Normal — 2FN

Los datos que pertenecen al usuario no dependen del vehículo, sino de `usuario_id`. Por ello se separan en la tabla `usuarios`.

Los datos de la categoría no dependen del vehículo, sino de `categoria_id`. Por ello se separan en la tabla `categorias`.

Las entidades principales quedan separadas y la tabla `vehiculos` contiene claves foráneas hacia ellas:

- `usuarios`: datos personales y autenticación.
- `categorias`: datos maestros de clasificación.
- `vehiculos`: datos propios de la publicación.

Como `favoritos` relaciona dos entidades con una clave compuesta lógica, contiene las claves `usuario_id` y `vehiculo_id` y evita repetir favoritos entre un usuario y un vehículo.

### Tercera Forma Normal — 3FN

Se eliminan las dependencias transitivas. Por ejemplo, el nombre de una categoría no debe repetirse dentro de cada vehículo si ese dato depende de la categoría:

- `vehiculos.categoria_id` → `categorias.id` → `categorias.nombre_categoria`.

De la misma forma, el nombre y correo pertenecen a `usuarios`, mientras que cada vehículo tiene únicamente la referencia `usuario_id` de su publicador.

Las relaciones resultantes son:

- Un usuario puede publicar muchos vehículos.
- Una categoría puede contener muchos vehículos.
- Un usuario puede marcar muchos vehículos como favoritos.
- Un vehículo puede ser marcado como favorito por muchos usuarios.

## Diagrama y tablas normalizadas

### `usuarios`

- **PK:** `id`
- `nombre`
- `email`
- `password`
- `rol`
- `created_at`
- `updated_at`

### `categorias`

- **PK:** `id`
- `nombre_categoria`
- `created_at`

### `vehiculos`

- **PK:** `id`
- **FK:** `usuario_id` → `usuarios(id)`
- **FK:** `categoria_id` → `categorias(id)`
- `titulo`
- `precio`
- `anio`
- `kilometros`
- `transmision`
- `combustible`
- `descripcion`
- `imagen`
- `created_at`
- `updated_at`

### `favoritos`

- **PK:** `id`
- **FK:** `usuario_id` → `usuarios(id)`
- **FK:** `vehiculo_id` → `vehiculos(id)`
- Restricción única sobre `(usuario_id, vehiculo_id)`
- `created_at`

## Scripts SQL

El esquema se encuentra en:

```text
includes/schema.sql
```

Los datos de prueba y las operaciones CRUD se encuentran en:

```text
includes/consultas.sql
```

### Base de datos

El script crea la base `automarket` y configura la codificación UTF-8:

```sql
CREATE DATABASE IF NOT EXISTS automarket
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;
USE automarket;
```

## Operaciones CRUD

### Create

Se insertan categorías maestras, un usuario y dos vehículos vinculados mediante claves foráneas.

### Read

Incluye:

- `SELECT` con `WHERE` y `ORDER BY` para filtrar vehículos por precio.
- `INNER JOIN` entre `vehiculos`, `usuarios` y `categorias` para mostrar el vehículo, su vendedor y su categoría.

### Update

Se actualiza un vehículo específico utilizando `WHERE id = 1`.

### Delete

Las eliminaciones deben ejecutarse únicamente con una cláusula `WHERE` que identifique el registro a borrar. Las claves foráneas están configuradas con `ON DELETE CASCADE` o `ON DELETE SET NULL`, según corresponda.

## Integridad referencial

Se utilizan claves primarias `INT AUTO_INCREMENT` y claves foráneas para mantener relaciones válidas:

- `vehiculos.usuario_id` referencia `usuarios.id`.
- `vehiculos.categoria_id` referencia `categorias.id`.
- `favoritos.usuario_id` referencia `usuarios.id`.
- `favoritos.vehiculo_id` referencia `vehiculos.id`.

La eliminación de usuarios o vehículos elimina sus relaciones dependientes mediante `ON DELETE CASCADE`. Si se elimina una categoría, los vehículos que la tenían quedan sin categoría mediante `ON DELETE SET NULL`.

## Índices

Se definieron índices explícitos para mejorar las búsquedas frecuentes:

- `idx_usuario_email`: búsqueda de usuarios por correo electrónico.
- `idx_vehiculo_precio`: filtrado y ordenamiento de vehículos por precio.

El campo `email` también posee una restricción `UNIQUE`.

## Ejecución en XAMPP

1. Iniciar Apache y MySQL desde el panel de XAMPP.
2. Abrir phpMyAdmin.
3. Importar `includes/schema.sql`.
4. Importar o ejecutar las sentencias de `includes/consultas.sql`.
5. Verificar las tablas desde el panel izquierdo de phpMyAdmin.
6. Ejecutar las consultas de prueba desde la pestaña **SQL**.

## Evidencias

### Creación del esquema

![Tablas creadas en phpMyAdmin](docs/screenshots/tp-base-datos-tablas.png)

### Inserciones de prueba

![Datos insertados en las tablas](docs/screenshots/tp-base-datos-inserts.png)

### Consulta con INNER JOIN

![Consulta relacional de vehículos](docs/screenshots/tp-base-datos-join.png)

### Operaciones UPDATE y DELETE

![Prueba de actualización y eliminación](docs/screenshots/tp-base-datos-crud.png)

## Verificación realizada

- [ ] Base de datos creada sin errores.
- [ ] Tablas creadas con claves primarias autoincrementales.
- [ ] Relaciones y claves foráneas configuradas.
- [ ] Categorías maestras insertadas.
- [ ] Usuarios y vehículos insertados.
- [ ] Consulta con `WHERE` y `ORDER BY` probada.
- [ ] Consulta `INNER JOIN` probada.
- [ ] `UPDATE` ejecutado con `WHERE`.
- [ ] `DELETE` ejecutado con `WHERE`.
- [ ] Índices creados correctamente.

## Tecnologías utilizadas

- MySQL / MariaDB
- phpMyAdmin
- XAMPP
- SQL
- PHP 8.x
- Git y GitHub
