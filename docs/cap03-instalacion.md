# Capítulo 3 — Instalación y configuración

## Lo que propone el libro

El libro instala dbt usando **pip + virtualenv**, que es el método clásico.

## Mi enfoque: usando uv en lugar de pip

Decidí usar **uv** porque es más rápido, moderno y reemplaza varias herramientas a la vez.
Ver [decisiones-tecnicas.md](decisiones-tecnicas.md) para más contexto.

### Flujo completo que usé

```bash
# 1. Crear carpeta y entrar
mkdir my_first_dbt_project
cd my_first_dbt_project
# 2. Crear entorno virtual
uv venv dbt-local-dev-env
# 3. Activar
source dbt-local-dev-env/bin/activate
# 4. Instalar dbt con adaptador Snowflake
uv pip install dbt-snowflake
# 5. Verificar
dbt --version
uv pip show dbt-snowflake
```

## Adaptadores disponibles

El libro menciona que dbt soporta múltiples data warehouses mediante adaptadores.
Algunos de los más comunes:

- `dbt-snowflake` — Snowflake (el que uso en este proyecto)
- `dbt-redshift` — Amazon Redshift
- `dbt-postgres` — PostgreSQL

Lista completa: [docs.getdbt.com/docs/supported-data-platforms](https://docs.getdbt.com/docs/supported-data-platforms)

## Configuración de snowflake previa
Obtenido del libro: Data engineering with dbt a practical guide to building a cloud based pragmatic and dependable data platform with sql
```bash
SHOW ROLES;
SHOW GRANTS TO ROLE ACCOUNTADMIN;
-- Cambiaremos el rol USERADMIN, no es conveniente usar ACCOUNTADMIN para tareas que no lo requieren, sino roles especializados, USERADMIN se encarga de agregar, eliminar editar roles y usuarios, cada objeto depende del rol que lo creo.
USE ROLE USERADMIN;
-- Crearemos el rol: DBT_EXECUTOR_ROLE se encargará de manejar proyectos DBT (manejar objetos de las bases de datos de proyectos dbt por ejemplo), ojo DBT_EXECUTOR es un usuario que usa snowflake/dbt para correr dbt en producción que tiene asignado el DBT_EXECUTOR_ROLE.
-- Se le puede asignar este rol a cualquier desarrollador que queramos que manejen todos los ambientes, o podriamos crear diferentes roles como para producción y otros para otros ambientes que queramos protegidos.
CREATE ROLE DBT_EXECUTOR_ROLE
    COMMENT = 'Role for the users running DBT models';
-- Ahora para trabajar más específicamente nos añadiremos el rol a nuestro usuario para luego usarlo
GRANT ROLE DBT_EXECUTOR_ROLE TO USER ALVARO9STYLE;
USE ROLE DBT_EXECUTOR_ROLE;
SELECT CURRENT_ROLE(); -- VER EL ROL ACTUAL
-- Asignaremos permisos a ese rol, cambiaremos el rol a sysadmin que tiene permisos de uso de bases de datos y warehouse, y puede asignar sus permisos 
USE ROLE SYSADMIN;
-- Asignamos permiso de crear bases de datos
GRANT CREATE DATABASE ON ACCOUNT
    TO ROLE DBT_EXECUTOR_ROLE;
-- Concedemos los privilegios de usar el poder de computo por default, el warehouse: COMPUTE_WH:}
GRANT USAGE ON WAREHOUSE COMPUTE_WH
    TO ROLE DBT_EXECUTOR_ROLE;
    
-- PG21. CREANDO LA PRIMERA BASE DE DATOS
USE ROLE DBT_EXECUTOR_ROLE;
SELECT CURRENT_ROLE(); -- DBT_EXECUTOR_ROLE
CREATE DATABASE DATA_ENG_DBT;
-- Reduciremos los créditos de consumo configurando de 10 a 1 minuto de suspensión para  el warehouse, para configurar el warehouse usaremos el rol SYSADMIN, el rol de dbt_executor solo tendrá el permiso de usarlo
USE ROLE SYSADMIN;
ALTER WAREHOUSE "COMPUTE_WH" SET
    WAREHOUSE_SIZE = 'XSMALL'
    AUTO_SUSPEND = 60
    AUTO_RESUME = TRUE
    COMMENT = 'Default Warehouse'
    
-- Creando un usuario para la aplicación de DBT, para crear un usuario cambiamos a useradmin
USE ROLE USERADMIN;
-- Crearemos el usuario para dbt llamado DBT_EXECUTOR
CREATE USER IF NOT EXISTS DBT_EXECUTOR
    COMMENT = 'User running DBT commands'
    PASSWORD = 'pick_a_password'
    DEFAULT_WAREHOUSE = 'COMPUTE_WH'
    DEFAULT_ROLE = 'DBT_EXECUTOR_ROLE'
;
-- Asignaremos el rol de executor al dbt user
GRANT ROLE DBT_EXECUTOR_ROLE TO USER DBT_EXECUTOR;
USE ROLE DBT_EXECUTOR_ROLE;

-- Consultas de data en SQL SINTAXIS Y OPERADORES
-- EXCLUDE Y RENAME son exclusivos de snowflake
SELECT * EXCLUDE (c3, c2) RENAME (c4 as cx, c5 as cy)
FROM table_1

-- Para el proceso de integración de dbt con snowflake, ésta es la estructura del account: twc21298.us-east-1
SELECT CURRENT_ACCOUNT(), CURRENT_REGION();
SELECT CURRENT_ACCOUNT_NAME(), CURRENT_REGION(), SYSTEM$ALLOWLIST();
```

## Creación de la Base de Datos y Esquema en Snowflake
Una vez configurados el warehouse, el rol y el usuario, es necesario crear la base de datos y el esquema que utilizará dbt para almacenar los objetos generados durante el desarrollo.
En este proyecto se utilizará la base de datos DBT_LEARNING y el esquema ALVARO.
### Crear la Base de Datos
```bash
CREATE DATABASE DBT_LEARNING;
```
La base de datos actúa como el contenedor principal de todos los objetos relacionados con el proyecto, incluyendo esquemas, tablas, vistas y otros recursos administrados por dbt.
### Crear el Esquema de Trabajo
```bash
USE DATABASE DBT_LEARNING;
CREATE SCHEMA YOUR_SCHEMA_HERE; 
```
El esquema permite organizar los objetos dentro de la base de datos. Durante las primeras etapas del proyecto, dbt utilizará este esquema para crear modelos, seeds y otros recursos definidos en el proyecto.
### Verificar la Creación
Comprobar que la base de datos existe:
```bash
SHOW DATABASES LIKE 'DBT_LEARNING';
```
Comprobar que el esquema fue creado correctamente:
```bash
SHOW SCHEMAS IN DATABASE DBT_LEARNING;
```
La configuración del archivo profiles.yml debe coincidir con los objetos creados en Snowflake:

## Flujo para inicializar el proyecto y comando debug
Para inicializar el proyecto, crear las carpetas y archivos por defecto realizaremos:
```bash
mkdir proyecto_dbt
cd proyecto_dbt
dbt init .
```
Se mostrará una serie de datos para que ingresemos referenciando a la configuración cloud con snowflake y del proyecto, creará en una carpeta oculta .dbt ubicada en el disco local de configuraciones junto a otras carpetas de otros programas dependiendo del sistema operativo, dentro un archivo llamado profiles.yml, el cual podemos volver a configurar, cabe mencionar que el proyecto usará por defecto este archivo, tenemos la opción de usar uno priorizable creándolo dentro de la carpeta del proyecto.
### Archivo profiles.yml
El archivo profiles.yml contiene las credenciales de nuestra cuenta cloud a la que vincularse en este caso de snowflake, con la previa configuración en snowflake se obtienen los datos para el llenado del archivo
```bash
my_first_dbt_project:
  target: dev
  outputs:
    dev:
      account: YOUR_ACCOUNT_HERE
      database: dbt_learning
      password: YOUR_PASSWORD_HERE
      role: DBT_EXECUTOR_ROLE
      schema: YOUR_NAME_HERE
      threads: 8
      type: snowflake
      user: YOUR_USER_HERE
      warehouse: COMPUTE_WH
```
### Archivo dbt_project.yml
El archivo dbt_project.yml contiene las configuraciones generales del proyecto, también será necesario para probar si todo fue correcto, probaremos la siguiente configuración base para el proyecto
```bash
name: 'my_first_dbt_project'
version: '1.0.0'
config-version: 2

# This setting configures which "profile" dbt uses for this project.
profile: 'my_first_dbt_project'

# These configurations specify where dbt should look for different types of files.
# The `model-paths` config, for example, states that models in this project can be
# found in the "models/" directory. You probably won't need to change these!
model-paths: ["models"]
analysis-paths: ["analyses"]
test-paths: ["tests"]
seed-paths: ["seeds"]
macro-paths: ["macros"]
snapshot-paths: ["snapshots"]

target-path: "target"  # directory which will store compiled SQL files
clean-targets:         # directories to be removed by `dbt clean`
  - "target"
  - "dbt_packages"

on-run-start: ["{{ create_logs_table('dbt_logs') }}"]
on-run-end: ["{{ insert_log_records('dbt_logs') }}"]

models:
  my_first_dbt_project:
    staging:
      +schema: staging
      crm:
        +schema: staging
      furniture_mart:
        +schema: staging
    intermediate:
      +schema: staging
      finance:
      operations:
    marts:
      finance:
        +schema: finance
      operations:
        +schema: operations

seeds:

snapshots:
```
### Probando la configuración...
Para comprobar que toda nuestra configuración local y cloud inicial fue correcta, utilizaremos el comando
```bash
dbt debug
```
<img width="1759" height="964" alt="debug" src="https://github.com/user-attachments/assets/25ad62f0-6a8e-4f73-b9a7-d699b12dfe0a" />

## Estructura del Directorio
Luego de crear el proyecto mediante el 'dbt init', dbt generó una estructura de directorios que organizará el proyecto
```bash
my_first_dbt_project/
├── analyses/    # Destinado a consultas sql para análisis, no forman parte del pipeline productivo
├── macros/    # Almacena funciones reutilizables escritas, facilita la automatización.
├── models/    # Aquí irán los modelos SQL que dbt transformará en tablas o vistas en snowflake.
├── seeds/
├── snapshots/
├── tests/
├── dbt_project.yml
├── profiles.yml
└── README.md
```
### Organización recomendada de models
En Unlocking dbt, los autores recomiendan organizar los modelos siguiendo una arquitectura de tres capas:
```bash
models/
├── staging/
├── intermediate/
└── marts/
```
La capa staging es la más cercana a los datos fuente, su objetivo es estandarizar nombres de columnas, corregir tipos de datos, aplicar limpieza básica,etc.
Los autores recomiendan organizar los modelos por una carpeta para cada sistema fuente y utilizar la convención:
```bash
stg_<fuente>_<entidad>
Ejemplo:
stg_crm_customers
stg_crm_orders
stg_furniture_mart_products
```
La capa intermediate contiene transformaciones más complejas, joins entre modelos, aplicación de reglas de negocio, reestructuración de datos.
```bash
stg_customers
stg_orders
      ↓
int_customer_orders
```
La capa marts contiene los modelos finales consumidos por usuarios de negocio, dashboards y herramientas de visualización.
```bash
marts/
├── finance/
    ├── mart_customer_revenue/
└── operations/
```
### Masterdata (opcional)
El libro propone añadir una carpeta adicional llamada masterdata, la cual no forma parte de la estructura estándar de dbt, pero resulta útil cuando existen dimensiones compartidas entre múltiples data marts, como por ejemplo cuando hablamos de los clientes, nos podemos preguntar ¿A qué área funcional debería pertenecer?.
```bash
models/
├── staging/
├── intermediate/
├── masterdata/
    ├── dim_customers.sql
    ├── dim_products.sql
    └── dim_stores.sql
└── marts/
```
La finalidad de esta carpeta es almacenar conjuntos de datos maestros (master data) que representan entidades centrales del negocio, como clientes, productos o tiendas. Estos modelos pueden ser reutilizados por varios data marts sin depender de un área funcional específica, también estas tablas suelen actuartarget/
dbt_packages/
logs/
dbt-local-dev-env/
profiles.yml
.env como fact tables (las que contienen los datos medibles).

## Notas personales

> No olvidar las buenas prácticas de no subir información confidencial a git hub, por ello crear un .gitignore en la carpeta del proyecto con la siguiente información:
```bash
target/
dbt_packages/
logs/
dbt-local-dev-env/
profiles.yml
.env
```
---

*Fuente: Unlocking dbt — Capítulo 2*
