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

> Debemos hacernos unas preguntas antes de decidir crear un warehouse, podemos considerarlo si nuestras respuestas dan 'si' en cada una de las siguientes preguntas:

1. Is it important to have a single source of truth for your data?
2. Are multiple areas of the business going to be consuming this data?
3. Do we need to analyze data from different sources?
4. Do we need to maintain historical tracking of data and make sure we are looking at measurables from a point in time?
5. Do you have a high volume of data that needs to be transformed?
6. Do you find yourself running a lot of complex queries regularly?


*Fuente: Unlocking dbt — Capítulo 1*

## Proceso Git - Git Hub para el proyecto
```bash
1. Conectar tu carpeta local con GitHub
  git remote add origin https://github.com/tu-usuario/my_first_dbt_project.git
2. Agregar todos los archivos al staging
  git add .
3. Realizamos el primer commit
  git commit -m "feat: initial project structure with dbt + uv setup"
4. Subimos a Git Hub
  git push -u origin main
```
## Proceso para generar el token de Git Hub

- Ve a github.com → tu foto de perfil → Settings
- En el menú izquierdo, baja hasta Developer settings (al fondo)
- Personal access tokens → Tokens (classic)
- Generate new token → Generate new token (classic)
- En Note ponle un nombre: mi-laptop
- En Expiration elige 90 días o "No expiration"
- En permisos marca solo repo (el primero)
- Clic en Generate token
- Copia el token inmediatamente — solo se muestra una vez
