---
title: Mi librero
author: Yanely Luna
date: "2021-10-28"
slug: []
categories: []
tags: []
subtitle: ''
excerpt: 'Sobre mi gusto por la lectura.'
images: ~
series: ~
layout: single
---




**Última actualización:** 2022-02-25

La actividad que más disfruto hacer durante mi tiempo libre (y de soledad) es leer. Desde temprana edad descubrí mi gusto por la lectura y tenía interés en leer más cosas de lo que proporcionaban los libros en la escuela primaria. Sin embargo, en mi hogar no abundaban libros que pudieran llamar mi atención, además de un libro sobre cuentos populares ([De maravillas y encantamientos](https://www.worldcat.org/title/de-maravillas-y-encantamientos/oclc/651484510) de Marines Medero) que me resultó muy entretenido pero que ahora que lo veo en retrospectiva, probablemente no era muy adecuado para niños, aunque aún así sigue ocupando un lugar especial en mi corazón de lectora.

Con el paso de los años, fui descubriendo más libros que hicieron crecer mi entusiasmo por descubrir nuevas historias y personajes interesantes. Tratando de encontrar una manera de introducir mi creciente amor por la lectura, en este post hago una exploración de los libros que he ido recolectando en mi librero a lo largo de los años. 

## Paqueterías


```r
library(readxl) # Para leer el archivo
library(ggplot2) # Gráficas
library(gridExtra) # También para gráficas
library(dplyr) # Manejar data.frames
library(janitor) # Limpieza de datos
library(knitr) # Presentar tablas
```

## Datos

Hace algún tiempo (en un momento de aburrimiento) me di a la tarea de hacer un inventario de los libros que tengo en mi librero y como resultado recolecté, de forma muy rústica en una hoja de cálculo, datos que me parecieron relevantes sobre estos libros. 


```r
# Cargar el archivo
libros <- read_xlsx("Librero_Inventario.xlsx",sheet = 1)
# Limpiar nombres de las variables
libros <- clean_names(libros)
# Estructura del archivo
str(libros)
```

```
## tibble [147 x 15] (S3: tbl_df/tbl/data.frame)
##  $ isb            : num [1:147] 9.79e+12 9.79e+12 9.79e+12 9.79e+12 9.79e+12 ...
##  $ titulo         : chr [1:147] "Harry Potter y la Piedra Filosofal" "Harry Potter y la Cámara de los Secretos" "Harry Potter y el Prisionero de Azkaban" "Harry Potter y el Cáliz de Fuego" ...
##  $ autor          : chr [1:147] "J.K. Rowling" "J.K. Rowling" "J.K. Rowling" "J.K. Rowling" ...
##  $ editorial      : chr [1:147] "Salamandra" "Salamandra" "Salamandra" "Salamandra" ...
##  $ genero         : chr [1:147] "Fantasía" "Fantasía" "Fantasía" "Fantasía" ...
##  $ ano_publicacion: num [1:147] 1997 1998 1999 2000 2003 ...
##  $ pasta_dura     : num [1:147] 0 0 0 0 0 0 0 1 1 1 ...
##  $ comprado       : num [1:147] 1 1 1 1 1 1 1 1 0 0 ...
##  $ saga           : chr [1:147] "Harry Potter" "Harry Potter" "Harry Potter" "Harry Potter" ...
##  $ ilustrado      : num [1:147] 0 0 0 0 0 0 0 1 0 0 ...
##  $ idioma         : chr [1:147] "Español" "Español" "Español" "Español" ...
##  $ leido_veces    : num [1:147] 2 2 2 1 1 1 1 1 2 2 ...
##  $ paginas        : num [1:147] 254 292 359 635 920 569 637 119 396 487 ...
##  $ rating         : num [1:147] 5 4 5 5 5 5 5 5 5 5 ...
##  $ ano_compra     : num [1:147] 2017 2017 2018 2018 2019 ...
```


## Libros grandotes o pequeñitos

La longitud (número de páginas) de un libro puede ser, para algunos lectores, el motivo por el que decidan leer o no un libro. Yo considero que esto no es algo que influya mucho en mi decisión y que en general mis libros no suelen ser muy pequeños. Para saber si es el caso o no, exploré la distribución del número de páginas por libro.


```r
# Número de páginas promedio por libro
mean(libros$paginas,na.rm=TRUE)
```

```
## [1] 349.1156
```

```r
# Histograma
ggplot(libros, aes(x=paginas)) + geom_histogram(bins = 15, fill=colores[1]) +
  theme_classic() + ggtitle("Número de páginas por libro") +
  xlab("Núm. de páginas")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/pag-1.png" width="672" />

La mayoría de mis libros tienen entre 250 y 400 páginas, mientras hay alguno(s) que tienen casi 1000.



Harry Potter y la Orden del Fénix de J.K. Rowling de la editorial Salamandra es el libro con mayor número de páginas (920) en mi librero.

## Libros viejitos o nuevos




```r
summary(libros$ano_publicacion)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    1603    1972    2007    1973    2015    2021
```

La mitad de mis libros fueron publicados antes del 2007, mientras que una cuarta parte son más recientes, ya que fueron publicados entre el 2015 y el 2021.


```r
ggplot(libros %>% count(ano_publicacion), aes(x=ano_publicacion,y=n)) +
  geom_point(color=colores[3]) +
  geom_line(color=colores[1]) +
  theme_classic() +
  xlab("Año de publicación") +
  ylab("Número de libros")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/a_publicacion-1.png" width="672" />



## ¿Me han gustado?

El sistema que uso para registrar qué tanto me ha gustado un libro es otorgarle entre 1 y 5 estrellas, donde 5 quiere decir que me gustó mucho el libro y 1 que básicamente desearía no haberlo leído.


```r
libros %>% group_by(rating) %>% summarise(n=n()) %>%
ggplot(aes(x=rating,y=n)) + geom_col(fill=colores[7]) + 
  theme_classic() +
  geom_label(aes(label=n),color=colores[7])
```

<img src="{{< blogdown/postref >}}index_files/figure-html/rating-1.png" width="672" />

Por suerte no tengo ningún libro con calificación de 1, y la mayoría realmente me gustaron. Puedo decir que con el tiempo, y mientras más leo he podido definir lo que me gusta y lo que no. Pongo más atención en elegir mis lecturas y en consecuencia la mayoría las disfruto.

La calificación promedio es de 4.22 estrellas.

## ¿Qué tan diverso es mi librero?

En cuanto a géneros, siempre he considerado que cuento con una colección bastante variada, pero para saber si esto es cierto traté de registrar el género de cada libro que tengo. A mi parecer esta es una tarea para nada sencilla, pues ni siquiera tengo bien claro cuáles son realmente los géneros literarios y cuándo un libro pertenece a un género o a otro. Sin embargo, hice lo que creo fue mi mejor aproximación y este fue el resultado.


```r
#Número de libros por género
gen_n <- libros %>% select(genero) %>% group_by(genero) %>% count() %>% arrange(desc(n))

head(gen_n) %>% kable(col.names = c("Género", "Núm. libros"))
```



|Género                | Núm. libros|
|:---------------------|-----------:|
|Fantasía/Juvenil      |          15|
|Ficción/Contemporánea |          13|
|Autoayuda             |          11|
|Fantasía              |           9|
|Ficción               |           9|
|Niños                 |           8|

```r
#Agrupando géneros poco representados
otros <- sum(gen_n$n[11:length(gen_n$genero)])
gen_m <- gen_n[1:11,]
gen_m[11,1] <- "Otros"
gen_m[11,2] <- otros
kable(gen_m, col.names = c("Género", "Núm. libros"))
```



|Género                | Núm. libros|
|:---------------------|-----------:|
|Fantasía/Juvenil      |          15|
|Ficción/Contemporánea |          13|
|Autoayuda             |          11|
|Fantasía              |           9|
|Ficción               |           9|
|Niños                 |           8|
|Clásicos/Detectivesca |           7|
|Clásicos/Romance      |           7|
|Ficción histórica     |           6|
|Autobiográfica        |           4|
|Otros                 |          58|

```r
# Gráfica de géneros
pie(gen_m$n, labels = gen_m$genero)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/gen_n-1.png" width="672" />

En el gráfico anterior se puede notar que hay muchas categorías que tienen pocos libros (`Otros`) pero esta clasificación no dice mucho sobre las tendencias generales en mi librero, por lo que decidí agrupar algunas categorías en otras que considero son más "generales" y podrían dar una mejor idea. 


```r
#Reagrupando los géneros
# Agrupando "Fantasía"
gen <- libros %>% mutate(gen_cat = case_when(
  genero %in% c("Fantasía/Juvenil","Fantasía","Horror/Fantasía") ~ "Fantasía",
# Agrupando "Ficción contemporánea"
  genero %in% c("Ficción","Ficción/Contemporánea") ~ "Ficción Contemporánea",
# Agrupando "Ficción histórica"
  genero %in% c("Ficción histórica","Novela rosa/Ficción histórica",
                "Juvenil/FiccHist") ~ "Ficción histórica",
# Agrupando "No ficción"
  genero %in% c("Autobiográfica","Poesía","No-ficción","Juvenil/No ficción") ~ "No ficción",
# Agrupando "Literatura clásica"
  genero %in% c("Clásicos/Gótica","Clásicos/Aventura","Clásicos/Infantil",
                "Clásicos/Tragedia","Clásicos/Detectivesca","Clásicos/Romance","Clásicos",
                "Clásicos/Histórica","Clásicos/Sátira","Clásicos/Autobigráfica",
                "Clásicos/Distópica","Clásicos/Novela corta") ~ "Literatura Clásica",
# Agrupando "Terror y horror"
  genero %in% c("Horror/Terror psicológico","Horror/Gótica") ~ "Terror y Horror",
# Agrupando "Aventura y misterio"
  genero %in% c("Aventuras","Misterio","Policial","Espionaje",
                "Distópica") ~ "Aventura y Misterio",
# Agrupando "Literatura juvenil"
  genero %in% c("Juvenil/Relatos","Juvenil","Juvenil/Fantasía",
                "Juvenil/No ficción", "Juvenil/Romance") ~ "Literatura Juvenil",
# Agrupando "Literatura infantil"
  genero %in% c("Niños/Misterio","Niños","Niños/Fantasía") ~ "Literatura infantil",
# Agrupando "Otros"
  genero %in% c("Mitología","Relatos cortos","Realismo mágico","No-ficción", NA) ~ "Otros",
# El género de Autoayuda no tuvo modificaciones.
  genero == "Autoayuda" ~ "Autoayuda"))

#Número de libros por género
generos <- gen %>% group_by(gen_cat) %>% count() %>% arrange(desc(n))
head(generos, n=3) %>% kable(col.names = c("Género", "Núm. libros"))
```



|Género                | Núm. libros|
|:---------------------|-----------:|
|Literatura Clásica    |          32|
|Fantasía              |          26|
|Ficción Contemporánea |          22|

```r
# Gráfica de géneros reagrupados
pie(generos$n, labels = generos$gen_cat, col = colores)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/gen_cat-1.png" width="672" />

En el gráfico anterior podemos ver que los libros de literatura clásica, de fantasía y de ficción contemporánea conforman más de la mitad de los libros que tengo. La literatura clásica y la ficción contemporánea son sin duda mis géneros favoritos, pero la fantasía no es un género por el que me incline mucho, por lo que puedo decir que los libros de esta categoría son más aportaciones de mis hermanas y hermano que mías.





Ahora quiero explorar qué tanto me ha gustado cada género.


```r
# Agrupar por rating (solo los libros que he leído)
gen %>% filter(is.na(rating) == F) %>% group_by(gen_cat,rating) %>% count() %>%

# Gráfica de barras por género y rating
ggplot(aes(x=gen_cat, y=n, fill=factor(rating))) + 
  geom_bar(position="stack",stat="identity") + 
  theme_classic() +
  theme(axis.text.x = element_text(angle = 40, hjust = 1)) +
  scale_fill_manual(values=colores[2:6],name="Rating") 
```

<img src="{{< blogdown/postref >}}index_files/figure-html/gen_rat-1.png" width="672" />

Podemos notar que aunque disfruto la mayoría de los libros de literatura clásica que leo (una gran parte tiene 5 estrellas) también he tenido algunas decepciones en ese género pues hay alguno al que le di solo 2 estrellas. Por otra parte, los pocos libros que he leído de terror y/u horror me han gustado mucho (4 y 5 estrellas) por lo que tal vez me lleve una buena sorpresa si exploro más en ese género.

## ¿En qué año compré más libros?


```r
# Agrupar por año de compra
libros%>% count(ano_compra) %>% 
# Gráfica de barras por año
ggplot(aes(x=ano_compra,y=n)) +
  geom_bar(stat = "identity",fill=colores[5]) +
  geom_label(aes(label=n),color=colores[5]) +
  xlab("Año") +
  ggtitle("Número de libros comprados por año") +
  theme_classic()+
  theme(axis.title.y=element_blank(),  #Para remover los márgenes
        axis.text.y=element_blank(),
        axis.ticks.y=element_blank(),
        axis.line.y = element_blank(),
        plot.title = element_text(hjust = 0.5)) +
  scale_x_continuous(breaks=c(2010:2022),label=as.character(2010:2022))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/ano_compra-1.png" width="672" />

Durante el 2020 aumentó considerablemente el número de libros comprados. Puedo decir que el comienzo de la pandemia es la causa de este incremento, debido a que tenía más tiempo libre para leer y mis gastos para entretenimiento fueron todos para adquirir nuevas lecturas. 

## Autores favoritos

```r
autores <- count(libros,autor) %>% arrange(desc(n)) 

head(autores) %>%
kable(col.names = c("Autor/a", "Núm. libros"))
```



|Autor/a            | Núm. libros|
|:------------------|-----------:|
|J.K. Rowling       |           8|
|Arthur Conan Doyle |           7|
|Jane Austen        |           6|
|Becca Fitzpatrick  |           4|
|Stephen King       |           4|
|Suzanne Collins    |           4|

En mi librero encontramos libros de 101 autores diferentes y los tres autores más populares en él son J.K. Rowling, Arthur Conan Doyle y Jane Austen. En lo personal, mi autora favorita es Jane Austen, seguida por Sally Rooney.

## Más datos

<img src="{{< blogdown/postref >}}index_files/figure-html/info_var-1.png" width="672" />

