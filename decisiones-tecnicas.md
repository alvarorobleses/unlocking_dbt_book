# Decisiones técnicas

Este archivo documenta las decisiones importantes tomadas en el proyecto y el razonamiento detrás de cada una. Esto se conoce como **ADR (Architecture Decision Record)** y es una práctica común en equipos de ingeniería.

---

## ADR-001: uv en lugar de pip + virtualenv

**Fecha:** 2026  
**Estado:** Adoptado

### Contexto

El libro *Unlocking dbt* propone usar `pip` + `virtualenv` para gestionar el entorno de Python. Es el método más documentado y ampliamente conocido.

### Decisión

Usar **uv** como gestor de entornos y paquetes en lugar de pip + virtualenv.

### Razones

- **Velocidad**: uv está escrito en Rust y es entre 10x y 100x más rápido que pip.
- **Todo en uno**: reemplaza pip, virtualenv, pip-tools y más con un solo binario.
- **Moderno**: es el estándar emergente en la comunidad Python en 2025-2026.
- **Compatible**: `uv pip install` es compatible con cualquier paquete de PyPI, incluyendo dbt.

### Consecuencias

- El flujo de instalación difiere ligeramente del libro, pero es equivalente.
- No se necesita el paso de actualizar `wheel` y `setuptools` porque uv lo maneja internamente.
- Cualquier comando `pip install X` del libro se traduce directamente a `uv pip install X`.

### Alternativas consideradas

| Opción | Por qué no |
|---|---|
| pip + virtualenv | Funciona, pero es más lento y requiere más pasos |
| Poetry | Más complejo para este proyecto de aprendizaje |
| Conda | Innecesario para un proyecto solo de Python/dbt |

---

## ADR-002: Snowflake como data warehouse

**Fecha:** 2026  
**Estado:** Adoptado

### Contexto

dbt soporta múltiples data warehouses. El libro también usa Snowflake como ejemplo principal.

### Decisión

Usar **Snowflake** como data warehouse siguiendo el libro.

### Razones

- Es el adaptador que el libro usa y documenta en detalle.
- Snowflake ofrece una capa gratuita (trial) para aprendizaje.
- Es uno de los data warehouses más usados en la industria con dbt.

---

*Este archivo se irá actualizando conforme avance el proyecto.*
