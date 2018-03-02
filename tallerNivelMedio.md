Taller nivel medio
================
Cristina Heredia GÃ³mez (@\_musicalnote)
19-mayo-2017

Bienvenidos al taller nivel medio de R. En Ã©l aprenderÃ¡s a hacer mÃ¡s eficiente tu cÃ³digo con alternativas a los bucles, como son **apply**, **sapply**, **lapply**. TambiÃ©n aprenderÃ¡s a paralelizar tu cÃ³digo R, usando mÃ¡s de un nÃºcleo, y por Ãºltimo aprenderÃ¡s con un ejemplo distintas formas de particionar un dataset y a hacer algunas de las representaciones grÃ¡ficas mÃ¡s comunes, como boxplots, nube de puntos o hibstogramas. Â¡Empezamos!

Parte 1: Agilizando el cÃ³digo - programaciÃ³n funcional
======================================================

Una prÃ¡ctica muy recomendable en R es evitar los bucles, sobre todo los **for**. Incluso hay quien prohÃ�be su uso totalmente... la razÃ³n principal es la eficiencia. Â¿Pero tanto se nota? Bueno... eso jÃºzgalo tÃº a continuaciÃ³n. En el siguiente cÃ³digo vamos a crear un conjunto de datos. Para ello sÃ�mplemente vamos a ir generando los datos mediante una distribuciÃ³n normal aleatoria con **rnorm**, especificando el nÃºmero de observaciones que queremos crear, la media y la desviaciÃ³n tÃ�pica de las mismas. Lo hacemos dentro de un for, y creamos el conjunto de datos con 10000 filas y 200 columnas:

``` r
# Creamos el conjunto de datos con un bucle for

dataset<-NULL

for(i in 1:200){
    dataset<-cbind(dataset,rnorm(10000,i,1))
}
# convertimos a data frame
dataset<-as.data.frame(dataset)
# renombramos las columnas
colnames(dataset)<-paste("c",1:200,sep = "")
# mostramos dos lÃ�neas de las 10 primeras columnas del conjunto creado
head(dataset[,1:10],2)
```

    ##           c1       c2       c3       c4       c5       c6       c7
    ## 1  0.4257519 1.466172 3.666870 3.637800 3.593805 5.880237 6.423245
    ## 2 -0.7323221 4.066882 4.233015 4.742546 5.763055 7.149386 8.010008
    ##         c8       c9       c10
    ## 1 6.972277 8.296800  9.359741
    ## 2 8.326721 8.165355 10.106913

Te habrÃ¡s percatado de que tarda unos segundos. Ahora hacemos lo mismo funcional:

``` r
# lo hacemos funcional
dataset<-NULL

dataset <-lapply(1:200, function(i) rnorm(10000,i,1) )
# convertimos a data frame
dataset<-as.data.frame(dataset)
# renombramos las columnas
colnames(dataset)<-paste("c",1:200,sep = "")
head(dataset[,1:10],2)
```

    ##          c1        c2       c3       c4       c5       c6       c7
    ## 1 1.6362300 1.9480161 3.668058 5.314274 4.357837 6.440521 7.557654
    ## 2 0.3792762 0.8968573 2.014382 2.144908 3.522807 4.468471 6.274952
    ##         c8       c9      c10
    ## 1 7.880451 8.828865 10.54529
    ## 2 6.565617 9.840067 10.15166

Â¿Has notado la diferencia? Aunque en este caso la diferencia de tiempo no es muy grande (~3s), es sÃ³lo un ejemplo muy simple para ilustrar. Cuando trabajamos con datasets mÃ¡s grandes u operaciones mÃ¡s complejas la diferencia puede llegar a ser de horas.

#### Ejercicio 1:

**Puedes comprobar como la diferencia de tiempo aumenta repitiendo lo anterior creando un dataset mÃ¡s grande.**

Apply, lapply y sapply
----------------------

**Funcional** debe su nombre a que recibe una funciÃ³n como parÃ¡metro, asÃ� que tanto apply, como sapply y lapply recibirÃ¡n una funciÃ³n entre sus parÃ¡metros.

### Apply

Cuando tenemos un conjunto de datos sobre el que queremos hacer alguna operaciÃ³n por filas o columnas, podemos usar **apply(datos, 1|2, funciÃ³n)**, donde ***datos*** son los datos sobre los que queremos operar, ***1*** es un flag para indicar si queremos operar por filas o ***2*** si queremos operar por columnas, y ***funciÃ³n*** es la funciÃ³n que queremos aplicar a cada fila o columna de los datos proporcionados. Es importante que los datos sean del mismo tipo para poder realizar las mismas operaciones sobre ellos.

Por ejemplo, podemos usar **apply** para calcular la media de cada columna de nuestro gran dataset:

``` r
col.means<-apply(dataset,2,mean)
# mostramos sÃ³lo las 30 Ãºltimas lÃ�neas por simplificar la salida
tail(col.means,30)
```

    ##     c171     c172     c173     c174     c175     c176     c177     c178
    ## 171.0136 172.0167 172.9978 173.9936 175.0088 175.9849 176.9969 178.0030
    ##     c179     c180     c181     c182     c183     c184     c185     c186
    ## 179.0045 179.9894 180.9764 181.9937 182.9951 183.9744 184.9810 186.0109
    ##     c187     c188     c189     c190     c191     c192     c193     c194
    ## 187.0063 187.9843 188.9939 190.0032 191.0130 192.0057 192.9822 193.9817
    ##     c195     c196     c197     c198     c199     c200
    ## 194.9971 196.0301 196.9927 197.9846 198.9938 199.9905

#### Ejercicio 2:

**Usa Apply para calcular el coeficiente de variaciÃ³n (desviaciÃ³n tÃ�pica dividido por la media) de cada columna de nuestro conjunto de datos.**

### Lapply

Esta funciÃ³n recibe unos datos, que suelen ser una lista o un vector, y una funciÃ³n como parÃ¡metros. Su funcionamiento consiste en recorrer los datos especificados y aplicarles la funciÃ³n especificada. Es como una versiÃ³n de Apply que devuelve el resultado en forma de lista, por eso la **L**.

Por ejemplo, podemos usar **lapply** para calcular cuÃ¡ntos nÃºmeros negativos hay en cada columna de nuestro dataset:

``` r
negatives.count<-lapply(1:ncol(dataset), function(i){
  length(which(dataset[[i]]<0))
})

# mostramos sÃ³lo un trozo del output por simplicidad
head(negatives.count,4)
```

    ## [[1]]
    ## [1] 1611
    ##
    ## [[2]]
    ## [1] 230
    ##
    ## [[3]]
    ## [1] 9
    ##
    ## [[4]]
    ## [1] 0

Como veis, nos devuelve el resultado en una lista. Si quisiÃ©ramos el resultado en forma de vector en vez de lista, podemos usar **unlist** sobre la salida de **lapply**.

### Sapply

La sintaxis es la misma que la de **lapply**, cambiando el nombre de la funciÃ³n. La dinÃ¡mica tambiÃ©n es la misma, pero **sapply** devuelve como resultado un vector en lugar de una lista.

Por ejemplo, podemos repetir el cÃ³digo anterior usando **sapply**, y veremos que sÃ³lo difiere en el formato que presenta la salida:

``` r
negatives.count<-sapply(1:ncol(dataset), function(i){
  length(which(dataset[[i]]<0))
})

# mostramos la salida
negatives.count
```

    ##   [1] 1611  230    9    0    0    0    0    0    0    0    0    0    0    0
    ##  [15]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ##  [29]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ##  [43]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ##  [57]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ##  [71]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ##  [85]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ##  [99]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ## [113]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ## [127]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ## [141]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ## [155]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ## [169]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ## [183]    0    0    0    0    0    0    0    0    0    0    0    0    0    0
    ## [197]    0    0    0    0

### Filtrado elegante con Dplyr

*Dplyr* es un paquete creado para la manipulaciÃ³n de datos, proporciona un conjunto de funciones para ayudar a resolver los problemas de manipulaciÃ³n de datos mÃ¡s comunes, entre ellos:

- `mutate()` AÃ±ade nuevas variables a un dataset en funciÃ³n de variables ya existentes.
- `select()` Selecciona variables basandose en sus nombres.
- `filter()` Selecciona instancias basandose en sus valores.
- `summarise()` Reduce varios valores a uno solo.
- `arrange()` Re-ordena las filas.

Para ponerlo en práctica, vamos a utilizar un dataset existente en R llamado **Orange**.
Para que lo conozcamos mejor, vamos a hacer un anÃ¡lisis exploratorio de este dataset, que incluye datos sobre el crecimiento de 5 naranjos, concretamente la edad del Ã¡rbol y el grosor de su tronco, cuando se le realizÃ³ la mediciÃ³n.

``` r
# InformaciÃ³n sobre estructura
str(Orange)
```

    ## Classes 'nfnGroupedData', 'nfGroupedData', 'groupedData' and 'data.frame':   35 obs. of  3 variables:
    ##  $ Tree         : Ord.factor w/ 5 levels "3"<"1"<"5"<"2"<..: 2 2 2 2 2 2 2 4 4 4 ...
    ##  $ age          : num  118 484 664 1004 1231 ...
    ##  $ circumference: num  30 58 87 115 120 142 145 33 69 111 ...
    ##  - attr(*, "formula")=Class 'formula'  language circumference ~ age | Tree
    ##   .. ..- attr(*, ".Environment")=<environment: R_EmptyEnv>
    ##  - attr(*, "labels")=List of 2
    ##   ..$ x: chr "Time since December 31, 1968"
    ##   ..$ y: chr "Trunk circumference"
    ##  - attr(*, "units")=List of 2
    ##   ..$ x: chr "(days)"
    ##   ..$ y: chr "(mm)"

``` r
# Resumen del dataset
summary(Orange)
```

    ##  Tree       age         circumference
    ##  3:7   Min.   : 118.0   Min.   : 30.0
    ##  1:7   1st Qu.: 484.0   1st Qu.: 65.5
    ##  5:7   Median :1004.0   Median :115.0
    ##  2:7   Mean   : 922.1   Mean   :115.9
    ##  4:7   3rd Qu.:1372.0   3rd Qu.:161.5
    ##        Max.   :1582.0   Max.   :214.0

Con **str** vemos que el dataset estÃ¡ compuesto por 35 filas y 3 columnas, y que una de sus columnas son factores, mientras que las otras dos son numÃ©ricas. Con **summary** obtenemos un resumen del dataset, como cual es el mÃ�nimo, mÃ¡ximo, media, mediana... de cada columna. Nota: En la columna *Tree* solo se muestra el nÃºmero de muestras de casa clase porque los datos no son numÃ©ricos.

Y ahora, vamos a poner en práctica lo que sabemos de *Dplyr*:

``` r
require(dplyr)

Orange %>% dplyr::mutate(circumferenceTimes2 = circumference^2)
dplyr::mutate(Orange, circumferenceTimes2 = circumference^2) # Equivalente

Orange %>% dplyr::select(age)  # Selecciona solo columna age
Orange %>% dplyr::select(-age) # Selecciona todas menos age
Orange %>% dplyr::select(age:circumference) # Seleccionar desde age a circumference
Orange %>% dplyr::select_if(is.numeric) # Seleccionar las columnas que cumplan ciertos requisitos

Orange %>% dplyr::filter(age < 500) # Seleccionar filas cuya columna age sea menor de 500
Orange %>% dplyr::filter(circumference %in% 1:100) # Seleccionar filas cuya circunferencia estÃ© comprendida entre 1 y 100
Orange %>% dplyr::filter(age < 500, circumference %in% 1:30)

Orange %>% dplyr::summarise(edadMediana = median(age)) # La mediana de la edad

Orange %>% dplyr::arrange(age, circumference) # Ordena las filas segÃºn el criterio dado

## Se pueden encadenar operaciones
Orange %>%
    dplyr::mutate(circumferenceTimes2 = circumference^2) %>%
    dplyr::filter(age < 500) %>%
    dplyr::arrange(age, circumference)
```

#### Ejercicio 3:

**El ejemplo que hemos hecho con lapply y sapply no es muy reutilizable, ya que estamos asumiendo que existe una variable que se llama *dataset* y que contiene un dataset. Â¿CÃ³mo podemos arreglarlo?** **Pista: las funciones pueden recibir mÃ¡s de un sÃ³lo parÃ¡metro.**

#### Ejercicio 4:

**Usa programaciÃ³n funcional para sustituir por *NAs* todos los nÃºmeros negativos del dataset. Puedes usar *length(which(is.na(dataset\[\[i\]\])))* para comprobar si lo has hecho bien, comprobando que el nÃºmero de NAs de cada columna coincide con el nÃºmero de negativos que obtuvimos anteriormente para cada columna.**

Por si quieres saber mÃ¡s: paralelizando en R
--------------------------------------------

Una prÃ¡ctica muy Ãºtil en R es el paralelismo. Supongamos que queremos ejecutar una misma funciÃ³n sobre distintos datasets del mismo tamaÃ±o. Si los datasets fueran aproximadamente del tamaÃ±o del nuestro, o la funciÃ³n fuera costosa en cÃ³mputo, podrÃ�a tardar mucho tiempo en realizar las ejecuciones.
Una opciÃ³n para acortar ese tiempo es hacerlo en paralelo, es decir, utilizar varios nÃºcleos del ordenador para realizar la tarea, y si has entendido **apply**, **lapply** y **sapply** es muy directo. Las tres funciones explicadas arriba, tienen una versiÃ³n paralela: **parApply**, **parLapply** y **parSapply** incluidas en el paquete **parallel**. Para usarlas, sÃ³lo hay que tener en cuenta que es necesario crear un cluster con tantos cores como deseemos utilizar, con la funciÃ³n **makeCluster()** y pasarle el cluster a la funciÃ³n **parApply** que deseemos utilizar, ademÃ¡s de los parÃ¡metros que ya le pasÃ¡bamos anteriormente. Por Ãºltimo, una buena prÃ¡ctica es detener el cluster creado cuando ya no lo necesitamos con **stopCluster()**.

Por ejemplo, se puede paralelizar la generaciÃ³n de 25 soluciones binarias aleatorias y su optimizaciÃ³n a travÃ©s de un algoritmo de bÃºsqueda local:

``` r
# cargamos librerÃ�a
  library(parallel)
# obtenemos nuestro nÃºmero de cores
  no_cores <- detectCores()
# creamos el cluster
  cl <- makeCluster(no_cores-1,type="FORK")
# utilizamos parLapply , pasÃ¡ndole el cluster
  ModelosBL <- parLapply(cl,seq_along(1:25),  function(i){
    set.seed(12345*i)
    # genera una soluciÃ³n vecina aleatoria
    vecina<-sample(0:1,350,replace=TRUE)
    # optimizar con bÃºsqueda local
    modelo<-LocalSearchModified(vecina)
    modelo
  })
# detenemos el cluster
stopCluster(cl)
```

(No se incluye el cÃ³digo del algoritmo de BÃºsqueda local).

#### Ejercicio 5 (Opcional):

**Crea otro dataset como el que creamos al comienzo del taller, con distinto nombre. Luego crea una lista con ambos datasets usando la funciÃ³n *list* , y paraleliza el cÃ³digo para aplicar sobre cada dataset una funciÃ³n que eleva cada valor al cuadrado. Â¡VerÃ¡s quÃ© rÃ¡pido lo hace a pesar del tamaÃ±o de los datasets!** **Nota: Es conveniente que crees el cluster con tu nÃºmero de cores -1.**

El paquete **parallel** incluye otras funciones que puedes consultar en [documentaciÃ³n paquete parallel](https://stat.ethz.ch/R-manual/R-devel/library/parallel/doc/parallel.pdf "paquete parallel").

Parte 2: VisualizaciÃ³n y particionado de datos
==============================================

Una buena prÃ¡ctica cada vez que tenemos delante un conjunto de datos es explorar su contenido, echando un vistazo a la estructura de las variables, los rangos en los que se mueven, de quÃ© tipo son... Esto nos da una idea de ante quÃ© problema estamos, y como podemos abordarlo. Para hacer el anÃ¡lisis exploratorio de los datos, podemos usar funciones como **class** para ver la clase de los objetos, **str** para ver informaciÃ³n sobre la estructura de los datos o **summary** para ver un resumen del dataset. TambiÃ©n es comÃºn hacer una representaciÃ³n visual como una nube de puntos, para ver visualmente cÃ³mo se distribuyen los datos, y boxplots o hibstogramas para visualizar los rangos en los que toman valores las variables. Veamos un ejemplo:

Con rstudio tenemos incluidos varios datasets que vienen incluidos en el paquete *datasets*. Para ver una lista completa de los datasets incluidos:

``` r
data()
```

El dataset **Orange** que vimos antes, es uno de ellos.

### Nube de puntos

Podemos hacer una primera exploraciÃ³n visual de los datos pintando los datos con la funciÃ³n **qplot** de **ggplot2**. Si no la tenemos instalada, la instalamos con `install.packages('ggplot2')`.

``` r
library(ggplot2)
qplot(age, circumference, data = Orange, geom = "point",xlab = "aÃ±os", ylab="circuferencia del tronco", colour=Tree, main="Crecimiento de los Ã¡rboles de naranja")+theme(plot.title = element_text(hjust = 0.5))+scale_color_discrete(name="Ãrbol")
```

![](tallerNivelMedio_files/figure-markdown_github/unnamed-chunk-8-1.png)

**qplot** es una forma rÃ¡pida y sencilla de hacer grÃ¡ficos completos. Le indicamos los datos que queremos representar, quÃ© geometrÃ�a deseamos ( *geom* ) y quÃ© deseamos poner en los ejes. Con `colour=Tree`, indicamos que nos coloree los datos en funciÃ³n de la clase a la que pertenezca, con los colores por defecto. Si quisiÃ©ramos darle unos colores personalizados, podrÃ�amos hacerlo aÃ±adiendo una nueva capa con **scale\_color\_manual**, lo que nos permitirÃ¡ cambiar los colores manualmente. Por ejemplo, aÃ±adiendo: `scale_color_manual(values = c("#ff0000","#ff4000","#ff8000","#ffbf00","#ffff00")` cambiarÃ�amos los colores por defecto por unos tonos cÃ¡lidos. Usando el atributo *name* en **scale\_color\_discrete**, ponemos cambiar el tÃ�tulo por defecto que le damos a la leyenda. Ãsto tambiÃ©n podemos hacerlo con **scale\_color\_manual** siempre y cuando especifiquemos tambiÃ©n un conjunto de valores. Por Ãºltimo, podemos aÃ±adirle un tÃ�tulo a nuestro grÃ¡fico usando el parÃ¡metro **main** o con **ggtitle()**, y podemos guardar la imÃ¡gen directamente con *ggsave()*.

En cuanto a los datos, podemos observar que parece que la tendencia para los 5 Ã¡rboles es que va aumentando el tamaÃ±o de la circuferencia de su tronco conforme van pasando los aÃ±os. SÃ�, parece que eso de "a mÃ¡s aÃ±os tienen mÃ¡s grande el tronco" es cierto.

#### Ejercicio 6:

**Â¿Conoces el dataset *iris* ? TambiÃ©n estÃ¡ incluido en el paquete *datasets* y ofrece 150 muestras de tres tipos de flor de lirio, que incluyen informaciÃ³n sobre la longitud y el ancho del sÃ©palo y del pÃ©talo, para cada clase de lirio. Explora el contenido de este dataset(estructura interna, resumen...) y realiza un grÃ¡fico de puntos como el que se hizo anteriormente. AÃ±ade tus propios colores a mano, un tÃ�tulo para la legenda...etc. Responde a las siguientes preguntas:**

**a) Â¿Observas algÃºn problema en los datos?Â¿Crees que las clases son fÃ¡cilmente separables?**

**b) Â¿CuÃ¡l es la longitud media del sÃ©palo segÃºn la especie?** (puedes usar filtros para seleccionar los datos de sÃ©palo por especie y luego calcular su media)

### Boxplots (o diagramas de cajas y bigotes)

Otra visualizaciÃ³n muy tÃ�pica y Ãºtil son los boxplots. Con ellos se representa visualmente cÃ³mo se distribuyen los valores que toma una variable en una cajita. La cajita se divide en tres cuartiles. El segundo cuartil, que se representa con la raya horizontal negra, es la mediana de esos datos. Las rayas verticales que atraviesan la cajita representan valores poco comunes. Concretamente, la raya de abajo representa todos los valores que estÃ¡n entre el primer cuartil y el mÃ�nimo valor que no es un outlier, mientras que la de arriba representa valores entre el tercer cuartil y el mÃ¡ximo valor que no es un outlier.

Por ejemplo, podemos usar boxplots para representar unas variables frente a otras y obtener conclusiones:

``` r
# obtenemos las columnas de aÃ±os y circuferencia
d<-subset(Orange[,-1])
# lo pasamos a data frame para que ggplot lo sepa manejar
d<-as.data.frame(d)
# creamos el grÃ¡fico
ggplot(d,aes(x=factor(age),y=circumference,fill=factor(age)))+geom_boxplot()+xlab("aÃ±os")+ylab("circuferencia")+scale_fill_discrete(name="aÃ±os")+ggtitle("RepresentaciÃ³n de circuferencia de tronco por aÃ±os ")+theme(plot.title = element_text(hjust = 0.5))
```

![](tallerNivelMedio_files/figure-markdown_github/unnamed-chunk-9-1.png)

Donde **d** son los datos que queremos representar (edad y circuferencia), con **fill** indicamos que nos coloree los boxplots con tantos colores por defecto como aÃ±os hay, con **geom\_boxplot()** indicamos que queremos la geometrÃ�a de boxplots, y usamos **xlab** e **ylab** para cambiar los nombres de los ejes. Por Ãºltimo, cambiamos el nombre de la legenda con **scale\_fill\_discrete** como ya sabemos, y aÃ±adimos un tÃ�tulo, que posicionamos.

Con el cÃ³digo anterior, representamos la circuferencia del Ã¡rbol por aÃ±os. De nuevo podemos observar que a mÃ¡s aÃ±os tienen los Ã¡rboles, mayor diÃ¡metro del tronco presentan. TambiÃ©n podemos obervar que hay un rango de aÃ±os \[1004-1582\] donde es difÃ�cil distinguir la edad exacta que tiene el Ã¡rbol en funciÃ³n de la circuferencia, y viceversa, pues muchos valores se solapan.

TambiÃ©n podemos representar la circuferencia del Ã¡rbol frente al tipo de Ã¡rbol que es:

``` r
# obtenemos las columnas de aÃ±os y circuferencia
d<-subset(Orange[,-2])
# lo pasamos a data frame para que ggplot lo sepa manejar
d<-as.data.frame(d)
# creamos el grÃ¡fico
ggplot(d,aes(x=Tree,y=circumference,fill=Tree))+geom_boxplot()+xlab("clase de Ã¡rbol")+ylab("circuferencia")+scale_fill_discrete(name="Ãrbol")+ggtitle("RepresentaciÃ³n de circuferencia de tronco por tipo de Ã¡rbol ")+theme(plot.title = element_text(hjust = 0.5))
```

![](tallerNivelMedio_files/figure-markdown_github/unnamed-chunk-10-1.png)

Pudiendo observar que los Ã¡rboles 1, 3 y 5 han presentado en general en las mediciones realizadas menor diÃ¡metro de tronco que los Ã¡rboles 2 y 4. Sin embargo, dado que los 5 Ã¡rboles son 5 naranjos, es sensato pensar que esto podrÃ�a deberse a que quizÃ¡s no se les han tomado las mediciones a la misma edad. Pero veamos si esta hipÃ³tesis tiene o no fundamento. Representemos la edad a la que se le hicieron las mediciones a los Ã¡rboles frente a los 5 tipos de Ã¡rboles:

``` r
# obtenemos las columnas de aÃ±os y circuferencia
d<-subset(Orange[,-3])
# lo pasamos a data frame para que ggplot lo sepa manejar
d<-as.data.frame(d)
# creamos el grÃ¡fico
ggplot(d,aes(x=Tree,y=age,fill=Tree))+geom_boxplot()+xlab("clase de Ã¡rbol")+ylab("aÃ±o en que se tomÃ³ mediciÃ³n")+scale_fill_discrete(name="Ãrbol")+ggtitle("AÃ±o en el que se tomÃ³ la mediciÃ³n por tipo de Ã¡rbol ")+theme(plot.title = element_text(hjust = 0.5))
```

![](tallerNivelMedio_files/figure-markdown_github/unnamed-chunk-11-1.png)

A la vista del grÃ¡fico, podemos afirmar que los 5 Ã¡rboles fueron medidos cuando tenÃ�an la misma edad, y por eso todos lo boxplots del grÃ¡fico toman los mismos valores. Podemos comprobarlo no obstante con la funciÃ³n **identical**:

``` r
identical(Orange[which(Orange$Tree==1),2],Orange[which(Orange$Tree==2),2])
```

    ## [1] TRUE

``` r
identical(Orange[which(Orange$Tree==1),2],Orange[which(Orange$Tree==3),2])
```

    ## [1] TRUE

#### Ejercicio 7:

**Explora visualmente el dataset de iris, esta vez usando boxplots. Representa la longitud del pÃ©talo por tipo de flor, el ancho del pÃ©talo por tipo de flor y repite lo mismo para el sÃ©palo. Â¿QuÃ© observas en los datos?**

### Histogramas

Lo Ãºltimo que vamos a ver en este taller sobre visualizaciÃ³n son los histogramas. Los histogramas representan la frecuencia con la que ocurren ciertos tÃ©rminos en nuestros datos, de una forma fÃ¡cil de entender. Por ejemplo, podemos usar un histograma para visualizar la frecuencia con la que ocurren los distintos diÃ¡metros de tronco entre los Ã¡rboles del conjunto de datos:

``` r
 qplot(Orange$circumference,geom = "bar",xlab = "circuferencia del tronco(cm)", ylab="Frecuencia")+geom_histogram(binwidth = 5,fill=I("#ff4000"),colour="#ff8000")+ggtitle("Ocurrencia de circuferencia del tronco")+theme(plot.title = element_text(hjust = 0.5))+scale_colour_discrete( guide = FALSE)
```

![](tallerNivelMedio_files/figure-markdown_github/unnamed-chunk-13-1.png)

Utilizamos **fill** para indicar el color de relleno y **colour** para indicar el color de la lÃ�neas del borde. Como queremos especificar en ancho de las bandas, tenemos que aplicar **geom\_hibstogram()**, pues qplot no acepta ese parÃ¡metro, a pesar de que hemos especificado ya que queremos geometrÃ�a de barras. con `scale_colour_discrete( guide = FALSE)` Estamos eliminando la leyenda de nuestro grÃ¡fico.

Podemos hacer lo mismo usando **ggplot**, tambiÃ©n de **ggplot2**:

``` r
ggplot(as.data.frame.numeric(Orange$circumference),aes(x=Orange$circumference,fill=I("#ff4000")))+geom_histogram(binwidth = 5,colour="#ff8000")+xlab("circuferencia del tronco(cm)")+ylab("Frecuencia")+ggtitle("Ocurrencia de circuferencia del tronco")+theme(plot.title = element_text(hjust = 0.5))
```

#### Ejercicio 8:

**Elige un atributo del dataset iris, (por ejemplo, el ancho del pÃ©talo) y representa la frecuencia de sus valores en un histograma. Puedes comprobar que la representaciÃ³n es correcta comprobando luego los valores de la variable.**

**Nota: PodÃ©is sacar el valor hexadecimal de los colores en la web [w3schools](https://www.w3schools.com/colors/colors_picker.asp "seleccionador de colores").**

### Particionado de datos

Una necesidad cuando se trabaja con datos es particionar el conjunto de muestras, especialmente en ciencia de datos, cuando se suele trabajar con un conjunto de entrenamiento y otro de test. Hay muchas formas de particionar un dataset, y dependiendo de lo que necesitemos a veces tendremos que hacerlo de una forma u otra.

La forma mÃ¡s usual de proceder si no se requieren especificaciones concretas, es destinar un porcentaje a test y otro a entrenamiento, que suele ser 60-80% para entrenamiento y el resto para test, dependiendo del nÃºmero de muestras de las que se disponga. Por ejemplo, si queremos destinar el 65% de los datos de **Orange** a entrenamiento y el resto para test:

``` r
# obtenemos los primeros datos que formen el 65%
nMuestrasTrain<-as.integer(nrow(Orange)*0.65)
# asignamos a sus respectivos conjuntos
train<-Orange[1:nMuestrasTrain,]
test<-Orange[(nMuestrasTrain+1):nrow(Orange),]
# comprobamos el nÃºmero de muestras de cada dataset
nrow(train)
```

    ## [1] 22

``` r
nrow(test)
```

    ## [1] 13

Otra forma de dividir un dataset es aleatorizando las muestras que se distribuyen en cada conjunto. En el ejemplo anterior, creÃ¡bamos un conjunto de train con el 65% de las primeras muestras, mientras que el resto iba para test. Pero asÃ� siempre tendrÃ�amos las mismas muestras para train y para test. Una forma de resolverlo, es destinar el 65% de las muestras a train tomando esas muestras aleatoriamente, y aÃ±adir las restantes al conjunto de test:

``` r
# tomamos n muestras aleatorias del dataset
muestrasAleatorias<-sample(1:nrow(Orange),nMuestrasTrain)
# asignamos a sus respectivos conjuntos
train<-Orange[muestrasAleatorias,]
test<-Orange[-muestrasAleatorias,]
# comprobamos el nÃºmero de muestras de cada dataset
nrow(train)
```

    ## [1] 22

``` r
nrow(test)
```

    ## [1] 13

Pero a veces, se necesita particionar los datos de una forma mÃ¡s "avanzada", como puede ser un particionamiento estratificado, o crear varias particiones. Esto puede resultar un quebradero de cabeza en algunos casos, por eso querÃ�a aprovechar y aÃ±adir esta secciÃ³n para mencionar el paquete [caret](https://cran.r-project.org/web/packages/caret/index.html "paquete caret"), ya en Ã©l podemos encontrar una variedad de mÃ©todos de visualizaciÃ³n, preprocesado, ajustes de parÃ¡metros...etc y segmentaciÃ³n de datos. Este extenso paquete incluye mÃ©todos como **createFolds** o **createDataPartition** que nos harÃ¡n de forma fÃ¡cil una divisiÃ³n de los datos en varias particiones o una divisiÃ³n estratificada de los mismos. Por ejemplo, podemos particionar los datos procurando que en ambos conjuntos caiga el mismo nÃºmero de muestras de cada clase (de forma estratificada):

``` r
library(caret)
```

    ## Loading required package: lattice

``` r
muestrasEstratificadas<-unlist(createDataPartition(Orange$Tree,p=0.65))
# asignamos a sus respectivos conjuntos
train<-Orange[muestrasEstratificadas,]
test<-Orange[-muestrasEstratificadas,]
# comprobamos el nÃºmero de muestras de cada dataset
nrow(train)
```

    ## [1] 25

``` r
nrow(test)
```

    ## [1] 10

Si no tienes instalada la librerÃ�a, puedes instalarla con `install.packages('caret')`, aunque tarda un poco porque es grande.

#### Ejercicio 9:

**Calcula el porcentaje de muestras por clase para cada conjunto de train y test que hemos creado, usando el dataset *Orange*. (Puedes usar la funciÃ³n summary para obtener el nÃºmero de muestras de cada clase, con summary(dataset$variable_clase)) Â¿QuÃ© diferencia hay entre las proporciones obtenidas con los dos primeros mÃ©todos y la particiÃ³n estratificada?**
