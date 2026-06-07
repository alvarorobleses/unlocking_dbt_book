# Capítulo 1 — Introducción a dbt

## ¿Qué es dbt?

dbt (data build tool) es una herramienta de transformación de datos que permite a los analistas y data engineers transformar datos en su data warehouse usando SQL.

El flujo básico es:
```
Fuente de datos → Data Warehouse (raw) → dbt transforma → tablas/vistas limpias
```

## Conceptos clave aprendidos

- **dbt Core**: versión open source, se instala localmente y se usa desde la terminal.
- **dbt Cloud**: versión SaaS con interfaz web, CI/CD y más funcionalidades.
- **Adaptador**: cómo dbt se conecta al data warehouse (Snowflake, BigQuery, Redshift, etc.).
- **Modelo**: un archivo `.sql` que define una transformación. dbt lo convierte en una tabla o vista.

## ¿Por qué dbt?

- Permite usar SQL para transformar datos de forma estructurada.
- Genera documentación automática.
- Permite pruebas de calidad de datos.
- Fomenta buenas prácticas de ingeniería (versionado, modularidad, testing).

## Notas personales

> Aquí puedes agregar tus propias reflexiones o dudas del capítulo.

---

*Fuente: Unlocking dbt — Capítulo 1*
