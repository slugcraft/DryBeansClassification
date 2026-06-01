# Clasificación de Frijoles con un modelo entrenado

Miguel Angel Uribe Esquivel     A01277614

## Introducción
Los frijoles secos pertenecen a una extensa familia de leguminosas, la cual se considera que es la más producida a nivel mundial. Originarios de América, estos frijoles se han distribuido ampliamente por casi todas las regiones del mundo, propagando múltiples especies a nivel global y generando nuevas variantes a medida que las semillas se seleccionan según las características óptimas para su cultivo. El principal desafío que enfrentan los agricultores no reside únicamente en el cuidado de la vida de la planta, sino en la clasificación posterior a la cosecha. El proceso manual de clasificación de las semillas de frijol representa una tarea considerablemente compleja y prolongada, lo que puede dificultar la identificación de semillas aptas para ser enviadas a las huertas o comercializadas. 

Bajo este concepto, el objetivo de este proyecto será el de **desarrollar un modelo que logre clasificar de manera efectiva las siete variedades de frijol más comunes en la región de Turquía** basadas en sus características fisiológicas, como lo son su área, perimetro, consistencia, patrón, entre otros más.

### Descripción del Dataset
El conjunto de datos utilizado para este proyecto se obtuvo del UC Irvine Machine Learning Repository (https://archive.ics.uci.edu/dataset/602/dry+bean+dataset). Este conjunto de datos comprende de 13,611 instancias de datos recopiladas de las siete variedades de frijol seco más comunes en la región de Turquía, las cuales son: Seker, Barbunya, Bombay, Cali, Dermosan, Horoz y Sira. Cada instancia de frijol seco se caracteriza por 16 atributos, de los cuales 13 son de tipo flotante, 2 son de tipo entero y 1 es un object.

La información contenida en este conjunto de datos se derivó de un estudio fotográfico exhaustivo de miles de imágenes de las variedades de frijol antes mencionadas. Los datos de estas imágenes fueron digitalizados para extraer las siguientes características:

| **Característica** | **Tipo** | **Descripción**
|--------------------|----------|------------------
|Area (A)	         | Integer	| The area of a bean zone and the number of pixels within its boundaries
|Perimeter (P)	     | Float	| Bean circumference is defined as the length of its border.	
|MajorAxisLength (L) | Float	| The distance between the ends of the longest line that can be drawn from a bean		
|MinorAxisLength (l) | Float	| The longest line that can be drawn from the bean while standing perpendicular to the main axis		
|AspectRatio (K)	 | Float	| Defines the relationship between MajorAxisLength and MinorAxisLength		
|Eccentricity (Ec)	 | Float	| Eccentricity of the ellipse having the same moments as the region		
|ConvexArea (C)	     | Integer	| Number of pixels in the smallest convex polygon that can contain the area of a bean seed	
|EquivDiameter (Ed)	 | Float	| Equivalent diameter: The diameter of a circle having the same area as a bean seed area		
|Extent (Ex)         | Float	| The ratio of the pixels in the bounding box to the bean area		
|Solidity (S)    	 | Float	| Also known as convexity. The ratio of the pixels in the convex shell to those found in beans.		
|Roundness (R)	     | Float    | Calculated with the following formula: (4piA)/(P^2)		
|Compactness (Co)	 | Float    | Measures the roundness of an object	
|ShapeFactor1		 | Float	| L/A	
|ShapeFactor2		 | Float	| l/A	
|ShapeFactor3		 | Float	| A/((L/2)*(L/2)*pi)
|ShapeFactor4		 | Float    | A/((L/2)*(l/2)*pi)
|Class               | Object   | Dry-bean species

<p align="center">
  <img src="./imagenes/frijoles_secos.png" alt="frijoles secos" width="50%"/>
  <br>
  <em>Imagen 1. Tipo de frijoles secos presentes en el dataset</em>
</p>

## Metodología

### Preprocesamiento

El conjunto de datos consta de 13,611 instancias y 16 columnas. De estas 16 columnas, 15 representan las características físicas de los frijoles, previamente descritas.  De las 15 columnas numéricas, 13 son de tipo `float` y 2 son de tipo `integer`. La variable de tipo `object` corresponde a la clase de frijol seco a la que pertenece cada instancia, la cual será utilizada como variable categórica.

De los 13,611 registros que componen el dataset, la distribucion de los datos se encuentra de la siguiente forma por clase:
- Seker: 2027
- Barbunya: 1322
- Bombay: 522
- Cali: 1630
- Horoz: 1928
- Sira: 2636
- Dermason: 3546

<p align="center">
  <img src="./imagenes/distribucion_clases.png" alt="grafico de distribución de clases" width="80%"/>
  <br>
  <em>Gráfico 1. Distribución de datos por clases</em>
</p>

Se observa un desbalance significativo en la distribución de datos entre las distintas clases. No obstante, no considero que este desbalance impacte considerablemente al rendimiento del modelo. Por ejemplo, la variedad de frijol con menor cantidad de datos es la Bombay, la cual, paradójicamente, es la más notablemente grande en comparación con las demás especies. La siguiente variedad con menos datos es la Barbunya, la cual presenta patrones de manchas en la superficie de su grano, lo que facilita su diferenciación del resto, que son completamente lisos y blancos.  Asimismo, es razonable que exista una mayor cantidad de datos para las variedades Sira y Dermason, dado que ambas presentan similitudes físicas considerables con diferencias sutiles; por consiguiente, se requiere una mayor cantidad de datos para estas dos variedades con el fin de que el modelo pueda diferenciarlas con precisión.

<p align="center">
  <img src="./imagenes/frijoles_lectura.png" alt="Diferencias frijoles" width="50%"/>
  <br>
  <em>Imagen 2. Diferencias morfológicas por especie</em>
</p>

#### Mapa de correlación

Para este conjunto de datos, se utilizó un mapa de calor con el fin de identificar la dependencia y correlación entre las variables. El gráfico revela una alta correlación entre las primeras variables de dimensiones, correspondientes al área, perímetro, eje longitudinal mayor y eje longitudinal menor. Esta alta dependencia mutua es razonable, dado que los ejes longitudinales dependen en gran medida del perímetro del frijol, y matemáticamente, el área del frijol también depende de su perímetro.  Asimismo, existe una relación con otras variables de medición similares, como el área convexa y el diámetro equivalente, así como los factores de forma, que comparten cálculos similares. A pesar de la presencia de múltiples variables redundantes, cada una cumple un propósito en el cálculo longitudinal de los frijoles. Por lo tanto, la eliminación de alguna de estas variables podría sesgar los datos de diferencia entre especies similares. En consecuencia, se han conservado todas las variables para el entrenamiento del modelo.

<p align="center">
  <img src="./imagenes/mapa_calor.png" alt="Mapa de calor" width="60%"/>
  <br>
  <em>Gráfico 2. Mapa de calor de correlación</em>
</p>