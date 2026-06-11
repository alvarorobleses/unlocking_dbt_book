# Capítulo 3: Sources y Seeds

## Objetivos de aprendizaje

Al finalizar este capítulo serás capaz de:

- Comprender qué son los Sources en dbt.
- Definir fuentes de datos mediante archivos YAML.
- Referenciar tablas fuente dentro de modelos dbt.
- Configurar y ejecutar verificaciones de frescura de datos.
- Comprender qué son los Seeds y cuándo utilizarlos.
- Cargar archivos CSV al Data Warehouse mediante dbt.

---

# ¿Qué son los Sources?

Los Sources representan tablas existentes dentro del Data Warehouse desde las cuales dbt leerá información. A diferencia de los modelos, los Sources no son creados por dbt; simplemente permiten referenciar datos ya disponibles en la plataforma.

En lugar de escribir repetidamente el nombre de la base de datos, esquema y tabla dentro de cada consulta SQL, dbt permite definir estas ubicaciones una sola vez en un archivo YAML y reutilizarlas a lo largo del proyecto.

Según las recomendaciones de *Unlocking dbt*, esta práctica mejora la mantenibilidad del proyecto y facilita la administración de múltiples entornos como desarrollo, pruebas y producción.

---

# Definiendo un Source

Por convención, los Sources suelen declararse dentro de un archivo YAML ubicado en el directorio `models`.

Ejemplo:

```yaml
version: 2

sources:
  - name: raw
    database: analytics
    schema: raw

    tables:
      - name: customers
      - name: orders
```

En este ejemplo:

- `raw` es el nombre lógico del Source.
- `analytics` corresponde a la base de datos.
- `raw` corresponde al esquema.
- Se definen las tablas `customers` y `orders`.

---

# Beneficios de utilizar Sources

Utilizar Sources aporta varias ventajas:

- Centralización de la configuración.
- Mayor legibilidad del código.
- Menor dependencia de nombres físicos de tablas.
- Facilita la migración entre entornos.
- Permite implementar monitoreo de frescura de datos.
- Mejora la documentación generada por dbt.

Además, dbt incorpora automáticamente los Sources dentro del DAG del proyecto, permitiendo visualizar las dependencias de forma clara.

---

# Referenciando Sources en Modelos

Una vez definido un Source, puede utilizarse mediante la función `source()`.

Ejemplo:

```sql
SELECT *
FROM {{ source('raw', 'customers') }}
```

Esta práctica evita el uso de nombres físicos directamente en el código SQL y permite que cualquier cambio en la ubicación de la tabla se realice únicamente en el archivo YAML.

---

# Source Freshness

Una de las características más interesantes de los Sources es la posibilidad de verificar la frescura de los datos (*Source Freshness*).

Esto permite responder preguntas como:

- ¿La tabla fuente fue actualizada hoy?
- ¿Existe algún retraso en la carga de datos?
- ¿La información utilizada por los modelos sigue siendo válida?

---

# Configurando Source Freshness

Ejemplo:

```yaml
version: 2

sources:
  - name: raw

    freshness:
      warn_after:
        count: 12
        period: hour

      error_after:
        count: 24
        period: hour

    loaded_at_field: ingestion_timestamp

    tables:
      - name: orders
```

En este caso:

- Se genera una advertencia si los datos tienen más de 12 horas sin actualizarse.
- Se genera un error si superan las 24 horas.
- La columna `ingestion_timestamp` se utiliza para determinar la fecha de carga.

---

# Ejecutando Verificaciones de Frescura

Para ejecutar las validaciones se utiliza:

```bash
dbt source freshness
```

dbt analizará las tablas configuradas y generará un reporte indicando si cumplen con los criterios establecidos.

---

# Sources en Múltiples Entornos

Una de las razones principales para utilizar Sources es facilitar el trabajo con distintos entornos.

Por ejemplo:

- Desarrollo
- QA
- Producción

En lugar de modificar múltiples consultas SQL, basta con actualizar la configuración del Source para apuntar a una nueva base de datos o esquema.

Esto reduce significativamente el esfuerzo de mantenimiento.

---

# Generación de Sources

Cuando existen muchas tablas fuente, crear manualmente el archivo YAML puede resultar tedioso.

El ecosistema de dbt ofrece herramientas como el paquete **dbt-codegen**, que permite generar automáticamente la estructura inicial de los Sources.

Posteriormente, el desarrollador puede complementar dicha configuración con:

- Descripciones.
- Pruebas.
- Reglas de frescura.
- Metadatos adicionales.

---

# ¿Qué son los Seeds?

Los Seeds son archivos CSV almacenados dentro del propio proyecto dbt.

A diferencia de los Sources, cuyos datos existen previamente en el Data Warehouse, los Seeds son cargados por dbt y convertidos en tablas.

Los Seeds son especialmente útiles para almacenar pequeños conjuntos de datos estáticos.

---

# Casos de Uso para Seeds

Algunos ejemplos comunes son:

- Códigos de países.
- Abreviaturas de estados o provincias.
- Tablas de equivalencias.
- Catálogos de clasificación.
- Listas de exclusión o inclusión.

Según los autores, los Seeds deben utilizarse únicamente para información pequeña y relativamente estable.

---

# Cuándo No Utilizar Seeds

Los Seeds no son apropiados para:

- Grandes volúmenes de información.
- Datos que cambian frecuentemente.
- Exportaciones completas de sistemas transaccionales.
- Información sensible o confidencial.

Como regla general, el libro recomienda mantener los Seeds pequeños y fáciles de administrar.

---

# Creando un Seed

Estructura del proyecto:

```text
project/
│
├── models/
│
├── seeds/
│   └── USStates.csv
│
└── dbt_project.yml
```

Contenido del archivo:

```csv
state_abbreviation,state_name
CA,California
TX,Texas
NY,New York
FL,Florida
```

La primera fila corresponde a los encabezados y las filas restantes contienen los datos.

---

# Ejecutando Seeds

Para cargar todos los Seeds:

```bash
dbt seed
```

Para cargar únicamente un Seed específico:

```bash
dbt seed --select USStates
```

dbt leerá el archivo CSV y creará una tabla dentro del Data Warehouse.

---

# Referenciando Seeds

Una vez cargados, los Seeds pueden utilizarse mediante la función `ref()`.

Ejemplo:

```sql
SELECT *
FROM {{ ref('USStates') }}
```

Desde la perspectiva de dbt, los Seeds se comportan de manera muy similar a los modelos.

---

# Resumen

En este capítulo se introdujeron dos componentes fundamentales de dbt:

- **Sources**, que permiten gestionar de forma centralizada las tablas fuente utilizadas por los modelos.
- **Seeds**, que permiten almacenar y versionar pequeños conjuntos de datos estáticos dentro del proyecto.

Los Sources facilitan la mantenibilidad, documentación y monitoreo de los datos de entrada, mientras que los Seeds proporcionan una solución sencilla para administrar tablas de referencia directamente desde el repositorio del proyecto.

En el siguiente capítulo se estudiarán los **Modelos (Models)**, considerados uno de los elementos más importantes dentro de la arquitectura de dbt.
