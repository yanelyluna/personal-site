---
title: Distribuciones de pérdida
author: ''
date: '2021-05-27'
slug: []
categories: []
tags: []
subtitle: ''
excerpt: 'Estimación de parámetros.'
images: ~
series: ~
layout: single
---



En el modelo de riesgo colectivo debemos especificar dos distribuciones de probabilidad: una para el número de reclamaciones (frecuencia) y otra para el monto de las reclamaciones (severidad). En este documento se presentan algunas distribuciones, conocidas en este contexto como **distribuciones de pérdida**, sus propiedades y sugerencias sobre cómo y cuándo usarlas.

## Para comenzar: Variables aleatorias en `R`

La paquetería `stats`de `R` (que ya viene precargada al iniciar sesión) proporciona cuatro funciones básicas para cada distribución de variables aleatorias. Dichas funciones comienzan con `d`, `p`, `q` o `r`; el resto del nombre depende de la distribución con la que queremos trabajar. Por ejemplo, para la distribución Poisson usamos el sufijo `pois`:

-   `dpois(x,lambda)` Calcula el valor de la función de densidad Poisson con parámetro `lambda` evaluada en `x`.

-   `ppois(x,lambda)` Calcula el valor de la función de distribución Poisson con parámetro `lambda` evaluada en `x`, es decir `\(P(X\leq x), X\sim Poisson(\lambda)\)`.

-   `qpois(p,lambda)` Calcula el cuantil `p` de la distribución Poisson con parámetro `lambda`, es decir, calcula `x` como el número más pequeño tal que `\(P(X\leq x) \geq p\)`.

-   `rpois(n,lambda)` Genera `n` observaciones de una v.a. de una distribución Poisson con parámetro `lambda`.

El número de parámetros de reciben las funciones depende de los parámetros de cada distribución, pero en general siguen la estructura anterior.

Para las distribuciones presentadas en este documento:

|                   |                                                                         |
|-------------------|-------------------------------------------------------------------------|
| Distribución      | Sufijo                                                                  |
| Poisson           | `pois`                                                                  |
| Binomial Negativa | `nbinom`                                                                |
| Binomial          | `binom`                                                                 |
| Exponencial       | `exp`                                                                   |
| Gamma             | `gamma`                                                                 |
| Pareto            | `pareto` (no se encuentra en la paquetería `stats` pero sí en `actuar`) |
| Lognormal         | `lnorm`                                                                 |

: Sufijos para distribuciones en `R`

## Distribuciones para la frecuencia

Cuando queremos modelar el número de reclamaciones, debemos considerar distribuciones de probabilidad para variables que tomen valores en los enteros no negativos, entre otras características propias del evento de interés. En esta sección se proponen dos familias paramétricas populares para la frecuencia de reclamaciones.

### Poisson

Si `\(Y \sim Poisson(\lambda)\)` ($\lambda > 0$), `$$f_Y(y) = \frac{\lambda^y}{y!}e^{-\lambda}$$` `\(y=0,1,2,...\)`

**Propiedades:**

-   `\(E(Y) = \lambda\)`

-   `\(Var(Y) = \lambda\)`

-   `\(M_Y(t) = e^{\lambda(e^t - 1)}\)`

-   Para una muestra de v.a.i.i.d. `\(Y_1,Y_2,...,Y_n\)` con distribución Poisson($\lambda$), el estimador máximo verosímil de `\(\lambda\)` es `\(\hat{\lambda} = \overline{Y}\)`.

Es importante notar que en esta distribución, la esperanza es igual a la varianza ($\lambda$) por lo que es una buena opción cuando la muestra tiene una media y varianza observada muy cercanas (el cociente de la media entre la varianza es cercano a 1).

<div class="figure" style="text-align: center">
<img src="{{< blogdown/postref >}}index_files/figure-html/pois-1.png" alt="Funciones de densidad Poisson para distintos valores de lambda." width="672" />
<p class="caption">Figure 1: Funciones de densidad Poisson para distintos valores de lambda.</p>
</div>

Podemos simular varias muestras Poisson (cada una de tamaño `\(n=\)` 300) con distintos parámetros:

<div class="figure" style="text-align: center">
<img src="{{< blogdown/postref >}}index_files/figure-html/m_pois-1.png" alt="La línea roja en cada gráfica muestra el valor estimado de lambda." width="672" />
<p class="caption">(\#fig:m_pois)La línea roja en cada gráfica muestra el valor estimado de lambda.</p>
</div>

Parámetros estimados:


| lambda | lambda estimada (media) |
|:------:|:-----------------------:|
|   4    |        4.010000         |
|   6    |        5.976667         |
|   8    |        7.986667         |

**Ejemplo 1**

Estimación de `\(\lambda\)` usando los datos de [`ClaimsLong`](https://teoria-del-riesgo.netlify.app/posts/2021-03-18-datasets/#claimslong-claims-longitudinal):


```r
#Cargamos los datos
library(insuranceData)
data("ClaimsLong")

#Numero de reclamaciones por póliza durante el primer año.
Y1 <- ClaimsLong$numclaims[ClaimsLong$period==1]

#Función de distribución empírica 
F_emp <- ecdf(Y1)

#Lambda estimada
(lambda1 <- mean(Y1))
```

```
## [1] 0.21525
```



**Observación**: La media del número de reclamaciones ($\hat{\lambda}$) es 0.21525 mientras que la varianza observada es 0.6688842, por lo que una ajustar una distribución Poisson no es lo más recomendable.

Si `\(Y_i\)` representa el número de reclamaciones de la póliza `\(i\)`, en el ejemplo anterior estariamos suponiendo que cada póliza (observación) tiene exposición 1. Cuando cada póliza tiene un valor de exposición diferente, digamos `\(w_i\)`, debemos tomarlo en cuenta para la estimación de `\(\lambda\)`. En tal caso `\(Y_i \sim Poisson(\lambda\cdot w_i)\)` y el estimador máximo verosímil de `\(\lambda\)` es `$$\hat{\lambda}=\frac{\sum_{i=1}^n Y_i}{\sum_{i=1}^n w_i}$$`

**Ejemplo 2**

Usando el conjunto de datos [`dataCar`](https://teoria-del-riesgo.netlify.app/posts/2021-03-18-datasets/#datacar-data-car) donde cada póliza tiene asociada su exposición, estimamos el valor de `\(\lambda\)`:


```r
#Cargamos los datos
data(dataCar)
#Variables del número de reclamaciones y de exposición
Y <- dataCar$numclaims
W <- dataCar$exposure
#Lambda estimada
(lambda_hat <- sum(Y)/sum(W))
```

```
## [1] 0.1552476
```



### Binomial Negativa

Si `\(Y \sim BinNeg(r,p)\)` entonces `$$f_Y(y) = \binom{r+y-1}{y}p^r(1-p)^y$$` para `\(y=0,1,2,...\)`

**Propiedades;**

-   `\(E(Y) = \frac{r(1-p)}{p}\)`

-   `\(Var(Y) = \frac{r(1-p)}{p^2}\)`

-   `\(E(Y) < Var(Y)\)`

-   `\(M_Y(t) = \left(\frac{p}{1-(1-p)e^t}\right)^r\)`

-   Usando el método de momentos, los estimadores de `\(r\)` y `\(p\)` son: `$$\hat{r}_{mm}= \frac{\overline{Y}}{\hat{\sigma}^2-\overline{Y}},$$` `$$\hat{p}_{mm} = \frac{\overline{Y}}{\hat{\sigma}^2}$$` donde `\(\hat{\sigma}^2 = \frac{1}{n}\sum_{i=1}^n (Y_i-\overline{Y})^2\)`. **NOTA:** En `R` la función `var()` calcula `\(\frac{1}{n-1}\sum_{i=1}^n (Y_i-\overline{Y})^2\)` por lo que obtenemos `\(\hat{\sigma}^2\)` como `var(Y)*(n-1)/n`.

-   Cuando `\(r\)` es conocida, el estimador máximo verosímil de `\(p\)` es `$$\hat{p}_{mv} = \frac{r}{\overline{Y}+r}$$`

<div class="figure" style="text-align: center">
<img src="{{< blogdown/postref >}}index_files/figure-html/binneg-1.png" alt="Funciones de densidad Binomial Negativa con distintos parámetros." width="672" />
<p class="caption">Figure 2: Funciones de densidad Binomial Negativa con distintos parámetros.</p>
</div>

**Ejemplo 3**

Estimamos el valor de los parámetros con los que simulamos la muestra usando el método de momentos y por máxima verosimilitud con el paquete `fitdistrplus`.


```r
n <- 300 #Tamaño de muesta
# Parámetros
r <- 5
p <-0.6

# Muestra simulada con los parámetros elegidos
set.seed(43)
y <- rnbinom(n,size = r,prob = p)

# Estimación de los parámetros con el método de momentos
(r_mm <- (mean(y)^2)/(var(y)*(n-1)/n-mean(y))) 
```

```
## [1] 4.506022
```

```r
(p_mm <- mean(y)/(var(y)*(n-1)/n)) 
```

```
## [1] 0.5682709
```

```r
# Usando la paquetería fitdistrplus
fd_bn <- fitdist(y,distr="nbinom",method = "mle")
fd_bn$estimate
```

```
##     size       mu 
## 4.446835 3.423506
```

La función `fitdist` estima los parámetros de la distribución que maximizan la log-verosimilitud. Sin embargo, para la distribución binomial negativa estima la esperanza de la distribución ($\mu = \frac{r(1-p)}{p}$) por lo que podemos obtener un valor estimado de `\(p\)` como `$$\hat{p}_{mv} = \frac{\hat{r}}{\hat{r}+\mu}$$`


```r
mu <- as.numeric(fd_bn$estimate[2])

#Valor estimado de r por máxima verosimilitud
(r_mv <- as.numeric(fd_bn$estimate[1]))
```

```
## [1] 4.446835
```

```r
#Valor estimado de p por máxima verosimilitud
(p_mv <- r_mv/(r_mv + mu))
```

```
## [1] 0.5650117
```


| Valor de los parámetros| Estimación por momentos| Estimación por MV|
|-----------------------:|-----------------------:|-----------------:|
|                     5.0|               4.5060217|         4.4468348|
|                     0.6|               0.5682709|         0.5650117|

**Observación**: A diferencia de la distribución Poisson, en la distribución Binomial Negativa la esperanza es menor a la varianza, por lo que puede ser una mejor opción para los datos del Ejemplo 1.

**Ejemplo**

Usando los mismos datos del Ejemplo 1.


```r
# Y1 <- Número de reclamaciones por póliza durante el primer año de vigencia
n1 <- length(Y1)
# Parámetros estimados con el método de momentos
(r_est <- mean(Y1)^2)/(var(Y1)*(n1-1)/n1-mean(Y1))
```

```
## [1] 0.1021402
```

```r
(p_est <- mean(Y1)/(var(Y1)*(n1-1)/n1))
```

```
## [1] 0.3218126
```



### Binomial

Si `\(Y \sim Binomial(m,p)\)` entonces `$$f_Y(y) = \binom{m}{y}p^y(1-p)^{m-y}$$` para `\(y=0,1,2,...,n\)`.

**Propiedades:**

-   `\(E(Y) = mp\)`

-   `\(Var(Y) = mp(1-p)\)`

-   `\(Var(Y) < E(Y)\)`

-   `\(M_Y(t) = (1-p+pe^t)^n\)`

-   Para una muestra `\(Y_i\)` de v.a.i.i.d. Binomial$(m,p)$, los estimadores por el método de momentos son `$$\hat{p}_{mm} = 1-\frac{\hat{\sigma}^2}{\overline{Y}}$$` `$$\hat{m}_{mm} = \frac{\overline{Y}^2}{\overline{Y}-\hat{\sigma}^2}$$`

-   Cuando el parámetro `\(m\)` es conocido, el estimador máximo verosímil de `\(p\)` es `$$\hat{p}_{mv}=\frac{\overline{Y}}{m}$$`

<div class="figure" style="text-align: center">
<img src="{{< blogdown/postref >}}index_files/figure-html/binom-1.png" alt="Función de densidad Binomial(m,p) para distintos parámetros m y p." width="672" />
<p class="caption">Figure 3: Función de densidad Binomial(m,p) para distintos parámetros m y p.</p>
</div>

**Ejemplo 4**

Simulamos una muestra y estimamos el parámetro `p` suponiendo que conocemos el valor de `m`.


```r
n <- 300 #Tamaño de muesta
# Parámetros
m <- 5
p <-0.6

# Muestra simulada con los parámetros elegidos
set.seed(43)
y <- rbinom(n,size = m,prob = p)

# Estimación p con el método de momentos
(p_mm <- 1-(var(y)*(n-1)/n)/mean(y)) 
```

```
## [1] 0.5628124
```

```r
# Estimación de p por máxima verosimilitud
(p_mv <- mean(y)/m)
```

```
## [1] 0.5886667
```


| Valor del parámetro| Estimación por momentos| Estimación por MV|
|-------------------:|-----------------------:|-----------------:|
|                 0.6|               0.5628124|         0.5886667|

## Distribuciones para la severidad

### Exponencial

Si `\(Y\sim\)` Expoencial$(\lambda)$ entonces, `$$f_Y(y) = \lambda e^{-\lambda y}$$` con `\(\lambda > 0\)` para `\(y>0\)`.

<div class="figure" style="text-align: center">
<img src="{{< blogdown/postref >}}index_files/figure-html/exp-1.png" alt="Función de densidad Exponencial con distintos parámetros." width="672" />
<p class="caption">Figure 4: Función de densidad Exponencial con distintos parámetros.</p>
</div>

**Propiedades:**

-   `\(E(Y) = \frac{1}{\lambda}\)`

-   `\(Var(Y) = \frac{1}{\lambda^2}\)`

-   `\(M_Y(t) = \frac{\lambda}{\lambda -t}\)` para `\(t<\lambda\)`

-   El estimador por momentos es igual al estimador máximo verosímil de `\(\lambda\)` y es `$$\hat{\lambda}_{mm}=\hat{\lambda}_{mv} = \frac{1}{\overline{Y}}$$`

**Ejemplo 5**


```r
n <- 300 #Tamaño de muesta
# Parámetro
lambda <- 6

# Muestra simulada con los parámetros elegidos
set.seed(47)
y <- rexp(n, rate = lambda)

# Estimación de lambda
(lambda_mv <- 1/mean(y))
```

```
## [1] 6.337856
```


| Valor del parámetro| Estimación|
|-------------------:|----------:|
|                   6|   6.337856|

### Gamma

Si `\(Y\sim\)` Gamma$(\alpha,\lambda)$ entonces, `$$f_Y(y) = \frac{(\lambda y)^{\alpha-1}}{\Gamma(\alpha)}\lambda e^{-\lambda y}$$` con `\(\alpha >0\)` y `\(\lambda > 0\)` para `\(y>0\)`.

**Propiedades:**

-   `\(E(Y) = \frac{\alpha}{\lambda}\)`

-   `\(Var(Y) = \frac{\alpha}{\lambda^2}\)`

-   `\(M_Y(t) = \left(\frac{\lambda}{\lambda -t}\right)^\alpha\)` para `\(t<\lambda\)`

-   Los estimadores por momentos son: `$$\hat{\alpha}_{mm}=\frac{\overline{Y}^2}{\hat{\sigma}^2}$$` `$$\hat{\lambda}_{mm} = \frac{\overline{Y}}{\hat{\sigma}^2}$$`

-   Cuando `\(\alpha\)` es conocida, el estimador máximo verosímil de `\(\lambda\)` es `$$\hat{\lambda}_{mv} = \frac{\alpha}{\overline{Y}}$$`

<div class="figure" style="text-align: center">
<img src="{{< blogdown/postref >}}index_files/figure-html/gamma-1.png" alt="Función de densidad Gamma con distintos parámetros." width="672" />
<p class="caption">Figure 5: Función de densidad Gamma con distintos parámetros.</p>
</div>



**Ejemplo 6**

Estimación de ambos parámetros con el método de momentos y por máxima verosimilitud usando la función `fitdist`. **Nota:** La distribución Gamma en `R` tiene una parametrización diferente en la que recibe el parámetro `rate` = `\(\lambda\)` o `scale`= `\(1/\lambda\)`.


```r
n <- 1000 #Tamaño de muesta

# Parámetros
alpha <- 6
lambda <- 2

# Muestra simulada con los parámetros elegidos
set.seed(72)
y <- rgamma(n = n,shape = alpha, rate = lambda)

# Estimación por momentos
(alpha_mm <- (mean(y)^2)/(var(y)*(n-1)/n))
```

```
## [1] 6.187858
```

```r
(lambda_mm <- mean(y)/(var(y)*(n-1)/n))
```

```
## [1] 2.020456
```

```r
# Estimación por máxima verosimilitud
fd_gm <- fitdist(y,distr="gamma",method = "mle")
fd_gm$estimate
```

```
##    shape     rate 
## 6.030038 1.968853
```


| Valor de los parámetros| Estimación por momentos| Estimación por MV|
|-----------------------:|-----------------------:|-----------------:|
|                       6|                6.187858|          6.030038|
|                       2|                2.020456|          1.968853|

### Pareto

Si `\(Y\sim\)` Pareto$(\alpha,\lambda)$ entonces, `$$f_Y(y) = \frac{\alpha \lambda^\alpha}{(\alpha+y)^{\alpha +1}}$$` con `\(\alpha >0\)` y `\(\lambda > 0\)` para `\(y>0\)`.

**Propiedades:**

-   `\(E(Y) = \frac{\lambda}{\alpha - 1}\)` cuando `\(\alpha > 1\)`.

-   `\(Var(Y) = \frac{\alpha\lambda^2}{(\alpha-1)^2(\alpha-2)}\)` cuando `\(\alpha > 2\)`.

-   Los estimadores por momentos son: `$$\hat{\alpha}_{mm}=\frac{2\hat{\sigma}^2}{\hat{\sigma}^2-\overline{Y}^2}$$` `$$\hat{\lambda}_{mm} = \overline{Y}(\hat{\alpha}_{mm}-1)$$`

<div class="figure" style="text-align: center">
<img src="{{< blogdown/postref >}}index_files/figure-html/pareto-1.png" alt="Función de densidad Pareto con distintos parámetros." width="672" />
<p class="caption">Figure 6: Función de densidad Pareto con distintos parámetros.</p>
</div>

**Ejemplo 7**

Estimación de parámetros (con `\(\alpha >2\)`).


```r
n <- 300 #Tamaño de muesta

# Parámetros
alpha <- 3
lambda <- 5

# Muestra simulada con los parámetros elegidos
set.seed(73)
y <- rpareto(n = n, shape = alpha, scale = lambda)

# Estimación por momentos
(alpha_mm <- (2*var(y)*(n-1)/n)/(var(y)*(n-1)/n - mean(y)^2))
```

```
## [1] 5.703524
```

```r
(lambda_mm <- mean(y)*(alpha_mm-1))
```

```
## [1] 10.9782
```

```r
# Estimación por máxima verosimilitud
fd_prt <- fitdist(y,distr="pareto",method = "mle")
fd_prt$estimate
```

```
##    shape    scale 
## 3.332713 5.580264
```


| Valor de los parámetros| Estimación por momentos| Estimación por MV|
|-----------------------:|-----------------------:|-----------------:|
|                       3|                5.703524|          3.332713|
|                       5|               10.978203|          5.580264|

### Lognormal

Si `\(Y\sim\)` Lognormal$(\mu,\sigma)$ entonces, `$$f_Y(y) = \frac{1}{\sqrt{2\pi \sigma^2}y}e^{-\frac{(log(y) - \mu )^2}{2\sigma^2}}$$` con `\(\mu \in \mathbb{R}\)` y `\(\sigma^2 > 0\)` para `\(y>0\)`.

**Propiedades:**

-   `\(E(Y) = e^{\mu + \frac{\sigma^2}{2}}\)`

-   `\(Var(Y) = e^{2\mu + \sigma^2}\cdot e^{\sigma^2-1}\)`

-   Los estimadores por momentos son: `$$\hat{\mu}_{mm}=log\left(\frac{\overline{Y}^2}{\sqrt{\frac{\sum_{i=1}^n{Y_i^2}}{n}}}\right)$$` `$$\hat{\sigma}_{mm} = log\left(\frac{\frac{1}{n}\sum_{i=1}^n{Y_i^2}}{\overline{Y}^2}\right)$$`

-   Los estimadores por máxima verosimilitud son `$$\hat{\mu}_{mv} = \frac{1}{n}\sum_{i=1}^n log(Y_i)$$` `$$\hat{\sigma}_{mv}^2 = \frac{1}{n}\sum_{i=1}^n (log(Y_i)-\hat{\mu}_{mv})$$`

<div class="figure" style="text-align: center">
<img src="{{< blogdown/postref >}}index_files/figure-html/lognorm-1.png" alt="Función de densidad Lognormal con distintos parámetros." width="672" />
<p class="caption">Figure 7: Función de densidad Lognormal con distintos parámetros.</p>
</div>


```r
n <- 300 #Tamaño de muesta

# Parámetros
mu <- 4
sigma <- 2.5

# Muestra simulada con los parámetros elegidos
set.seed(39)
y <- rlnorm(n,meanlog = mu, sdlog = sigma)

# Estimación por momentos
(mu_mm <- log(mean(y)^2/sqrt(sum(y^2)/n)))
```

```
## [1] 4.962159
```

```r
(sigma_mm <- log((sum(y^2)/n)/mean(y)^2))
```

```
## [1] 2.392263
```

```r
# Estimación por máxima verosimilitud
#Con la fórmula
(mu_mv <- sum(log(y))/n)
```

```
## [1] 3.892119
```

```r
(sigma_mv <- sqrt(sum((log(y)-mu_mv)^2)/n))
```

```
## [1] 2.407174
```

```r
#Usando fitdist
fd_lnorm <- fitdist(y,distr="lnorm",method = "mle")
fd_lnorm$estimate
```

```
##  meanlog    sdlog 
## 3.892119 2.407174
```


| Valor de los parámetros| Estimación por momentos| Estimación por MV|
|-----------------------:|-----------------------:|-----------------:|
|                     4.0|                4.962159|          3.892119|
|                     2.5|                2.392263|          2.407174|
