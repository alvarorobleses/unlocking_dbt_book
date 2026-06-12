# Capítulo 5: Models

## Objetivos de aprendizaje

Al finalizar este capítulo serás capaz de:

- Comprender qué es un modelo en dbt.
- Entender el ciclo de vida de un modelo.
- Diferenciar entre tablas y vistas materializadas por dbt.
- Utilizar la función `ref()` para crear dependencias entre modelos.
- Organizar modelos siguiendo buenas prácticas.
- Comprender el enfoque de capas recomendado por los autores.

---

# ¿Qué es un Modelo?

Los modelos constituyen el componente central de dbt. Un modelo es simplemente un archivo SQL almacenado dentro del directorio `models/` que contiene una consulta `SELECT`.

Cuando se ejecuta dbt, estas consultas son compiladas y transformadas en objetos dentro del Data Warehouse, normalmente tablas o vistas.

Por ejemplo:

```sql
SELECT
    customer_id,
    first_name,
    last_name
FROM raw.customers
```

Al ejecutar dbt, esta consulta puede convertirse en una vista o tabla dentro de Snowflake.

---

# El Directorio Models

Todos los modelos se almacenan dentro del directorio:

```text
models/
```

Cuando dbt ejecuta el proyecto, analiza este directorio y construye cada modelo según las dependencias definidas.

Por esta razón, el directorio `models` suele considerarse el corazón del proyecto dbt.

---

# Filosofía de los Modelos

Una de las ideas principales presentadas en *Unlocking dbt* es que los modelos deben representar pasos lógicos de transformación de datos.

En lugar de construir una única consulta SQL enorme y difícil de mantener, se recomienda dividir la lógica en múltiples modelos pequeños y reutilizables.

Esto permite:

- Mayor legibilidad.
- Facilidad de mantenimiento.
- Reutilización de transformaciones.
- Mejor documentación.
- Depuración más sencilla.

---

# Referenciando Modelos

dbt permite conectar modelos utilizando la función `ref()`.

Supongamos que existe un modelo llamado:

```text
stg_customers.sql
```

Otro modelo puede utilizarlo mediante:

```sql
SELECT *
FROM {{ ref('stg_customers') }}
```

La función `ref()` ofrece varias ventajas:

- Construye automáticamente las dependencias.
- Permite cambiar nombres físicos sin modificar consultas.
- Facilita la construcción del DAG.
- Garantiza el orden correcto de ejecución.

---

# El DAG de dbt

Cada vez que utilizamos `ref()`, dbt genera relaciones entre modelos.

Por ejemplo:

```text
stg_customers
        │
        ▼
int_customer_orders
        │
        ▼
dim_customers
```

Estas dependencias forman el DAG (*Directed Acyclic Graph*), una representación visual del flujo de transformación de datos.

El DAG es uno de los conceptos fundamentales dentro de dbt.

---

# Materializaciones

Cuando dbt ejecuta un modelo, debe decidir cómo crear el objeto resultante en el Data Warehouse.

Este comportamiento se controla mediante las materializaciones.

Las más comunes son:

- View
- Table
- Incremental
- Ephemeral

---

# Materialización View

Las vistas almacenan únicamente la consulta SQL.

Cada vez que un usuario consulta la vista, Snowflake ejecuta nuevamente la lógica definida.

Ejemplo:

```yaml
models:
  my_first_dbt_project:
    +materialized: view
```

Ventajas:

- Menor almacenamiento.
- Actualización inmediata.

Desventajas:

- Puede incrementar el tiempo de consulta.

---

# Materialización Table

Las tablas almacenan físicamente los resultados de la consulta.

Cada ejecución de dbt reconstruye completamente la tabla.

Ejemplo:

```yaml
models:
  my_first_dbt_project:
    +materialized: table
```

Ventajas:

- Consultas más rápidas.
- Menor carga computacional durante la lectura.

Desventajas:

- Mayor consumo de almacenamiento.

---

# Configuración de Materializaciones

La materialización puede definirse dentro del modelo:

```sql
{{ config(materialized='table') }}

SELECT *
FROM {{ ref('stg_customers') }}
```

O de manera centralizada dentro de `dbt_project.yml`.

---

# Organización Recomendada de Modelos

Los autores recomiendan organizar los modelos en tres capas principales:

```text
models/
│
├── staging/
├── intermediate/
└── marts/
```

Esta estructura facilita el mantenimiento y escalabilidad del proyecto.

---

# Staging

La capa de staging representa la primera transformación sobre los datos fuente.

Sus principales características son:

- Relación cercana con los Sources.
- Estandarización de nombres.
- Limpieza básica de datos.
- Una tabla de entrada suele producir un modelo de staging.

Ejemplo:

```text
stg_crm_customers.sql
```

---

# Intermediate

La capa intermedia contiene transformaciones más complejas.

Aquí suelen realizarse:

- Joins.
- Agregaciones.
- Reglas de negocio.
- Cálculos intermedios.

Estos modelos normalmente no son consumidos directamente por usuarios finales.

Ejemplo:

```text
int_customer_orders.sql
```

---

# Marts

Los marts representan los modelos finales utilizados para análisis y reporting.

Generalmente siguen diseños dimensionales y estructuras optimizadas para consumo.

Ejemplos:

```text
dim_customers.sql
fct_orders.sql
```

Según los autores, gran parte de los proyectos dbt terminan adoptando algún tipo de modelado dimensional o esquema estrella.

---

# Convenciones de Nomenclatura

El libro recomienda utilizar prefijos que permitan identificar rápidamente el propósito de cada modelo.

Ejemplos:

```text
stg_customers
stg_orders

int_customer_orders

dim_customers
fct_orders
```

Esta convención mejora la comprensión del proyecto a medida que crece.

---

# Ejecutando Modelos

Para ejecutar todos los modelos:

```bash
dbt run
```

Para ejecutar un modelo específico:

```bash
dbt run --select stg_customers
```

Para ejecutar un modelo y sus dependencias:

```bash
dbt run --select +stg_customers
```

---

# Buenas Prácticas

Según las recomendaciones presentadas en *Unlocking dbt*:

- Mantener modelos pequeños y enfocados.
- Evitar consultas extremadamente largas.
- Utilizar `ref()` en lugar de nombres físicos.
- Organizar modelos por capas.
- Documentar cada transformación importante.
- Favorecer la reutilización de lógica.

---

# Resumen

En este capítulo se estudió el componente más importante de dbt: los modelos.

Se revisó:

- Qué es un modelo.
- Cómo funciona `ref()`.
- El concepto de DAG.
- Las principales materializaciones.
- La organización en capas recomendada por los autores.
- Las buenas prácticas para construir proyectos mantenibles.

Los modelos constituyen el núcleo de dbt y permiten transformar datos de manera modular, documentada y escalable dentro del Data Warehouse.
