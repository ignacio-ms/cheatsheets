# Cheatsheet

## Reading and writing {#read-write}

Files

```Python
x <- read.csv("file.csv")
x <- read.table("file.txt", header=TRUE, sep="\t")

summary(x)
head(x)

write.table(x, "file.txt", sep="\t", row.names=FALSE)
```

Sessions

```Python
save.image("file.RData")
load("file.RData")
```

Remove variables

```Python
rm(list = ls())
```

## Data types {#data-types}

```Python
x <- c(1, 2, 3, 4, 5)  # Vector (All elements must be of the same type)

x <- c(juan = 1, pedro = 2, maria = 3)  # Named vector (All elements must be of the same type)

x <- factor(c('male', 'male', 'female))  
# male male female
# Levels: male female

x <- array(50 * 1000, dim = c(50, 1000))  # Array (All elements must be of the same type)

x <- matrix(50 * 1000, ncols = 50)  # Matrix (All elements must be of the same type)

x <- list(
  one.vector = 1:10,
  hello = 'world',
  one.matrix = matrix(rnorm(20), ncol = 5),
  another.list = list(
    a = 5,
    b = factor(c('male', 'female', 'female'))
  )
) # List (Elements can be of different types)

x <- data.frame(
    y = runif(8),
    Drug = rep(c("DrugA", "DrugB"), 4),
    Age = rep(c("YOUNG", "YOUNG", "OLD", "OLD"), 2)
)  # Data frame (Elements can be of different types)
```

Functions for creating vectors

```Python
x <- 1:10  # 1 2 3 4 5 6 7 8 9 10
x <- seq(1, 10, by = 2)  # 1 3 5 7 9
x <- seq(1, 10, length = 5)  # 1 3.25 5.5 7.75 10
x <- rep(1:3, 2)  # 1 2 3 1 2 3
x <- x * 2 # 2 4 6 2 4 6
```

Matrixes 

```Python
A <- matrix(1:10, nrow = 5)
B <- matrix(11:20, nrow = 5)
cbind(A, B)
## [,1] [,2] [,3] [,4]
## [1,] 1 6 11 16
## [2,] 2 7 12 17
## [3,] 3 8 13 18
## [4,] 4 9 14 19
## [5,] 5 10 15 20
rbind(A, B)
## [,1] [,2]
## [1,] 1 6
## [2,] 2 7
## [3,] 3 8
## [4,] 4 9
## [5,] 5 10
## [6,] 11 16
## [7,] 12 17
## [8,] 13 18
## [9,] 14 19
## [10,] 15 20
```

Logical operators

```Python
x <- 1:5
x > 3  # FALSE FALSE FALSE TRUE TRUE
which(x > 3)  # 4 5 (x[x > 3] -> 4 5)

y <- 2:6
x == y  # FALSE FALSE FALSE FALSE FALSE
identical(x, y)  # FALSE
```


## Scripts {#scripts}

```Python
source("script.R", echo = TRUE, max.deparse.length = 999999)
```

## Functions {#functions}

```Python
my.function <- function(x, y) {
  return(x + y)
}

my.function <- function(x, y) x + y
```

## Plotting {#plotting}

```Python
x <- 1:10
y <- 2 * x + rnorm(length(x))

plot(y ~ x, xlab = 'x label', ylab = 'y label', main = 'plot')
abline(h = 5, lty = 2, col = 'red')
abline(v = 4, col = 'blue')
abline(a = 1, b = 2, col = 'green')
abline(lm(y ~ x), col = 'purple')
```

![plot](Rplot01.png)

```Python
plot(
  c(1, 21), c(1, 2.3),
  type = 'n', axes = FALSE, ann = FALSE
)
points(1:20, rep(1, 20), pch = 1:20)
text(1:20, 1.2, labels = 1:20)
text(11, 1.5, 'pch', cex = 1.3)

points(1:20, rep(2, 20), pch = 16, col = rainbow(20))
text(11, 2.2, 'col', cex = 1.3)
```

![plot](Rplot02.png)

Subplots

```Python
hit <- read.table("hit-table-500-text.txt")

par(mfrow = c(1, 2))
hist(hit[,5], breaks = 50, xlab = '', main = 'Alignment length')
plot(hit[,13] ~ hit[,3], xlab = 'Percent.identity', ylab = 'Bit score')
```

![plot](Rplot03.png)

Basic Linear Regression plots

```Python
load('anage.RData')
summary(anage)
```

![out1.png](out1.png)

```Python
library(car)
scatterplot(Metabolic.rate..W. ~ Body.mass..g., log = 'xy', data = anage)
```

![Rplot04.png](Rplot04.png)

Manual

```Python
anage$logMetRate <- log(anage$Metabolic.rate..W.)
anage$logBodyMass <- log(anage$Body.mass..g.)

birds <- anage[anage$Class == 'Aves', ]
reptiles <- anage[anage$Class == 'Reptilia', ]

lm_birds <- lm(logMetRate ~ logBodyMass, data = birds)
lm_reptiles <- lm(logMetRate ~ logBodyMass, data = reptiles)
```

Linear model `summary`

```Python
lm(formula = logMetRate ~ logBodyMass, data = birds)

Residuals:
Min       1Q   Median       3Q      Max
-1.00686 -0.14349  0.01545  0.16584  0.61638

Coefficients:
Estimate Std. Error t value Pr(>|t|)    
(Intercept) -3.15949    0.04895  -64.55   <2e-16 ***
logBodyMass  0.65037    0.01095   59.38   <2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 0.2452 on 164 degrees of freedom
(1020 observations deleted due to missingness)
Multiple R-squared:  0.9556,	Adjusted R-squared:  0.9553
F-statistic:  3527 on 1 and 164 DF,  p-value: < 2.2e-16
```

Linear model `names`

```Python
[1] "coefficients"  "residuals"     "effects"       "rank"          "fitted.values" "assign"       
[7] "qr"            "df.residual"   "na.action"     "xlevels"       "call"          "terms"        
[13] "model"
``` 

Linear model `coefficients`

```Python
(Intercept) logBodyMass 
 -6.0993295   0.6045932
```

```Python
plot(logMetRate ~ logBodyMass, data = anage, col = c('salmon', 'darkgreen')[Class])
# scatterplot(Metabolic.rate..W. ~ Body.mass..g. | Class, log = 'xy', data = anage)

legend(
  1, 1, legend = levels(anage$Class),
  col = c('salmon', 'darkgreen'), pch = 1
)
abline(lm_birds, col = 'salmon')
abline(lm_reptiles, col = 'darkgreen')
```

![Rplot05.png](Rplot05.png)

### Plotting with `ggplot2` {#ggplot2}

```Python
library(ggplot2)
ggplot(anage, aes(x = logBodyMass, y = logMetRate)) +
  geom_point(aes(color = Class)) +
  geom_smooth(method = 'lm', color = 'salmon', se = FALSE) +
  scale_color_manual(values = c('salmon', 'darkgreen')) +
  theme_bw()
```

![Rplot06.png](Rplot06.png)

Subplots with `ggplot2`

```Python
p_birds <- ggplot(birds, aes(x = logBodyMass, y = logMetRate)) +
  geom_point(aes(color = Class)) +
  geom_smooth(method = 'lm', color = 'salmon', se = FALSE) +
  scale_color_manual(values = 'salmon') +
  theme_bw()

p_reptiles <- ggplot(reptiles, aes(x = logBodyMass, y = logMetRate)) +
  geom_point(aes(color = Class)) +
  geom_smooth(method = 'lm', color = 'darkgreen', se = FALSE) +
  scale_color_manual(values = 'darkgreen') +
  theme_bw()

library(gridExtra)
grid.arrange(p_birds, p_reptiles)
```

![Rplot07.png](Rplot07.png)

## Hypothesis testing {#hypothesis-testing}

We will create an artificial dataset following a normal distribution with mean 0 and standard deviation 1.

```Python
gene_a <- rnorm(50)
round(gene_a, 3)

type <- factor(c(rep('Colon', 30), rep('Lung', 20)))
```

Hypothesis test

H~0~: mean(Colon) - mean(Lung) = 0

H~1~: mean(Colon) - mean(Lung) != 0

> Can we assume normality?

```Python
shapiro.test(gene_a)

#   Shapiro-Wilk normality test
#
# data:  gene_a
# W = 0.97329, p-value = 0.3133
```

As the p-value is greater than 0.05, we can't reject the null hypothesis. We can assume normality.

> Can we assume equal variances?

```Python
var.test(gene_a ~ type, ratio = 1, alternative = 'two.sided')

# 	F test to compare two variances
# 
# data:  gene_a by type
# F = 0.82919, num df = 29, denom df = 19, p-value = 0.6343
# alternative hypothesis: true ratio of variances is not equal to 1
# 95 percent confidence interval:
#  0.345217 1.850153
# sample estimates:
# ratio of variances 
#          0.8291914
```

As the p-value is greater than 0.05, we can't reject the null hypothesis. We can assume equal variances.

```Python
t.test(gene_a ~ type, var.equal = TRUE, alternative = 'two.sided')

# 	Two Sample t-test
# 
# data:  gene_a by type
# t = -0.1017, df = 48, p-value = 0.9194
# alternative hypothesis: true difference in means between group Colon and group Lung is not equal to 0
# 95 percent confidence interval:
#  -0.7377233  0.6666863
# sample estimates:
# mean in group Colon  mean in group Lung 
#          0.05973564          0.09525410
```

As the p-value is greater than 0.05, we can't reject the null hypothesis. 
We can assume that the means are equal.

Let's create a new dataset without a random normal distribution.

```Python
gene_b <- c(rep(-1, 30), rep(2, 20)) + rnorm(50)
```

Hypothesis test

H0: mean(Colon) - mean(Lung) = 0

H1: mean(Colon) - mean(Lung) != 0

> Can we assume normality?

```Python
shapiro.test(gene_b)

# 	Shapiro-Wilk normality test
# 
# data:  gene_b
# W = 0.95749, p-value = 0.06981
```

As the p-value is greater than 0.05, we can't reject the null hypothesis. We can assume normality.

> Can we assume equal variances?

```Python
var.test(gene_b ~ type, ratio = 1, alternative = 'two.sided')

# 	F test to compare two variances
# 
# data:  gene_b by type
# F = 0.77174, num df = 29, denom df = 19, p-value = 0.5169
# alternative hypothesis: true ratio of variances is not equal to 1
# 95 percent confidence interval:
#  0.3212986 1.7219654
# sample estimates:
# ratio of variances 
#          0.7717409
```

As the p-value is greater than 0.05, we can't reject the null hypothesis. We can assume equal variances.

```Python
t.test(gene_b ~ type, var.equal = TRUE, alternative = 'two.sided')

# 	Two Sample t-test
# 
# data:  gene_b by type
# t = -10.086, df = 48, p-value = 1.91e-13
# alternative hypothesis: true difference in means between group Colon and group Lung is not equal to 0
# 95 percent confidence interval:
#  -4.020633 -2.684094
# sample estimates:
# mean in group Colon  mean in group Lung 
#           -1.245847            2.106517
```

As the p-value is less than 0.05, we can reject the null hypothesis.
We can assume that the means are not equal.

```Python
par(mfrow = c(2, 2))
boxplot(gene_a ~ type)
stripchart(gene_a ~ type, vertical = TRUE, pch = 1)
boxplot(gene_b ~ type)
stripchart(gene_b ~ type, vertical = TRUE, pch = 1)
```

![Rplot08.png](Rplot08.png)

`ggplot2` version

```Python
dftt <- data.frame(
  gene_a = gene_a,
  gene_b = gene_b,
  type = type
)

p_genes_a <- ggplot(dftt, aes(x = type, y = gene_a, color = type)) +
  geom_boxplot() +
  geom_jitter(shape = 16, width = 0.1, height = 0) +
  stat_summary(fun = mean, geom = "point", shape = 23, size = 4)

p_genes_b <- ggplot(dftt, aes(x = type, y = gene_b, color = type)) +
  geom_boxplot() +
  geom_jitter(shape = 16, width = 0.1, height = 0) +
  stat_summary(fun = mean, geom = "point", shape = 23, size = 4)

library(gridExtra)
grid.arrange(p_genes_a, p_genes_b)
```

![Rplot09.png](Rplot09.png)

## Tables {#tables}

```Python
table(AB2$Sex)

## F M
## 6 4

with(AB2, table(Sex, status))
## status
## Sex V Z
##   F 3 3
##   M 2 2

xtabs( ~ Sex + status, data = AB2)
## status
## Sex V Z
##   F 3 3
##   M 2 2
```

```Python
x <- data.frame(
    a = c(1,2,2,1,2,2,1),
    b = c(1,2,2,1,1,2,1),
    c = c(1,1,2,1,2,2,1)
)
##   a b c
## 1 1 1 1
## 2 2 2 1
## 3 2 2 2
## 4 1 1 1
## 5 2 1 2
## 6 2 2 2
## 7 1 1 1

table(x)
## , , c = 1
##
##   b
## a   1 2
##   1 3 0
##   2 0 1
##
## , , c = 2
##
##   b
## a   1 2
##   1 0 0
##   2 1 2

xtabs(~ a + b + c, data = x)
## , , c = 1
##
##   b
## a   1 2
##   1 3 0
##   2 0 1
##
## , , c = 2
##
##   b
## a   1 2
##   1 0 0
##   2 1 2
```

With more than two variables we can use flat tables `ftable` 

```Python
ftable(xtabs(~ a + b + c, data = x))
## c 1 2
## a b
## 1 1 3 0
## 2 0 0
## 2 1 0 1
## 2 1 2
## same as ftable(table(x))
```

Summarizing 

```Python
dfx3 <- expand.grid(F1 = c("A", "B"), Age = c(10, 20, 30), Loc = c("k1", "m3"))
dfx3$Vx <- c(0, rnorm(nrow(dfx3)-1))


## F1 Age Loc Vx
## 1 A 10 k1 0.00000000
## 2 B 10 k1 -1.63109321
## 3 A 20 k1 -2.20410435
## 4 B 20 k1 -0.57744297
## 5 A 30 k1 0.33789947
## 6 B 30 k1 -1.63574476
## 7 A 10 m3 -0.36619724
## 8 B 10 m3 0.44451329
## 9 A 20 m3 0.49454397
## 10 B 20 m3 0.09340544
## 11 A 30 m3 1.36691810
## 12 B 30 m3 -0.54316170

ftable(xtabs(Vx ~ ., data = dfx3))
## Loc k1 m3
## F1 Age
## A 10 0.00000000 -0.36619724
## 20 -2.20410435 0.49454397
## 30 0.33789947 1.36691810
## B 10 -1.63109321 0.44451329
## 20 -0.57744297 0.09340544
## 30 -1.63574476 -0.54316170
```

