# Capítulo 2 — Instalación y configuración

## Lo que propone el libro

El libro instala dbt usando **pip + virtualenv**, que es el método clásico:

```bash
pip install virtualenv --upgrade
python3 -m venv dbt-local-dev-env
source dbt-local-dev-env/bin/activate
pip install pip wheel setuptools --upgrade
pip install dbt-snowflake
```

## Mi enfoque: usando uv en lugar de pip

Decidí usar **uv** porque es más rápido, moderno y reemplaza varias herramientas a la vez.
Ver [decisiones-tecnicas.md](decisiones-tecnicas.md) para más contexto.

### Equivalencia pip → uv

| Paso | Con pip (libro) | Con uv (mi enfoque) |
|---|---|---|
| Crear entorno virtual | `python3 -m venv dbt-local-dev-env` | `uv venv dbt-local-dev-env` |
| Activar entorno | `source dbt-local-dev-env/bin/activate` | `source dbt-local-dev-env/bin/activate` |
| Actualizar setuptools | `pip install pip wheel setuptools --upgrade` | No necesario con uv ✓ |
| Instalar dbt | `pip install dbt-snowflake` | `uv pip install dbt-snowflake` |
| Verificar instalación | `pip show dbt-snowflake` | `uv pip show dbt-snowflake` |
| Desactivar entorno | `deactivate` | `deactivate` |

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
- `dbt-bigquery` — Google BigQuery
- `dbt-redshift` — Amazon Redshift
- `dbt-postgres` — PostgreSQL

Lista completa: [docs.getdbt.com/docs/supported-data-platforms](https://docs.getdbt.com/docs/supported-data-platforms)

## Notas personales

> Aquí puedes agregar tus propias reflexiones o dudas del capítulo.

---

*Fuente: Unlocking dbt — Capítulo 2*
