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

## Notas

> Debemos hacernos unas preguntas antes de decidir crear un warehouse, podemos considerarlo si nuestras respuestas dan 'si' en cada una de las siguientes preguntas:

1. ¿Es importante tener una única fuente de verdad para sus datos?
2. ¿Van a consumir estos datos múltiples áreas del negocio?
3. ¿Necesitamos analizar datos procedentes de diferentes fuentes?
4. ¿Necesitamos mantener un historial de seguimiento de los datos y asegurarnos de que estamos observando mediciones desde un punto específico en el tiempo?
5. ¿Tiene un gran volumen de datos que necesita ser transformado?
6. ¿Se encuentra ejecutando con frecuencia muchas consultas complejas?


*Fuente: Unlocking dbt — Capítulo 1*

## Flujo Git - Git Hub inicio
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

## Flujo Git - Git Hub para la documentación del proyecto
```bash
1. Editamos la carpeta docs/ desde git hub, al empezar descargamos los archivos a local
git pull --rebase origin main
2. Trabajamos el proyecto localmente, para terminar hacemos
git add .
git commit -m "..."
git push origin main
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
