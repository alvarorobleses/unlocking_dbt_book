# Capítulo 2 — Instalación y configuración

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
-- Obtenido del libro: Data engineering with dbt a practical guide to building a cloud based pragmatic and dependable data platform with sql
```

## Flujo para inicializar el proyecto y comando debug
Para inicializar el proyecto, crear las carpetas y archivos por defecto ubicándonos en la carpeta del proyecto con 'cd' usaremos el comando
dbt init
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
      role: dbt_transformer
      schema: YOUR_NAME_HERE
      threads: 8
      type: snowflake
      user: YOUR_USER_HERE
      warehouse: dbt
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

## Notas personales

> Reflexiones o dudas del capítulo.

---

*Fuente: Unlocking dbt — Capítulo 2*
