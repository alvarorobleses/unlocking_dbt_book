# Capítulo 1: Introducción a dbt

## ¿Qué es dbt?

dbt (data build tool) es una herramienta de transformación de datos que permite construir pipelines ELT utilizando SQL y buenas prácticas de ingeniería de software.
A diferencia de las herramientas ETL tradicionales, dbt no extrae ni carga datos. Su función principal es transformar datos que ya se encuentran almacenados en un Data Warehouse
El flujo básico es:
```
Fuente de datos → Data Warehouse (raw) → dbt transforma → tablas/vistas limpias
```

## Conceptos clave aprendidos

- **dbt Core**: versión open source, se instala localmente y se usa desde la terminal.
- **dbt Cloud**: versión SaaS con interfaz web, CI/CD y más funcionalidades.
- **Adaptador**: cómo dbt se conecta al data warehouse (Snowflake, BigQuery, Redshift, etc.).
- **Modelo**: un archivo `.sql` que define una transformación. dbt lo convierte en una tabla o vista.
- **Motor Fusion:** A través de la adquisición de SDF Labs, dbt ha introducido "Fusion", un nuevo motor de ejecución escrito en Rust que reemplaza al motor original en Python. Fusion mejora drásticamente la velocidad de compilación, ofrece validación de SQL en tiempo real y potencia funciones inteligentes de autocompletado en el IDE.

## ¿Por qué dbt?

- Permite usar SQL para transformar datos de forma estructurada.
- Genera documentación automática.
- Permite pruebas de calidad de datos.
- Fomenta buenas prácticas de ingeniería (versionado, modularidad, testing).

## ¿Cuándo tiene sentido implementar un Data Warehouse?

> Si la respuesta es sí para la mayoría de ellas, probablemente un Data Warehouse aporte valor al negocio.

- ¿Es importante tener una única fuente de verdad para los datos?
- ¿Los datos serán consumidos por múltiples áreas del negocio?
- ¿Necesitamos integrar información procedente de diferentes sistemas?
- ¿Necesitamos mantener histórico y trazabilidad de los datos?
- ¿Existe un gran volumen de datos que requiere transformaciones frecuentes?
- ¿Se ejecutan habitualmente consultas complejas o analíticas?

## Estructura de un Proyecto de dbt
Cuando inicializas un proyecto, dbt crea carpetas predeterminadas con propósitos muy específicos:
* **models/:** El corazón del proyecto. Contiene los archivos `.sql` con la lógica de la transformación.
* **seeds/:** Almacena archivos CSV estáticos y pequeños (usualmente mapeos o catálogos como códigos de países) que dbt carga directamente a tu base de datos.
* **snapshots/:** Crea un registro histórico de cómo cambian los datos a lo largo del tiempo.
* **tests/:** Archivos con consultas SQL que verifican la precisión e integridad de los datos en tus modelos.
* **macros/:** Contiene bloques de código Jinja reutilizables.
* **dbt_packages/:** Donde se descargan paquetes de dependencias creados por la comunidad.
* **target/:** Directorio autogenerado que almacena metadatos compilados (como `manifest.json`) y el SQL crudo que dbt enviará a tu almacén de datos.
* **analyses/ y logs/:** Analíticas puntuales que no se materializan, y registros de depuración, respectivamente.

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

⚠️ GitHub solo muestra el token una única vez.

