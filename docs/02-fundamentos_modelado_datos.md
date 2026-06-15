El modelado de datos es el diseño mediante el cual estructuraremos los datos para guardarse y consultarse efectivamente, antes de escribir cualquier código en dbt, es importante definir como se estructurarán los datos, ya que corregir una mala arquitectura posteriormente requiere un esfuerzo significativo.
## Tipos de Modelos de Datos
Existen diversas formas de modelar los datos para una empresa, cada una con un nivel diferente de **normalización** (reducción de redundancia, más usado para transacciones, enfocado en la operatividad) o **desnormalización** (optimización de lectura y consulta, enfocado en el análisis de datos):

<br>
<img width="926" height="159" alt="imagen" src="https://github.com/user-attachments/assets/00ea7f34-d267-425d-9b31-166c04eeed91" />
<br>
<em>Figura: Vista de modelos de datos en una escala de normalización.</em>
<br>
<br>

* **Data Vault o DV 2.0:** Enfatiza la escalabilidad, flexibilidad y auditoría. Está altamente normalizado y se compone de *Hubs*, *Satellites* y *Links*. Es excelente para rastrear auditorías, pero su escritura es compleja y su lectura lenta, por lo que no se recomienda para reportes directos.

<img width="802" height="471" alt="imagen" src="https://github.com/user-attachments/assets/309cae69-44a8-47ab-8cc1-db9eef61047b" />
<br>
<em>Figura. Modelado con Data Vault 2.0. Nota. Imagen generada por IA mediante Claude (Anthropic)</em>
<br>
<br>

* **Tercera Forma Normal (3NF / Relacional / Inmon):** Estructura que elimina datos duplicados y dependencias. Es ideal para sistemas transaccionales porque optimiza la escritura rápida, pero las consultas de lectura suelen ser complejas y lentas, destaca que cada tabla tiene exactamente una responsabilidad, por ejemplo la tabla clientes solo sabe de clientes, no encontraremos el nombre de su país, ese dato se hallará en la tabla ciudad.

<img width="712" height="760" alt="imagen" src="https://github.com/user-attachments/assets/4c6d5d07-b4a7-43b0-806e-a96ffb51651e" />
<br>
<em>Figura. Modelado con 3NF. Nota. Imagen generada por IA mediante Claude (Anthropic)</em>
<br>
<br>

* **Dimensional (Esquema Estrella / Kimball / Star schema):** Diseñado específicamente para optimizar lecturas y generación de reportes. Divide la información en tablas de hechos o *fact tables* (datos cuantitativos), la cual es la tabla que va en el centro de la estrella y de la que salen las tablas de dimensiones o *dimensional tables* (información que describe la fact table, como por ejemplo la descripción de un producto).

<img width="900" height="578" alt="imagen" src="https://github.com/user-attachments/assets/348e9e94-6a2c-41d0-abd8-5b9c7ccc2ff7" />
<br>
<em>Figura. Modelado Star Schema. Imagen generada por IA mediante Claude (Anthropic)</em>
<br>
<br>

* **One Big Table (OBT):** Consiste en tener todos los datos necesarios en una sola y enorme tabla. Las consultas son extremadamente rápidas porque no hay "joins", pero tiene una alta tasa de redundancia y es muy difícil de escalar a medida que el negocio crece.

### Notas
* El modelado dimensional es el estándar recomendado en la industria cuando se trabaja con dbt, debido al rendimiento que tiene con las características de dbt, también se menciona que el modelado se debe ajustar al negocio y cualquiera puede ser programado en la herramienta.
* Por ejemplo, una empresa de servicios financieros altamente regulada podría beneficiarse de un modelo Data Vault debido a su capacidad de auditoría y trazabilidad histórica. Una empresa de comercio electrónico que necesita construir una tienda operativa central podría preferir un enfoque 3NF para organizar los datos transaccionales. Un equipo de análisis minorista que crea paneles para ejecutivos probablemente se beneficiaría más de un modelo dimensional. Y una startup que se mueve rápido y opera de forma ajustada puede optar por una estructura One Big Table para acelerar la entrega sin la sobrecarga inicial del modelado.
* Es válido también el enfoque de utilizar cada modelado en una capa distinta de dbt por ejemplo, la capa de staging/raw es transaccional, la capa intermedia es un data vault y la capa final es un modelo dimensional.

## El Modelo Dimensional: El estándar para dbt
Dbt brilla y prospera cuando se utiliza para desarrollar modelos dimensionales, beneficios:
* **Simplicidad:** Están diseñados para alinearse con la forma en que los usuarios de negocio piensan.
* **Recuperación rápida:** La estructura minimiza las uniones (joins), permitiendo que las herramientas de inteligencia de negocios (BI) como Power BI o Tableau consuman los datos a gran velocidad.
* **Flexibilidad:** Permite agregar nuevos atributos o tablas de hechos sin interrumpir las consultas o reportes existentes.

### Esquema Estrella vs. Esquema Copo de Nieve
* **Esquema Estrella (Star Schema):** Tiene una estructura simple donde la tabla de hechos central se une directamente a las tablas de dimensiones, requiriendo un solo join para acceder a los atributos.
* **Esquema Copo de Nieve (Snowflake Schema):** Normaliza las dimensiones dividiéndolas en sub-dimensiones adicionales (tablas de búsqueda o lookups). Aunque reduce la redundancia, introduce una complejidad innecesaria que ralentiza las consultas y dificulta el uso para los analistas al tener que hacer más uniones para tener las tablas completas para consulta.

<img width="1100" height="930" alt="imagen" src="https://github.com/user-attachments/assets/df5d7e11-042c-44c6-92c4-25718042bf75" />
<br>
<em>Figura. Modelado Snowflake Schema.</em>
<br>
<br>

### Notas
* Utilizar el modelado de copo de nieve puede ser tentador para los que modelan puesto que facilita la mantenibilidad al normalizar los datos, sin embargo esto afecta a los usuarios como analistas de datos que ahora se les ha desplazado la complejidad al tener que hacer varias uniones en sus tablas.
* Utilizando el modelado de estrella, solo es necesario realizar la lógica compleja una vez en la transformación (la t del ETL - Extracción, Transformación y Load-Cargado de datos), así se les facilita el trabajo a los consumidores de los datos, lo que apoya a que los informes, paneles se construyan de manera más eficiente.
* Algunos casos puntuales pueden requerir una normalización de dimensiones muy grandes, pero debería ser excepcional, en casi todos los casos el esquema de estrella es el más óptimo para modelos dimensionales escalables, de alto rendimiento y fáciles de usar.

## Construcción de un modelo dimensional
Necesitamos construir un warehouse (almacén de datos) para la escalabilidad, el rendimiento y la facilidad de uso.

### 1. **Recopilación de requisitos del negocio**
Qué necesita medir y analizar la empresa, es el paso más crucial, un modelo bien diseñado debe reflejar lo que es relevante para la empresa, se debe comprender primero los objetivos, desafíos y métricas clave del negocio, la mejor manera de lograrlo es mediante entrevistas.
* Al interactuar con las partes interesadas, es crucial escuchar sin imponer limitaciones técnicas, recopilar información de forma abierta, animar a describir los flujos de trabajo, puntos conflictivos, resultados ideales.

Este es un ciclo interminable que siempre está en curso, las cosas cambian a menudo, asegurarse de comunicarse frecuentemente para comprender los cambios.

```bash
Preguntas base de ejemplo, se recomienda adaptarlas al negocio en cuestión.
* ¿Qué datos necesita para poder alcanzar sus metas y objetivos empresariales?
* ¿Qué preguntas basadas en datos son más importantes para su equipo?
* ¿Qué métricas clave o  KPI está buscando?
```


<img width="1200" height="423" alt="imagen" src="https://github.com/user-attachments/assets/cb1854df-5af2-47af-bb1c-fc04666bcae4" />
<br>
<em>Figura. Data Model - BUS Matrix .</em>
<br>
<br>



### 2. **Determinar la prioridad del mart(área funcional que necesita los datos) y por dónde empezar**
3. **Traducir los requisitos del negocio a requisitos técnicos**
4. **Determinar los hechos y dimensiones:** Identificar los datos clave medibles y los atributos descriptivos en el modelo.
5. **Validar el diseño:** Asegurar que el modelo se alinee con la lógica empresarial y los requisitos de informe.
6. **Crear un documento de mapeo de origen a destino (Source to Target Mapping:** Documentar como los datos crudos de origen se transformarán en el modelo dimensional.
7. **Construcción del diagrama Entidad-Relación(ER):** Mapeo visual de las tablas y sus relaciones.
8. **Desarrollo:** Implementación del modelo, asegurando rendimiento y escalabilidad.
9. **Mantenimiento y desarrollo continuos:** Adaptación y optimización del modelo a medida que evolucionan las necesidades del negocio.

## Dimensiones Lentamente Cambiantes (SCD)
Las dimensiones lentamente cambiantes o SCD (Slowly Changing Dimensions) son técnicas para gestionar cómo se actualiza la información histórica.
* **Tipo 0:** Los valores son inmutables; una vez creado el registro, nunca cambia.
* **Tipo 1:** El valor antiguo se sobrescribe completamente con el nuevo valor, perdiendo todo el historial.
* **Tipo 2:** Se conserva el historial creando una nueva versión del registro cada vez que ocurre un cambio. Se utilizan campos de fecha de inicio (Date_From) y fecha de fin (Date_To) para saber en qué periodo fue válido ese atributo.
