# Capítulo 2: Modelado de Datos
El modelado de datos es el diseño mediante el cual estructuraremos los datos para guardarse y consultarse efectivamente, antes de escribir cualquier código en dbt, es importante definir como se estructurarán los datos, ya que corregir una mala arquitectura posteriormente requiere un esfuerzo significativo.

## Tipos de Modelos de Datos
Existen diversas formas de modelar los datos para una empresa, cada una con un nivel diferente de **normalización** (reducción de redundancia, más usado para transacciones, enfocado en la operatividad) o **desnormalización** (optimización de lectura y consulta, enfocado en el análisis de datos):

<br>

<img width="926" height="159" alt="imagen" src="https://github.com/user-attachments/assets/00ea7f34-d267-425d-9b31-166c04eeed91" />
<em>Figura: Vista de modelos de datos en una escala de normalización. <br> Recuperado de Unlocking dbt, Second Edition</em>

<br>


<br>

* **Data Vault o DV 2.0:** Enfatiza la escalabilidad, flexibilidad y auditoría. Está altamente normalizado y se compone de *Hubs*, *Satellites* y *Links*. Es excelente para rastrear auditorías, pero su escritura es compleja y su lectura lenta, por lo que no se recomienda para reportes directos.
<img width="802" height="471" alt="imagen" src="https://github.com/user-attachments/assets/309cae69-44a8-47ab-8cc1-db9eef61047b" />
Figura. Modelado con Data Vault 2.0. Nota. Imagen generada por IA mediante Claude (Anthropic)

* **Tercera Forma Normal (3NF / Relacional / Inmon):** Estructura que elimina datos duplicados y dependencias. Es ideal para sistemas transaccionales porque optimiza la escritura rápida, pero las consultas de lectura suelen ser complejas y lentas, destaca que cada tabla tiene exactamente una responsabilidad, por ejemplo la tabla clientes solo sabe de clientes, no encontraremos el nombre de su país, ese dato se hallará en la tabla ciudad. 
<img width="712" height="760" alt="imagen" src="https://github.com/user-attachments/assets/4c6d5d07-b4a7-43b0-806e-a96ffb51651e" />
Figura. Modelado con 3NF Nota. Imagen generada por IA mediante Claude (Anthropic)

* **Dimensional (Esquema Estrella / Kimball):** Diseñado específicamente para optimizar lecturas y generación de reportes. Divide la información en tablas de hechos o *fact tables* (datos cuantitativos), la cual es la tabla que va en el centro de la estrella y de la que salen las tablas de dimensiones o *dimensional tables* (información que describe la fact table, como por ejemplo la descripción de un producto).
<img width="900" height="578" alt="imagen" src="https://github.com/user-attachments/assets/348e9e94-6a2c-41d0-abd8-5b9c7ccc2ff7" />
Figura. Modelado Star Schema Imagen generada por IA mediante Claude (Anthropic)

* **One Big Table (OBT):** Consiste en tener todos los datos necesarios en una sola y enorme tabla. Las consultas son extremadamente rápidas porque no hay "joins", pero tiene una alta tasa de redundancia y es muy difícil de escalar a medida que el negocio crece.

<br>

El modelado dimensional es el más usado en el libro, debido a que dbt se utiliza para procesamiento por lotes y flujos programados, funciona bien con este modelado.

## El Modelo Dimensional: El estándar para dbt
dbt brilla y prospera cuando se utiliza para desarrollar modelos dimensionales[cite: 1]. Este enfoque es considerado el estándar de oro en almacenes de datos (Data Warehouses) por sus múltiples beneficios[cite: 1]:
* **Simplicidad:** Están diseñados para alinearse con la forma en que los usuarios de negocio piensan, facilitando el análisis de autoservicio (self-service analytics)[cite: 1].
* **Recuperación rápida:** La estructura minimiza las uniones (joins), permitiendo que las herramientas de inteligencia de negocios (BI) como Power BI o Tableau consuman los datos a gran velocidad[cite: 1].
* **Flexibilidad:** Permite agregar nuevos atributos o tablas de hechos sin interrumpir las consultas o reportes existentes[cite: 1].

### Esquema Estrella vs. Esquema Copo de Nieve
* **Esquema Estrella (Star Schema):** Tiene una estructura simple donde la tabla de hechos central se une directamente a las tablas de dimensiones, requiriendo un solo join para acceder a los atributos[cite: 1]. **Debe ser siempre tu elección predeterminada**[cite: 1].
* **Esquema Copo de Nieve (Snowflake Schema):** Normaliza las dimensiones dividiéndolas en sub-dimensiones adicionales (tablas de búsqueda o lookups)[cite: 1]. Aunque reduce la redundancia, introduce una complejidad innecesaria que ralentiza las consultas y dificulta el uso para los analistas[cite: 1].

## Componentes del Modelo Dimensional

### 1. Dimensiones (El contexto)
Las dimensiones representan los atributos descriptivos del negocio; nos dicen el quién, qué, dónde y cuándo de los datos[cite: 1]. 
* **Denormalización intencional:** En las dimensiones, la redundancia de datos es esperada y beneficiosa[cite: 1]. Por ejemplo, en una dimensión de cliente, se debe almacenar el país, estado y ciudad en la misma tabla para facilitar el filtrado, en lugar de separar esa jerarquía en múltiples tablas[cite: 1].
* **Claves Sustitutas (Surrogate Keys):** Nunca debes usar las claves primarias del sistema origen (business keys) como claves de dimensión, ya que pueden cambiar y romper el modelo[cite: 1]. En dbt, la mejor práctica actual es utilizar un valor "hash" como identificador único[cite: 1].

### 2. Hechos (Las métricas)
Las tablas de hechos (Facts) son la columna vertebral del modelo analítico y representan los procesos del negocio[cite: 1]. 
* Almacenan los datos cuantitativos y medibles que el negocio necesita analizar, como ingresos, cantidades o recuentos[cite: 1].
* Contienen las claves foráneas (foreign keys) que vinculan los registros numéricos con sus respectivas tablas de dimensiones[cite: 1].

## Dimensiones Lentamente Cambiantes (SCD)
Las dimensiones lentamente cambiantes o SCD (Slowly Changing Dimensions) son técnicas para gestionar cómo se actualiza la información histórica[cite: 1].
* **Tipo 0:** Los valores son inmutables; una vez creado el registro, nunca cambia[cite: 1].
* **Tipo 1:** El valor antiguo se sobrescribe completamente con el nuevo valor, perdiendo todo el historial[cite: 1].
* **Tipo 2:** Se conserva el historial creando una nueva versión del registro cada vez que ocurre un cambio[cite: 1]. Se utilizan campos de fecha de inicio (Date_From) y fecha de fin (Date_To) para saber en qué periodo fue válido ese atributo[cite: 1].

## Pasos para construir el modelo
1. **Recopilación de requisitos del negocio:** Escuchar a los interesados para entender sus problemas y métricas clave sin imponer limitaciones técnicas iniciales[cite: 1].
2. **Matriz de Bus (Bus Matrix):** Un plano visual (generalmente una hoja de cálculo) que cruza los procesos de negocio (filas) con las dimensiones (columnas) para definir cómo interactuarán[cite: 1].
3. **Determinar el grano (Grain):** Definir el nivel más bajo de detalle requerido para cada tabla (por ejemplo, "una fila por ítem de cada pedido")[cite: 1].
4. **Mapeo de Origen a Destino (STTM):** Un documento crucial que mapea exactamente qué campos del sistema origen alimentarán a las tablas destino en el Data Warehouse, definiendo tipos de datos, lógicas y reglas de nulos[cite: 1].
5. **Validación y Desarrollo:** Revisar el diseño con ingenieros para detectar riesgos antes de escribir la lógica de transformación[cite: 1].
