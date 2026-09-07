MÓDULO DE POWER BI
TRABAJO FINAL: IMPLEMENTACIÓN Y ANÁLISIS
Optimización, Limpieza de Datos y Modelado de Negocio para 'Global Electronics Retail'
Estudiante: Maria Elena
Curso: Análisis y Visualización de Datos
Entregable: Trabajo_Final_Global_Electronic_Maria_Elena.docx / Apellido_Nombre_TrabajoPBI.pbix
____________________________________________________________________________
1. INTRODUCCIÓN Y DATOS ORIGEN
1.1. Contexto del Caso de Negocio
El proyecto se centra en el análisis del modelo de negocio de una cadena minorista global ficticia de comercio electrónico y tiendas físicas de tecnología llamada "Global Electronics Retail". La compañía opera a nivel mundial vendiendo dispositivos móviles, ordenadores, sistemas de audio, accesorios y televisores. El reto estratégico del negocio radica en consolidar y analizar información de transacciones realizadas en múltiples países, bajo divisas locales diferentes (EUR, GBP, USD, etc.) y distribuidas a través de dos canales principales: tiendas físicas y un canal online unificado. Asimismo, la cadena logística enfrenta el desafío de monitorizar los tiempos de entrega para optimizar la satisfacción del cliente.
1.2. Ficheros y Dataset Seleccionado
Se ha seleccionado un dataset estructurado con fines de Business Intelligence que simula este entorno transaccional. Compuesto por 5 tablas principales y un histórico transaccional de ventas. Proveniente de un repositorio público y educativo de Maven Analytics / Microsoft. El dataset original contiene una estructura relacional lista para ser integrada en un modelo estrella, con un total de 26 mil pedidos en la tabla de hechos y múltiples campos descriptivos en sus dimensiones.
1.3. Justificación de la Elección
Elegí este dataset porque simula un negocio real con desafíos cotidianos y prácticos, ideales para aplicar y reforzar todo lo aprendido en el curso. En concreto, nos permite dar respuesta a tres problemas clave que enfrenta cualquier empresa internacional:
•	Ventas en diferentes monedas: Como la tienda vende en varios países usando monedas distintas (como euros, libras o dólares), el modelo calcula automáticamente el valor de cada venta en dólares usando el tipo de cambio exacto del día. Esto nos permite sumar y comparar los ingresos totales de manera correcta y sin mezclar monedas.
•	Tiendas físicas frente a internet: El modelo ayuda a comparar de forma muy clara cómo funcionan las ventas en los locales físico frente al rendimiento del canal digital de internet.
•	Control de los envíos a domicilio: Al registrar el día en que se hace el pedido y el día en que se entrega, podemos calcular con total precisión cuántos días tarda en llegar la compra al cliente y detectar en qué ciudades o países estamos teniendo retrasos con el reparto.
2. TRANSFORMACIONES PRINCIPALES EN POWER QUERY (ETL)
Siguiendo las mejores prácticas de la arquitectura de datos ("calcular en el origen siempre que sea posible para favorecer la compresión"), se aplicaron transformaciones en el editor de Power Query (Lenguaje M) para limpiar, estandarizar y enriquecer el modelo antes de realizar cualquier análisis. A continuación se desglosan las transformaciones por cada tabla:
2.1. Tabla Store (Tiendas)
•	Asignación de Tipos de Datos: Se configuró el campo StoreKey como número entero (clave), Country y State como texto, y SquareMeters como número entero. Se corrigió y asignó el formato de fecha al campo OpenDate (única fecha de esta tabla).
•	Gestión de Valores Nulos: El canal de ventas online está representado por una tienda ficticia que originalmente poseía un valor nulo (null) en SquareMeters. Para estandarizar y evitar que afecte a posibles cálculos matemáticos o promedios sobre el tamaño físico de los locales, se sustituyeron los valores null por un valor de 0.
•	Creación de Nueva Columna 'TipoTienda': Se agregó una columna condicional basada en SquareMeters con la siguiente lógica: Si SquareMeters es igual a 0 el valor asignado es "Online"; de lo contrario, se asigna "Tienda Física". 
2.2. Tabla Products (Productos)
•	Asignación de Tipos de Datos: Los campos cruciales de Unit Cost y Unit Price se configuraron como "Número decimal fijo" (moneda) para garantizar la precisión matemática y evitar errores de redondeo en cálculos de ventas y márgenes. Los campos ProductKey, SubcategoryKey y CategoryKey se configuraron como números enteros.
•	Estandarización de Nombres: Se aplicó la función de limpieza "Recortar" (Trim) en las columnas Brand, Category y Subcategory para remover cualquier espacio en blanco al inicio o al final del texto.
2.3. Tabla Customers (Clientes)
•	Tipos de Datos y Limpieza: Se configuró el campo Birthday en formato Fecha y CustomerKey como número entero. Se aplicó la función "Recortar" (Trim) a las columnas de texto descriptivo Name, City, Country y Continent para unificar el texto de la dimensión.
•	Preparación Geográfica: Se validó que los campos geográficos estuvieran bien estructurados para habilitar su categorización en Power BI y permitir de este modo la graficación en mapas y el desarrollo de jerarquías regionales.
2.4. Tabla Sales (Ventas - Tabla de Hechos)
•	Tipo de Campos de Relación: Se configuró la columna de cantidad (Quantity) y todas las llaves de relación (Order Number, CustomerKey, ProductKey, StoreKey) en los formatos correctos de números enteros para asegurar una relación limpia sin errores de incompatibilidad.
•	Tratamiento de Nulos y Estado de Entrega: Se identificó la presencia de valores nulos (null) en la columna Delivery Date, que representan pedidos que aún están pendientes de entrega. Se creó la columna condicional "Estado de Entrega" con la lógica: si Delivery Date es nulo, entonces "Pendiente"; de lo contrario, "Entregado". Esto permite que actúe como un segmentador de datos en todas las páginas para monitorear el estado logístico.
•	Creacion de columnas 'Días Entrega': Para calcular el tiempo de entrega de los pedidos, se creó una columna personalizada llamada Días Entrega en Power Query utilizando la siguiente fórmula 
Días Entrega = if [Delivery Date] = null then null else Duration.Days([Delivery Date] - [Order Date])
Al mantener el valor vacío como null en lugar de rellenarlo artificialmente con un 0, garantizamos que la medida promedio en DAX ignore estos registros y no sesgue el KPI promedio de tiempos de entrega.



3. MODELO DE DATOS Y RELACIONES (ESQUEMA EN ESTRELLA)
El modelo de datos implementado sigue el estándar de diseño de Esquema en Estrella (Star Schema), donde una única tabla de hechos central es rodeada y filtrada directamente por las tablas de dimensiones. Este modelo optimiza el flujo de filtrado unidireccional y previene problemas de filtros cruzados circulares.
3.1. Tablas del Modelo
•	Tabla de Hechos - Sales: Contiene cada fila transaccional del histórico de ventas unificado con la cantidad y las claves de relación.
•	Tabla de Dimensión - Products: Información comercial de los artículos, categorías, subcategorías, colores, marcas, costos y precios.
•	Tabla de Dimensión - Customers: Datos demográficos, género, edad y ubicación geográfica de los clientes.
•	Tabla de Dimensión - Stores: Permite la segmentación física por tamaño en metros cuadrados y clasifica los canales Online y Tienda Física.
•	Tabla de Dimensión - Calendar: Tabla generada mediante DAX para el análisis temporal correcto, marcada obligatoriamente como Tabla de Fechas.
•	Tabla de Soporte - Exchange_Rates: Contiene las cotizaciones de divisas por fecha para normalizar de forma transaccional todos los importes a dólares americanos (USD).
3.2. Relaciones del Modelo
Se configuraron relaciones de tipo Uno a Varios (1:*) con dirección de filtro único (Single) desde las dimensiones hacia la tabla de hechos Sales:
• Customers[CustomerKey] (1) ---> Sales[CustomerKey] (*)
• Products[ProductKey] (1) ---> Sales[ProductKey] (*)
• Stores[StoreKey] (1) ---> Sales[StoreKey] (*)
• Calendar[Date] (1) ---> Sales[Order Date] (*)
3.3. Higiene del Modelo: Campos Ocultos
Para mantener un panel de campos ordenado e intuitivo para el usuario del informe y cumplir con las pautas de Higiene del Modelo, se ocultaron de la vista de reporte todas las llaves técnicas y columnas de relación:
• En Customers: CustomerKey, Birthday, StateCode, ZipCode.
• En Products: ProductKey, SubcategoryKey y CategoryKey.
• En Stores: StoreKey.
• En Sales: CustomerKey, ProductKey, StoreKey y Tipo Cambio (columna de soporte interna).
3.4. Jerarquías Técnicas Creadas para Drill-down
Para enriquecer la interactividad y permitir que los usuarios profundicen en el análisis jerárquico directamente desde los gráficos, se crearon tres jerarquías fundamentales:
1. Jerarquía Geográfica de Clientes: Continente > País > Estado > Ciudad (diseñada para el análisis demográfico).
2. Jerarquía Geográfica de Tiendas: Country > State.
3. Jerarquía de Productos: Category > Subcategory > ProductName (permite realizar drill-down para analizar desde categorías generales como Computadoras hasta modelos específicos de producto).
  

4. CÁLCULOS DAX (MEDIDAS Y COLUMNAS CALCULADAS)
4.1. Columnas Agregadas en PowerQuery
•	Products[RangoPrecio]: Clasifica los productos según su precio unitario en tres niveles:
 
RangoPrecio = SWITCH( TRUE(), 'Products'[UnitPriceUSD] < 100, "Económico", 'Products'[UnitPriceUSD] <= 500, "Estándar", "Premium")
•	Customers[EdadCliente]: Obtiene la edad exacta en años a partir de la fecha de nacimiento: 
DATEDIFF('Customers'[Birthday], TODAY(), YEAR).
•	Customers[RangoEdad]: Segmenta a los clientes para el análisis demográfico del reporte:
RangoEdad = SWITCH( TRUE(), 'Customers'[EdadCliente] <= 25, "18-25", 'Customers'[EdadCliente] <= 40, "26-40", 'Customers'[EdadCliente] <= 60, "41-60", "Más de 60" )
•	Sales[Tipo Cambio]: Calcula el tipo de cambio uniendo Sales con la tabla soporte Exchange_Rates por moneda y fecha sin crear relaciones complejas:
Tipo Cambio = LOOKUPVALUE(Exchange_Rates[Exchange], Exchange_Rates[Currency], Sales[Currency Code], Exchange_Rates[Date], Sales[Order Date])
4.2. Columnas Calculadas DAX
•	Sales[Importe Venta] e [Importe Coste]: Efectúa la conversión de la divisa de origen multiplicando las unidades vendidas por el precio/costo y aplicando el tipo de cambio:  
Importe Venta = Sales[Quantity] * RELATED(Products[UnitPriceUSD]) * Sales[Tipo Cambio]
   Importe Coste = Sales[Quantity] * RELATED(Products[UnitCostUSD]) * Sales[Tipo Cambio]

4.3. Creación de la Tabla de Calendario
Se implementó una tabla de calendario propia mediante Lenguaje M que abarca el rango histórico completo de la tienda, marcada como Tabla de Fechas obligatoria para el uso correcto de las fórmulas de inteligencia de tiempo. Se ordeno la columna Nombre Mes por la columna numérica Mes para evitar que los gráficos se ordenen alfabéticamente.
   let
    FechaInicio = List.Min(Sales[Order Date]),
    FechaFin = List.Max(Sales[Order Date]),
    Dias = Duration.Days(FechaFin - FechaInicio) + 1,
    ListaFechas = List.Dates(FechaInicio, Dias, #duration(1, 0, 0, 0)),
    Tabla = Table.FromList(ListaFechas, Splitter.SplitByNothing(), {"Fecha"}),
    TipoFecha = Table.TransformColumnTypes(Tabla, {{"Fecha", type date}}),
    Anio = Table.AddColumn(TipoFecha, "Año", each Date.Year([Fecha]), Int64.Type),
    Trimestre = Table.AddColumn(Anio, "Trimestre", each "Q" & Text.From(Date.QuarterOfYear([Fecha])), type text),
    Mes = Table.AddColumn(Trimestre, "Mes", each Date.Month([Fecha]), Int64.Type),
    NombreMes = Table.AddColumn(Mes, "Nombre Mes", each Date.ToText([Fecha], "MMMM", "es-ES"), type text),
    Semana = Table.AddColumn(NombreMes, "Semana", each Date.WeekOfYear([Fecha]), Int64.Type),
    DiaSemana = Table.AddColumn(Semana, "Día Semana", each Date.DayOfWeek([Fecha], Day.Monday) + 1, Int64.Type)
in
    DiaSemana
4.3. Medidas DAX
Las medidas DAX fueron organizadas en carpetas lógicas para una navegación cómoda e intuitiva:
Carpeta: Costes
•	Costes Totales: Consolida el coste de los productos.
Costes Totales = SUM(Sales[Importe Coste])
•	Costes Medio: Calcula el coste medio por pedido.
Coste Medio = DIVIDE([Costes Totales], [Total de Pedidos], 0)
•	Costes Año Anterior: Calcula los Costes del Año Anterior. Obtiene la cifra de los costes del mismo periodo, pero del año pasado.
Costes Año Anterior = CALCULATE(
    [Costes Totales],
    SAMEPERIODLASTYEAR('Calendar'[Fecha]) )
•	Costes Variación YoY : Variación vs año anterior (Valor monetario). Calcula el crecimiento o decrecimiento de los costes comparando el periodo actual con el año.
Costes Variación YoY = [Costes Totales] - [Costes Año Anterior]
•	Costes Variación YoY %: Mide la variación relativa de costos. 
Costes Variación YoY % = DIVIDE([Costes Variación YoY], [Costes Año Anterior], 0)
•	Costes YTD: (Acumulado Anual) Permite mostrar cuánto dinero ha gastado la empresa en la compra de productos desde el 1 de enero hasta el día seleccionado en el filtro.
Costes YTD = TOTALYTD([Costes Totales], 'Calendar'[Fecha])

Carpeta: Ventas
•	Ventas Totales: Consolida las ventas realizadas
Ventas Totales = SUM(Sales[Importe Venta])
•	% Sobre Ventas Total
% Sobre Ventas Total = DIVIDE( [Ventas Totales], CALCULATE([Ventas Totales], ALL(Products)) )
•	Ticket Medio: Calcula el importe medio de ventas por pedido, dividiendo las ventas totales entre el número total de pedidos.
Ticket Medio = DIVIDE([Ventas Totales], [Total de Pedidos],0)
•	Ventas Año Anterior: Ventas del Año Anterior. Obtiene la cifra de ventas del mismo periodo pero del año pasado.
Ventas Año Anterior = 
CALCULATE(
    [Ventas Totales], 
    SAMEPERIODLASTYEAR('Calendar'[Fecha]))
•	Ventas Variación YoY: Calcula el crecimiento o decrecimiento de las ventas comparando el periodo actual con el año anterior.
Ventas Variación YoY = [Ventas Totales] - [Ventas Año Anterior]
•	Ventas Variación YoY %: Calcula el crecimiento o decrecimiento en % de las ventas comparando el periodo actual con el año anterior
Ventas Variación YoY % = DIVIDE([Ventas Variación YoY], [Ventas Año Anterior], 0) 
•	Ventas YTD: Acumulado Anual: Permite mostrar cuánto dinero ha ingresado la empresa desde el 1 de enero hasta el día seleccionado en el filtro.
Ventas YTD = TOTALYTD([Ventas Totales], 'Calendar'[Fecha]) 


Carpeta: Margen
•	Margen %: Suma del importe de transacciones monetarias consolidado uniformemente en USD
Margen % = 
DIVIDE(
    [Margen Total], 
    [Ventas Totales], 
    0
)
•	Margen Año Anterior: Calcula el margen bruto correspondiente al mismo periodo del año anterior, utilizando la fecha seleccionada en el calendario para realizar la comparación interanual.
Margen Año Anterior = 
CALCULATE(
    [Margen Total], 
    SAMEPERIODLASTYEAR('Calendar'[Fecha]))
•	Margen Total: Calcula el margen bruto en términos monetarios, obtenido como la diferencia entre las ventas y los costes totales.
Margen Total = [Ventas Totales] - [Costes Totales] 
•	Margen Variación YoY: Calcula la variación interanual del margen bruto, comparando el margen total del periodo actual con el margen del mismo periodo del año anterior.
Margen Variación YoY = [Margen Total] - [Margen Año Anterior]
•	Margen Variación YoY %: Calcula la variación interanual del margen bruto en %, comparando la diferencia entre el margen actual y el del año anterior respecto al margen del mismo periodo del año anterior. 
Margen Variación YoY % = DIVIDE([Margen Variación YoY], [Margen Año Anterior], 0) 
•	Margen YTD: Calcula el margen bruto acumulado desde el inicio del año hasta la fecha seleccionada, permitiendo analizar la evolución de la rentabilidad durante el año.
Margen YTD = TOTALYTD([Margen Total], 'Calendar'[Fecha]) 

Carpeta: Clientes
•	Clientes Únicos: Cuenta los clientes distintos que compraron en la plataforma.
Clientes Únicos = DISTINCTCOUNT('Sales'[CustomerKey])
•	% Masculino: Mide la participación porcentual de pedidos realizados por hombres sobre el total.
% Masculino = DIVIDE(
    CALCULATE(
        [Total de Pedidos],
        Customers[Gender] = "Male"
    ),
    CALCULATE(
        [Total de Pedidos],
        REMOVEFILTERS(Customers[Gender])
    )
)
•	% Femenino: Mide la participación porcentual de pedidos realizados por mujeres sobre el total.
% Femenino = DIVIDE(
    CALCULATE(
        [Total de Pedidos],
        Customers[Gender] = "Female"
    ),
    CALCULATE(
        [Total de Pedidos],
        REMOVEFILTERS(Customers[Gender])
    )
)
Carpeta: Productos
•	Precio Catálogo Medio: Calcula el precio medio de catálogo de los productos, obteniendo el promedio de los precios unitarios definidos en la tabla de productos.
Precio Catálogo Medio = AVERAGE(Products[UnitPriceUSD])
•	Total Lineas de Producto: 
Total Lineas de Producto = DISTINCTCOUNT('Sales'[line Item])
•	Total Productos: Total Productos en el Catalogo: cuenta cuantos productos hay.
Total Productos = COUNTROWS(Products)
•	Total Productos Vendidos: cuenta cuantos productos se han vendido en total.
Total Productos Vendidos = SUM('Sales'[Quantity])
•	Unidades Año Anterior
Unidades Año Anterior = 
CALCULATE(
    [Total Productos Vendidos],
    SAMEPERIODLASTYEAR('Calendar'[Fecha])
)
•	Variación Unidades Vendidos %
Variación Unidades Vendidos % = 
DIVIDE(
    [Total Productos Vendidos] - [Unidades Año Anterior],
    [Unidades Año Anterior],
    0
)
Carpeta: Pedidos
•	Pedidos Año Anterior: Consolida el número total de pedidos únicos que se realizaron en el mismo periodo de tiempo del año calendario anterior.
CALCULATE(
    [Total de Pedidos],
    SAMEPERIODLASTYEAR('Calendar'[Fecha])
)
•	Promedio Días Entrega: Calcula el promedio real de días que tardan los envíos en llegar a los clientes.
Promedio Días Entrega = AVERAGEX(
    FILTER(
        'Sales', 
        NOT(ISBLANK('Sales'[Días Entrega]))
    ),'Sales'[Días Entrega]
)
•	Total de Pedidos: Total de pedidos realizados. Cuántos "números de orden" únicos existen. Una roden es un cliente que ordena varios tipos de producto y de cada tipo de producto varias unidades. Por tanto un solo pedido puede tener varias transacciones()Order Number una por cada cliente-producto-fecha.
Total de Pedidos = DISTINCTCOUNT('Sales'[Order Number])
•	Variación Pedidos %: Calcula el crecimiento o decrecimiento porcentual interanual (YoY) en el número de pedidos realizados.
Variación Pedidos % = 
DIVIDE(
    [Total de Pedidos] - [Pedidos Año Anterior],
    [Pedidos Año Anterior],
    0
)

5. ESTRUCTURA DE PÁGINAS E INTERACTIVIDAD DEL REPORTE
Resumen Ejecutivo: Consolida y unifica las métricas vitales de salud del negocio en un panel lateral de tarjetas KPI: Ventas Totales ($55.35 mill.), Total Margen (32.43 mill.), Total Coste (22.923 mill.) con una variación del ↑ 12.3% vs. el Año Anterior, y Unidades Vendidas (198 mil) con un incremento del ↑ 12.6%. Estas métricas se combinan con un gráfico de líneas y columnas agrupadas de evolución mensual de ingresos y costes, donde la línea de Margen % se mantiene de forma muy saludable fluctuando establemente entre el 57% y 59% (eje secundario) a lo largo de los años 2016-2019. 
Uso de Marcadores (Bookmarks): Se implementó un panel interactivo con marcadores mediante los botones de selección vertical "Canal Online" y "Tienda Física". Al pulsarlos (por ejemplo, al tener seleccionado Canal Online), se reconfiguran dinámicamente los visuales inferiores de "Venta por Categoría" (donde destaca Computers como líder de ventas en línea con $15.0 mill., seguido de Home Appliances con $8.6 mill.) y el de "Evolución Ganancias" (que compara de forma acumulativa el Margen YTD vs. Año Anterior), facilitando una comparativa directa entre canales sin recargar visualmente el lienzo.
Drillthrough (Obtención de detalles): Configurado como origen de navegación. Al hacer clic derecho en cualquier barra del gráfico de categorías (por ejemplo, sobre Computers), el usuario puede seleccionar "Obtener detalles" para viajar de forma contextual a la Página Detalle de Ventas de detalle transaccional con el filtro de categoría ya aplicado.
Analisis de Producto: Esta página muestra la rentabilidad y estructura de costos del negocio. Presenta tarjetas KPI con las Unidades Vendidas (3 mil, ↑ 12,6% vs. Año Anterior) y un Precio Medio ($356.83). Incluye un gráfico de barras del Estado de los Pedidos (Pendiente vs. Entregado) para identificar retrasos, una matriz cruzada para analizar ventas e ingresos por color, y un gráfico de Esquema de Árbol (Decomposition Tree). 
Parámetros de Campo y Marcadores: Los botones de marcador VENTAS, COSTES y MARGEN controlan un parámetro de campo que reconfigura el Esquema de Árbol. Al seleccionar COSTES (como se ve en tu captura), el árbol desglosa de manera jerárquica los $22.92M de costes totales en Category (Computers: $7.95M), Subcategory (Desktops: $4.22M) y Brand (Adventure Works: $2.35M) con un solo clic.

Detalles de Producto: Muestra en una matriz la ficha técnica y comercial completa de cada artículo del catálogo, detallando: ProductName, Color, Brand, Category, Subcategory, UnitCostUSD, UnitPriceUSD, RangoPrecio (columna calculada) y Total Productos. 
Navegación e Interactividad: Vinculada al menú superior interactivo del reporte para alternar ágilmente entre el análisis ejecutivo y la auditoría del portafolio físico, y equipada con el botón de retorno (🔙) para regresar de forma fluida a la página de origen.

6. CONCLUSIONES, APRENDIZAJES E INSIGHTS DE NEGOCIO
Aprendizajes del Módulo
Se comprendieron las bases del modelado de datos estrella y el valor de optimizar la ETL en Power Query en lugar de saturar el modelo con cálculos DAX redundantes. El mantenimiento del valor de nulos en Power Query para la columna Días Entrega y la estructuración del tipo de cambio utilizando LOOKUPVALUE demuestran que es posible resolver problemas reales de negocio optimizando el rendimiento de la RAM activa.
Comprender la importancia de una arquitectura de datos sólida basada en el diseño de un esquema en estrella (Star Schema). 
La estructuración de la tabla de calendario y su marcado oficial en Power BI resultaron fundamentales para que las fórmulas de Inteligencia de Tiempo (como SAMEPERIODLASTYEAR y TOTALYTD) pudieran calcular con total precisión la evolución histórica y las comparaciones interanuales.
Entender el uso de marcadores y parámetros de campos que son muy útiles para mostrar más información en menos espacio. 
El uso de páginas de ayuda para mostrar muchos datos en los tooltips y así ahorar mas espacio y organizar el panel.
Organizar las ideas en paginas para mostrar los datos que permitan entender la evolución y salud financiera del negocio.
Dificultades encontradas
El Desafío Multidivisa (LOOKUPVALUE y Lógica Financiera): La mayor dificultad técnica radicó en normalizar transacciones originadas en diferentes monedas (EUR, GBP, CAD, etc.) para evitar el gravísimo error financiero de sumar divisas distintas. Lo solucionaste implementando de forma transaccional la función LOOKUPVALUE para buscar el tipo de cambio diario exacto en la tabla Exchange_Rates e integrarlo en la tabla Sales.
En la columna Delivery Date existían valores vacíos (null) correspondientes a pedidos que aún no habían sido entregados. Al calcular los días de reparto, usar un valor predeterminado como cero habría alterado de forma grave el promedio logístico de la empresa. Lo solucione manteniendo el valor null desde Power Query y aplicando la función AVERAGEX con un filtro condicional NOT(ISBLANK) en DAX para ignorar las transacciones activas y calcular una media real de entrega.
Insights Clave de los Datos
•	Se detectaron picos históricos masivos y recurrentes en el cuarto trimestre de cada año, específicamente en noviembre y diciembre.
•	El análisis del catálogo confirma que los artículos más vendidos pertenecen a la categoría Computers.
•	Aunque el e-commerce (Online) muestra una gran cobertura geográfica y un excelente promedio de compra las tiendas físicas concentran la mayor cantidad de ingresos brutos de la compañía.
•	Se muestra una tendencia constante de mejora en el Promedio Días Entrega por Año, logrando un descenso progresivo en el tiempo de despacho año tras año.
•	La mayor cantidad de clientes acumulados de la tienda tiene más de 60 años.
•	La mayor parte de las tiendas físicas y la mayor concentración de la base de clientes se localizan en Estados Unidos.
•	El cliente estrella (Matthew Flemming) que ha realizado la mayor cantidad de pedidos es a su vez el que más dinero ha gastado ($61,871.70). Sin embargo, esta relación lineal no se cumple en el resto del ranking, donde clientes con muy pocos pedidos (como Roy Le con solo 3 compras) acumulan gastos elevados ($49,704.93).
