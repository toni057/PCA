Principal components analysis
-----------------------------

This report aims to show the basic ideas of the principal components analysis (PCA) in a lightweight manner.

### Starting dataset

We start by generating two dimensional data on which we apply PCA. For convenience we will shape the data as two distinct data clouds and plot it.

``` r
library(ggplot2)

set.seed(1000)

n <- 10

x1 <- rnorm(n, 0, 1)
y1 <- rnorm(n, 0, 1)

x2 <- rnorm(n, 2, 1)
y2 <- rnorm(n, 2, 1)

x <- c(x1, x2)
y <- c(y1, y2)

# combine two groups of data to a single data source
data <- cbind.data.frame(group = rep(c("1", "2"), each = n), x = x, y = y)

# plot the data
ggplot(data, aes(x = x, y = y, color = group)) + geom_point(size = 2)
```

<img src="PCA_files/figure-markdown_github/unnamed-chunk-1-1.png" style="display: block; margin: auto;" />

### PCA

A natural step would be to scale (standardize) the data before doing the PCA, but in this case we will omit that step to show the PCA effects better. Scaling is an important step primarily to get all dimensions on the same scale, otherwise the data that is measured in largest units will dominate and the most variance usually lies there. PCA without scaling is usually called covariance PCA, while scaled is called correlation PCA.

``` r
# run the PCA
pca <- princomp(data[, -1], cor = F)

# Minus sign included in the x and y axis is due to arbitrary choice of
# direction of the PC coordinate system, and only affects the
# orientation of the axes - included purely for plotting convenience
pca$loadings[, 1] <- -pca$loadings[, 1]
pca$loadings[, 2] <- -pca$loadings[, 2]
```

The standard pca outputs are the standard deviations (`pca$sdev`) and the rotation (or the loadings) matrix (`pca$loadings`), which provides a link (a rotation) between the original coordinate system and the PC coordinate system.

#### Explained variances

The cumulative variance plot can provide us with a quick insight into their distribution.

``` r
# plot PC variances
pca.var <- data.frame(PC = 0:length(pca$sdev), var = c(NA, pca$sdev^2), 
    cumvar = cumsum(c(0, pca$sdev)), cumvar_scaled = cumsum(c(0, pca$sdev))/sum(c(0, 
        pca$sdev)))

ggplot(pca.var, aes(x = PC, y = cumvar_scaled)) + geom_point(size = 2) + 
    geom_line() + geom_text(aes(label = round(cumvar_scaled, 2)), hjust = 0, 
    vjust = c(-2, 2, 2))
```

<img src="PCA_files/figure-markdown_github/unnamed-chunk-3-1.png" style="display: block; margin: auto;" />

Next we plot the principal components coordinates in the original coordinate system. Principal components are ordered by the size of explained variance and they are always orthogonal.

Notice: PCA is a rotation of the original coordinate system into the PC coordinate system and PC points are the representation of original points in the PC coordinate system.

``` r
ggplot(data, aes(x = x, y = y, color = group)) + geom_point(size = 2) + 
    geom_abline(intercept = 0, slope = pca$loadings[2, 1]/pca$loadings[1, 
        1]) + geom_point(size = 2) + geom_abline(intercept = 0, slope = pca$loadings[2, 
    2]/pca$loadings[1, 2]) + coord_fixed(xlim = c(-2, 3), ylim = c(-1, 
    3))
```

<img src="PCA_files/figure-markdown_github/unnamed-chunk-4-1.png" style="display: block; margin: auto;" />

After establishing the PC coordinate system we can rotate our original dataset into the PC coordinate system. The plot shows both coordinate system to better illustrate the PCA rotation.

``` r
# rotate original data
x.pca <- as.matrix(data[, -1]) %*% (pca$loadings[, 1])
y.pca <- as.matrix(data[, -1]) %*% (pca$loadings[, 2])

# group into a new dataset
data.pca <- cbind.data.frame(dataset = "Original coordinate system", data)
data.pca <- rbind.data.frame(data.pca, data.frame(dataset = "Rotated coordinate system", 
    group = data$group, x = x.pca, y = y.pca))

data.abline.coord <- data.frame(dataset = rep(c("Original coordinate system", 
    "Rotated coordinate system"), each = 2), slope = c(pca$loadings[2, 
    1]/pca$loadings[1, 1], pca$loadings[2, 2]/pca$loadings[1, 2], 0, 1e+10), 
    intercept = rep(0, 4))

ggplot(data.pca) + geom_point(aes(x = x, y = y, color = group), size = 2) + 
    coord_fixed(xlim = c(-2, 4), ylim = c(-2, 3)) + facet_grid(~dataset) + 
    geom_abline(aes(slope = slope, intercept = intercept), data = data.abline.coord)
```

<img src="PCA_files/figure-markdown_github/unnamed-chunk-5-1.png" style="display: block; margin: auto;" />

#### Dimensionality reduction with PCA

Dimensionality reduction with PCA comes down to projecting the data onto the first k principal components axes. In the case the implemented algorithm returns data in the PC space it is just a matter of selecting first k dimensions.

In this example we only take the first PC and the plots show the effect in the original and the PC coordinate systems.

``` r
newx <- (pca$loadings[1, 1]) * (as.matrix(data[, -1]) %*% (pca$loadings[, 
    1]))
newy <- (pca$loadings[2, 1]) * (as.matrix(data[, -1]) %*% (pca$loadings[, 
    1]))

# append rotated data
data.pca <- cbind.data.frame(data.pca, newx = c(newx, x.pca), newy = c(newy, 
    rep(0, 2 * n)))

ggplot(data.pca) + geom_point(aes(x = x, y = y, color = group), size = 2) + 
    coord_fixed(xlim = c(-2, 4), ylim = c(-2, 3)) + facet_grid(~dataset) + 
    geom_abline(aes(slope = slope, intercept = intercept), data = data.abline.coord) + 
    geom_point(aes(x = newx, y = newy, color = group), shape = 7, size = 2) + 
    geom_segment(aes(x = x, y = y, xend = newx, yend = newy, color = group), 
        linetype = 2)
```

<img src="PCA_files/figure-markdown_github/unnamed-chunk-6-1.png" style="display: block; margin: auto;" />

``` r
# check to see whether points in the original and PC coordinate systems
# match
print(sqrt(newx[18]^2 + newy[18]^2))
```

    ## [1] 3.659198

``` r
print(x.pca[18])
```

    ## [1] 3.659198

### Comparison of PCA and least squares

PCA in this setting compellingly reminds of the least squares fitting, and here we aim to show the difference between fitting the least squares line and projecting onto the first principal component from the previous example.

Although the first PC and least squares line are very similar to each other, the way the lines are fitted are conceptually quite different as shown in the plot.

``` r
# calculate the least squares line
least.squares <- lm(y ~ x)

# append least squares line to the firs principal component data
data.abline.ls <- data.frame(dataset = "Least squares", slope = least.squares$coefficients[2], 
    intercept = least.squares$coefficients[1])
data.abline.ls <- rbind.data.frame(data.abline.coord[data.abline.coord$dataset == 
    "Original coordinate system", ], data.abline.ls)

data.ls <- data.pca[data.pca$dataset == "PCA", ]
data.ls <- rbind.data.frame(data.ls, cbind.data.frame(dataset = "Least squares", 
    data, data.frame(newx = x, newy = least.squares$fitted.values)))

ggplot(data.ls) + geom_point(aes(x = x, y = y, color = group), size = 2) + 
    coord_fixed(xlim = c(-2, 4), ylim = c(-2, 3)) + geom_abline(aes(slope = slope, 
    intercept = intercept), data = data.abline.ls) + facet_grid(~dataset) + 
    geom_point(aes(x = newx, y = newy, color = group), shape = 7, size = 2) + 
    geom_segment(aes(x = x, y = y, xend = newx, yend = newy, color = group), 
        linetype = 2)
```

<img src="PCA_files/figure-markdown_github/unnamed-chunk-7-1.png" style="display: block; margin: auto;" />

### Some additional PCA resuts and features

In order to effectively use and interpret the PCA some additional metrics are calculated. The calculations are not going to be presented in detail, but instead used as standard outputs of the *P**C**A* function form the *F**a**c**t**o**M**i**n**e**R* package. Package *f**a**c**t**o**e**x**t**r**a* provides convenience plotting functions.

Here we will use the Motor Trend Car Road Tests (mtcars) dataset, extracted from the 1974 *Motor Trend* US magazine and comprises of fuel consumption and 10 other characteristics of 32 automobiles (1973-74 models).

``` r
library(FactoMineR)
library(factoextra)

data(mtcars)
head(mtcars)
```

    ##                    mpg cyl disp  hp drat    wt  qsec vs am gear carb
    ## Mazda RX4         21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
    ## Mazda RX4 Wag     21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
    ## Datsun 710        22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
    ## Hornet 4 Drive    21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
    ## Hornet Sportabout 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2
    ## Valiant           18.1   6  225 105 2.76 3.460 20.22  1  0    3    1

``` r
pca.fmr <- PCA(mtcars, scale.unit = T, graph = F)
```

#### Biplot

It is very usefull to plot the original axes and individuals in the PC coordinate system to immediately see their relationship. We can also see where variables lie compared to the principal components and how individuals plot against them.

``` r
fviz_pca_biplot(pca.fmr, axes = c(1, 2))
```

<img src="PCA_files/figure-markdown_github/unnamed-chunk-9-1.png" style="display: block; margin: auto;" />

We can immediately see that the first principal component is positively correlated with the size of the engine (hp, cyl, disp) and negatively correlated with mpg (meaning larger engines consume more fuel).

The second PC is more about the the car's acceleration, number of carburators and gears.

Also, by looking at the individual cars and where they are placed in the PC space, we can see that cars like Cadillac Fleetwood or Lincoln Continental are placed directly on PC1 axis. This makes sense, since these are large limousines, with the largest engines (&gt;7l) that consume a lot of fuel and the feature vectors confirm this. Similarly, on the other spectrum of the PC1 axis we can find small and economic cars like Honda Civic and Toyota Corolla

#### Contribution plots

##### Variable contribution

Contributions plot is great to visualize how much each of the variables (here engine specifications) contributes to each of the principal components. We can immediately see that the first principal component is almost entirely made up the size of the engine, while the second principal component is mostly made of the speed and acceleration characteristics. This is in alignment with out previous conclusions.

``` r
library(gridExtra)
grid.arrange(fviz_contrib(pca.fmr, choice = "var", axes = 1), fviz_contrib(pca.fmr, 
    choice = "var", axes = 2), ncol = 2)
```

<img src="PCA_files/figure-markdown_github/unnamed-chunk-10-1.png" style="display: block; margin: auto;" />

##### Observation contribution

It is also beneficial to see how much each of the observations (in this case cars) contributes to each of the principal components. Some major contributors to the first principal component are Honda Civic and Toyota Corola, Lincoln Continental and Cadillac Fleetwood, exactly the cars that define small economic cars and extavagant limousines.

``` r
grid.arrange(fviz_contrib(pca.fmr, choice = "ind", axes = 1), fviz_contrib(pca.fmr, 
    choice = "ind", axes = 2), ncol = 2)
```

<img src="PCA_files/figure-markdown_github/unnamed-chunk-11-1.png" style="display: block; margin: auto;" />

#### Cosine squared metric

##### Variable cosine squared

The *c**o**s*<sup>2</sup> metric shows the angle distance of the variable to the principal component. The closer a variable to a principal components axis is, the better the principal component represents that variable. The direction can be positive or negative.

``` r
grid.arrange(fviz_cos2(pca.fmr, choice = "var", axes = 1), fviz_cos2(pca.fmr, 
    choice = "var", axes = 2), ncol = 2)
```

<img src="PCA_files/figure-markdown_github/unnamed-chunk-12-1.png" style="display: block; margin: auto;" />

In this example the three crime variables have a high *c**o**s*<sup>2</sup> with the first principal component which is in alignment with our previous observations.

##### Observation cosine squared

It is also possible to check the cosine squared distance of each observation (individual) from the principal component.

``` r
grid.arrange(fviz_cos2(pca.fmr, choice = "ind", axes = 1), fviz_cos2(pca.fmr, 
    choice = "ind", axes = 2), ncol = 2)
```

<img src="PCA_files/figure-markdown_github/unnamed-chunk-13-1.png" style="display: block; margin: auto;" />
