# 📦 My First dbt Project

Proyecto de aprendizaje siguiendo el libro **Unlocking dbt**, con implementación práctica usando dbt Core + Snowflake. Se usa **uv** como gestor moderno de entornos y paquetes en lugar del pip clásico.

<img width="827" height="1180" alt="61slJz1ECFL _SL1180_" src="https://github.com/user-attachments/assets/30ce04e2-75b3-4845-8e72-76ce2616cbd6" />

---

## 🧰 Stack

| Herramienta | Versión | Propósito |
|---|---|---|
| Python | 3.12+ | Lenguaje base |
| dbt Core | latest | Transformación de datos |
| dbt-snowflake | latest | Adaptador para Snowflake |
| uv | latest | Gestor de entornos y paquetes |
| Snowflake | — | Data warehouse |

---

## ⚡ Setup rápido

### 1. Prerrequisitos

- Tener [uv instalado](https://docs.astral.sh/uv/getting-started/installation/)
- Tener acceso a una cuenta de Snowflake
- Tener git instalado

### 2. Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/my_first_dbt_project.git
cd my_first_dbt_project
```

### 3. Crear y activar el entorno virtual

```bash
uv venv dbt-local-dev-env
source dbt-local-dev-env/bin/activate   # Mac/Linux
# dbt-local-dev-env\Scripts\activate   # Windows
```

### 4. Instalar dependencias

```bash
uv pip install dbt-snowflake
```

### 5. Verificar instalación

```bash
dbt --version
```

### 6. Al terminar de trabajar

```bash
deactivate
```

---

## 📁 Estructura del proyecto

```bash
unlocking_dbt_book/
├── README.md
│
├── docs/
│   ├── cap01-introduccion.md
│   ├── cap02-instalacion.md
│   ├── decisiones-tecnicas.md
│   └── ...
│
└── my_first_dbt_project/
    ├── analyses/
    ├── macros/
    ├── models/
    ├── seeds/
    ├── snapshots/
    ├── tests/
    ├── dbt_project.yml
    ├── profiles.yml
    ├── README.md
    └── .gitignore
```

---

## 📚 Documentación

Cada capítulo del libro tiene sus notas en la carpeta `docs/`:

- [Capítulo 1 — Introducción a dbt](docs/cap01-introduccion.md)
- [Capítulo 2 — Instalación y configuración](docs/cap02-instalacion.md)
- [Decisiones técnicas](docs/decisiones-tecnicas.md) ← por qué elegí uv sobre pip, etc.

---

## 📖 Recursos

- 📘 Libro: *Unlocking dbt*
- 📄 Documentación oficial dbt: [docs.getdbt.com](https://docs.getdbt.com)
- ⚡ Documentación uv: [docs.astral.sh/uv](https://docs.astral.sh/uv)
- ❄️ Snowflake: [snowflake.com](https://www.snowflake.com)
