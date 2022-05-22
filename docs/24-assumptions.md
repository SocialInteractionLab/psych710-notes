# Model assumptions

## Learning goals 

- Review model assumptions.
- Explore how to test for model assumptions. 
- What to do if model assumptions aren't met. 

## Load packages and set plotting theme  


```r
library("knitr")      # for knitting RMarkdown 
library("kableExtra") # for nice RMarkdown tables
library("tidybayes")  # tidying up results from Bayesian models
library("lme4")       # for linear mixed effects models 
library("brms")       # Bayesian regression models with Stan
library("car")        # for bootstrapping regression models 
library("broom")      # for tidy regression results
library("janitor")    # for cleaning up variable names 
library("patchwork")  # for figure panels
library("ggeffects")  # for visualizing estimated marginal means
library("stargazer")  # for latex regression tables 
library("sjPlot")     # for nice RMarkdown regression tables
library("xtable")     # for latex tables
library("ggrepel")    # for smart text annotation in ggplot
library("tidyverse")  # for wrangling, plotting, etc. 
```


```r
theme_set(theme_classic() + #set the theme 
            theme(text = element_text(size = 20))) #set the default text size

# set rstan options
rstan::rstan_options(auto_write = TRUE)
options(mc.cores = parallel::detectCores())
```

## Model assumptions and what to do if they are violated 

"Regression diagnostics are methods for determining whether a fitted regression model adequately represents the data." (p. 385) [@fox2018r]

### Influential data points 

Because linear regression models are fitted by minimizing the squared error between prediction and data, the results can be strongly influenced by outliers. There are a number of ways of checking for outliers. 

#### Leverage: Hat-values 

Data points that are far from the center of the predictor space have potentially greater influence on the results -- these points have *high leverage*. hat-values are a way of characterizing how much influence individual data points have.


```r
df.credit = read_csv("data/credit.csv") %>% 
  clean_names()
```

```
## Warning: Missing column names filled in: 'X1' [1]
```

```r
fit.credit = lm(formula = balance ~ income,
                data = df.credit)

# fit model without the data point of interest 
fit.credit2 = update(fit.credit,
                     data = df.credit %>% 
                       filter(x1 != 324))

res_with_outlier = fit.credit %>% 
  augment() %>% 
  filter(row_number() == 324) %>% 
  pull(.resid)

res_without_outlier = fit.credit2 %>% 
  augment(newdata = df.credit) %>% 
  mutate(.resid = balance - .fitted) %>% 
  filter(row_number() == 324) %>% 
  pull(.resid)

hat1 = 1 - (res_with_outlier/res_without_outlier) %>% 
  round(3)

hat2 = fit.credit %>% 
  augment() %>% 
  filter(row_number() == 324) %>% 
  pull(.hat) %>% 
  round(3)

print(str_c("hat1: ", hat1))
```

```
## [1] "hat1: 0.041"
```

```r
print(str_c("hat2: ", hat2))
```

```
## [1] "hat2: 0.041"
```

Cook's distance is defined as 

$$D_i = \frac{e^2_{Si}}{k + 1} \times \frac{h_i}{1-h_1}$$,

where $e^2_{Si}$ is the squared standardized residual, $k$ is the number of coefficients in the model (excluding the intercept), and $h_i$ is the hat-value for case $i$. 

Let's double check here: 


```r
fit.credit %>% 
  augment() %>% 
  mutate(cook = ((.std.resid^2)/(2 + 1)) * (.hat/(1 - .hat))) %>% 
  select(contains("cook")) %>% 
  head(10)
```

```
## # A tibble: 10 x 2
##        .cooksd        cook
##          <dbl>       <dbl>
##  1 0.000000169 0.000000113
##  2 0.00000706  0.00000471 
##  3 0.00264     0.00176    
##  4 0.00257     0.00171    
##  5 0.000530    0.000353   
##  6 0.00265     0.00177    
##  7 0.000324    0.000216   
##  8 0.000441    0.000294   
##  9 0.0000457   0.0000304  
## 10 0.00529     0.00353
```

Looking good! 


```r
fit.credit %>% 
  augment() %>% 
  ggplot(aes(x = .hat,
             y = .std.resid)) + 
  geom_point() +
  geom_line(aes(y = .cooksd),
            color = "red")
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-5-1.png" width="672" />

##### Toy example  

Generate some data with an outlier. 


```r
set.seed(1)
df.hat = tibble(x = runif(n = 5),
                y = 10 * x + rnorm(n = 5, sd = 2)) %>% 
  bind_rows(tibble(x = 0.7,
                   y = 15)) %>% 
  mutate(index = 1:n())
```

Illustrate the hat-values and cook's distance. 


```r
fit.hat = lm(formula = y ~ x,
             data = df.hat)

fit.hat %>% 
  augment() %>% 
  mutate(index = 1:n()) %>% 
  ggplot(aes(x = .hat,
             y = .std.resid)) + 
  geom_point() +
  geom_line(aes(y = .cooksd),
            color = "red") +
  geom_text(aes(label = index),
            nudge_y = -0.2)
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-7-1.png" width="672" />

Illustrate what the regression line looks like when all points are fit vs. one of the points is excluded. 


```r
ggplot(data = df.hat,
       mapping = aes(x = x,
                     y = y)) + 
  geom_point(size = 2) + 
  geom_smooth(method = "lm",
              se = F,
              color = "blue") + 
  geom_smooth(data = df.hat %>% 
                filter(index != 6),
                method = "lm",
              se = F,
              color = "red")
```

```
## `geom_smooth()` using formula 'y ~ x'
## `geom_smooth()` using formula 'y ~ x'
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-8-1.png" width="672" />

Summary of each observation. 


```r
fit.hat %>% 
  augment() %>% 
  clean_names() %>% 
  kable(digits = 2) %>% 
  kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;"> y </th>
   <th style="text-align:right;"> x </th>
   <th style="text-align:right;"> fitted </th>
   <th style="text-align:right;"> resid </th>
   <th style="text-align:right;"> std_resid </th>
   <th style="text-align:right;"> hat </th>
   <th style="text-align:right;"> sigma </th>
   <th style="text-align:right;"> cooksd </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> 5.20 </td>
   <td style="text-align:right;"> 0.27 </td>
   <td style="text-align:right;"> 3.58 </td>
   <td style="text-align:right;"> 1.62 </td>
   <td style="text-align:right;"> 0.44 </td>
   <td style="text-align:right;"> 0.32 </td>
   <td style="text-align:right;"> 5.00 </td>
   <td style="text-align:right;"> 0.05 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 4.55 </td>
   <td style="text-align:right;"> 0.37 </td>
   <td style="text-align:right;"> 4.67 </td>
   <td style="text-align:right;"> -0.12 </td>
   <td style="text-align:right;"> -0.03 </td>
   <td style="text-align:right;"> 0.21 </td>
   <td style="text-align:right;"> 5.12 </td>
   <td style="text-align:right;"> 0.00 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 2.65 </td>
   <td style="text-align:right;"> 0.57 </td>
   <td style="text-align:right;"> 6.72 </td>
   <td style="text-align:right;"> -4.07 </td>
   <td style="text-align:right;"> -1.01 </td>
   <td style="text-align:right;"> 0.18 </td>
   <td style="text-align:right;"> 4.42 </td>
   <td style="text-align:right;"> 0.11 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 7.22 </td>
   <td style="text-align:right;"> 0.91 </td>
   <td style="text-align:right;"> 10.13 </td>
   <td style="text-align:right;"> -2.91 </td>
   <td style="text-align:right;"> -1.05 </td>
   <td style="text-align:right;"> 0.61 </td>
   <td style="text-align:right;"> 4.37 </td>
   <td style="text-align:right;"> 0.84 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 1.43 </td>
   <td style="text-align:right;"> 0.20 </td>
   <td style="text-align:right;"> 2.93 </td>
   <td style="text-align:right;"> -1.51 </td>
   <td style="text-align:right;"> -0.44 </td>
   <td style="text-align:right;"> 0.41 </td>
   <td style="text-align:right;"> 5.00 </td>
   <td style="text-align:right;"> 0.07 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 15.00 </td>
   <td style="text-align:right;"> 0.70 </td>
   <td style="text-align:right;"> 8.01 </td>
   <td style="text-align:right;"> 6.99 </td>
   <td style="text-align:right;"> 1.84 </td>
   <td style="text-align:right;"> 0.27 </td>
   <td style="text-align:right;"> 1.98 </td>
   <td style="text-align:right;"> 0.63 </td>
  </tr>
</tbody>
</table>

Compute cook's distance 


```r
fit.hat_with = lm(formula = y ~ x,
                  data = df.hat)

fit.hat_without = lm(formula = y ~ x,
                     data = df.hat %>% 
                       filter(index != 6))

residual_without = fit.hat_without %>% 
  augment(newdata = df.hat) %>% 
  clean_names() %>% 
  mutate(resid = y - fitted) %>% 
  filter(row_number() == 6) %>% 
  pull(resid)

residual_with = fit.hat %>% 
  augment() %>% 
  clean_names() %>% 
  filter(row_number() == 6) %>% 
  pull(resid)

hat = 1 - (residual_with/residual_without)
hat
```

```
## [1] 0.270516
```

### Linear and additive 


```r
df.car = mtcars
```


```r
df.car %>% 
  head(6) %>% 
  kable(digits = 2) %>% 
  kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> mpg </th>
   <th style="text-align:right;"> cyl </th>
   <th style="text-align:right;"> disp </th>
   <th style="text-align:right;"> hp </th>
   <th style="text-align:right;"> drat </th>
   <th style="text-align:right;"> wt </th>
   <th style="text-align:right;"> qsec </th>
   <th style="text-align:right;"> vs </th>
   <th style="text-align:right;"> am </th>
   <th style="text-align:right;"> gear </th>
   <th style="text-align:right;"> carb </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Mazda RX4 </td>
   <td style="text-align:right;"> 21.0 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 160 </td>
   <td style="text-align:right;"> 110 </td>
   <td style="text-align:right;"> 3.90 </td>
   <td style="text-align:right;"> 2.62 </td>
   <td style="text-align:right;"> 16.46 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 4 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mazda RX4 Wag </td>
   <td style="text-align:right;"> 21.0 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 160 </td>
   <td style="text-align:right;"> 110 </td>
   <td style="text-align:right;"> 3.90 </td>
   <td style="text-align:right;"> 2.88 </td>
   <td style="text-align:right;"> 17.02 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 4 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Datsun 710 </td>
   <td style="text-align:right;"> 22.8 </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 108 </td>
   <td style="text-align:right;"> 93 </td>
   <td style="text-align:right;"> 3.85 </td>
   <td style="text-align:right;"> 2.32 </td>
   <td style="text-align:right;"> 18.61 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Hornet 4 Drive </td>
   <td style="text-align:right;"> 21.4 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 258 </td>
   <td style="text-align:right;"> 110 </td>
   <td style="text-align:right;"> 3.08 </td>
   <td style="text-align:right;"> 3.21 </td>
   <td style="text-align:right;"> 19.44 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Hornet Sportabout </td>
   <td style="text-align:right;"> 18.7 </td>
   <td style="text-align:right;"> 8 </td>
   <td style="text-align:right;"> 360 </td>
   <td style="text-align:right;"> 175 </td>
   <td style="text-align:right;"> 3.15 </td>
   <td style="text-align:right;"> 3.44 </td>
   <td style="text-align:right;"> 17.02 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 2 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Valiant </td>
   <td style="text-align:right;"> 18.1 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 225 </td>
   <td style="text-align:right;"> 105 </td>
   <td style="text-align:right;"> 2.76 </td>
   <td style="text-align:right;"> 3.46 </td>
   <td style="text-align:right;"> 20.22 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
</tbody>
</table>



```r
fit.car = lm(formula = mpg ~ 1 + hp,
             data = df.car)

ggplot(data = df.car,
       mapping = aes(x = hp,
                     y = mpg)) + 
  geom_smooth(method = "lm") + 
  geom_smooth(color = "red",
              se = F) + 
  geom_point()
```

```
## `geom_smooth()` using formula 'y ~ x'
```

```
## `geom_smooth()` using method = 'loess' and formula 'y ~ x'
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-13-1.png" width="672" />

Residual plot 


```r
fit.car %>% 
  augment() %>% 
  clean_names() %>% 
  ggplot(data = .,
         mapping = aes(x = fitted,
                       y = resid)) + 
  geom_hline(yintercept = 0, 
             linetype = 2) + 
  geom_point() + 
  geom_smooth(color = "red",
              se = F)
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-14-1.png" width="672" />

Include a squared predictor


```r
ggplot(data = df.car,
       mapping = aes(x = hp,
                     y = mpg)) + 
  geom_smooth(method = "lm",
              formula = y ~ 1 + x + I(x^2)) +
  geom_point()
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-15-1.png" width="672" />



```r
fit.car2 = lm(formula = mpg ~ 1 + hp + I(hp^2),
             data = df.car)

fit.car2 %>% 
  augment() %>% 
  clean_names() %>% 
  ggplot(data = .,
         mapping = aes(x = fitted,
                       y = resid)) + 
  geom_hline(yintercept = 0, 
             linetype = 2) + 
  geom_point() + 
  geom_smooth(color = "red",
              se = F)
```

```
## `geom_smooth()` using method = 'loess' and formula 'y ~ x'
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-16-1.png" width="672" />


### Normally distributed residuals

Let's look at the residuals for the credit card model.


```r
fit.credit %>% 
  augment() %>% 
  clean_names() %>% 
  ggplot(data = ., 
       mapping = aes(x = fitted,
                     y = resid)) + 
  geom_point()  
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-17-1.png" width="672" />

This plot helps assess whether there is homogeneity of variance. Overall, the residual plot looks pretty ok. The diagonal points in the bottom left of th plot arise because credit card balance is not an unbounded variable, and some of the people have a credit card balance of 0. 

We can also check whether the residuals are normally distributed by plotting a density of the residuals, and a quantile quantile plot. 


```r
df.plot = fit.credit %>% 
  augment() %>% 
  clean_names()

p1 = ggplot(data = df.plot,
            mapping = aes(x = resid)) + 
  geom_density() +
  labs(title = "Density plot")

p2 = ggplot(data = df.plot,
            mapping = aes(sample = scale(resid))) +
  geom_qq_line() + 
  geom_qq() + 
  labs(title = "QQ plot",
       x = "theoretical",
       y = "standardized residuals")

p1 + p2
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-18-1.png" width="672" />

The residuals aren't really normally distributed. As both the density and the QQ plot show, residuals with low/negative values are more frequent than residuals with high/positive values. 

#### Transforming the outcome variable 

When the residuals aren't normally distributed and/or when the variance is not homogeneous, one option is to transform some of the variables. 

##### Logarithmic transform 


```r
df.un = UN %>% 
  clean_names() %>% 
  drop_na(infant_mortality, ppgdp)

df.un %>% 
  head(5) %>% 
  kable(digits = 2) %>% 
  kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:left;"> region </th>
   <th style="text-align:left;"> group </th>
   <th style="text-align:right;"> fertility </th>
   <th style="text-align:right;"> ppgdp </th>
   <th style="text-align:right;"> life_exp_f </th>
   <th style="text-align:right;"> pct_urban </th>
   <th style="text-align:right;"> infant_mortality </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Afghanistan </td>
   <td style="text-align:left;"> Asia </td>
   <td style="text-align:left;"> other </td>
   <td style="text-align:right;"> 5.97 </td>
   <td style="text-align:right;"> 499.0 </td>
   <td style="text-align:right;"> 49.49 </td>
   <td style="text-align:right;"> 23 </td>
   <td style="text-align:right;"> 124.53 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Albania </td>
   <td style="text-align:left;"> Europe </td>
   <td style="text-align:left;"> other </td>
   <td style="text-align:right;"> 1.52 </td>
   <td style="text-align:right;"> 3677.2 </td>
   <td style="text-align:right;"> 80.40 </td>
   <td style="text-align:right;"> 53 </td>
   <td style="text-align:right;"> 16.56 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Algeria </td>
   <td style="text-align:left;"> Africa </td>
   <td style="text-align:left;"> africa </td>
   <td style="text-align:right;"> 2.14 </td>
   <td style="text-align:right;"> 4473.0 </td>
   <td style="text-align:right;"> 75.00 </td>
   <td style="text-align:right;"> 67 </td>
   <td style="text-align:right;"> 21.46 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Angola </td>
   <td style="text-align:left;"> Africa </td>
   <td style="text-align:left;"> africa </td>
   <td style="text-align:right;"> 5.14 </td>
   <td style="text-align:right;"> 4321.9 </td>
   <td style="text-align:right;"> 53.17 </td>
   <td style="text-align:right;"> 59 </td>
   <td style="text-align:right;"> 96.19 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Argentina </td>
   <td style="text-align:left;"> Latin Amer </td>
   <td style="text-align:left;"> other </td>
   <td style="text-align:right;"> 2.17 </td>
   <td style="text-align:right;"> 9162.1 </td>
   <td style="text-align:right;"> 79.89 </td>
   <td style="text-align:right;"> 93 </td>
   <td style="text-align:right;"> 12.34 </td>
  </tr>
</tbody>
</table>

The linear model fit (blue) versus the "loess" (local regression) fit (red). 


```r
ggplot(data = df.un,
       mapping = aes(x = ppgdp,
                     y = infant_mortality)) + 
  geom_point() + 
  geom_smooth(method = "lm",
              aes(color = "lm"),
              fill = "blue",
              alpha = 0.1) + 
  geom_smooth(aes(color = "loess"),
              fill = "red",
              alpha = 0.1) +
  scale_color_manual(values = c("blue", "red")) +
  theme(legend.title = element_blank(),
        legend.position = c(1, 1),
        legend.justification = c(1, 1)) +
  guides(color = guide_legend(override.aes = list(fill = c("red", "blue")),
                              reverse = T))
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-20-1.png" width="672" />

Densities of the untransformed and log-transformed variables. 


```r
p1 = ggplot(data = df.un,
       mapping = aes(x = infant_mortality)) + 
  geom_density()

# log transformed 
p2 = ggplot(data = df.un,
       mapping = aes(x = log(infant_mortality))) + 
  geom_density()

p3 = ggplot(data = df.un,
       mapping = aes(x = ppgdp)) + 
  geom_density()

# log transformed 
p4 = ggplot(data = df.un,
       mapping = aes(x = log(ppgdp))) + 
  geom_density()

p1 + p2 + p3 + p4 + 
  plot_layout(nrow = 2)
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-21-1.png" width="672" />

Fitting different models with / without transformations. 


```r
fit.mortality1 = lm(formula = infant_mortality ~ ppgdp,
                   data = df.un)

fit.mortality2 = lm(formula = log(infant_mortality) ~ log(ppgdp),
                   data = df.un)

fit.mortality3 = lm(formula = log(infant_mortality) ~ ppgdp,
                   data = df.un)

fit.mortality4 = lm(formula = infant_mortality ~ log(ppgdp),
                   data = df.un)

summary(fit.mortality1)
```

```
## 
## Call:
## lm(formula = infant_mortality ~ ppgdp, data = df.un)
## 
## Residuals:
##    Min     1Q Median     3Q    Max 
## -31.48 -18.65  -8.59  10.86  83.59 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 41.3780016  2.2157454  18.675  < 2e-16 ***
## ppgdp       -0.0008656  0.0001041  -8.312 1.73e-14 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 25.13 on 191 degrees of freedom
## Multiple R-squared:  0.2656,	Adjusted R-squared:  0.2618 
## F-statistic: 69.08 on 1 and 191 DF,  p-value: 1.73e-14
```

```r
summary(fit.mortality2)
```

```
## 
## Call:
## lm(formula = log(infant_mortality) ~ log(ppgdp), data = df.un)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.16789 -0.36738 -0.02351  0.24544  2.43503 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  8.10377    0.21087   38.43   <2e-16 ***
## log(ppgdp)  -0.61680    0.02465  -25.02   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.5281 on 191 degrees of freedom
## Multiple R-squared:  0.7662,	Adjusted R-squared:  0.765 
## F-statistic: 625.9 on 1 and 191 DF,  p-value: < 2.2e-16
```

```r
summary(fit.mortality3)
```

```
## 
## Call:
## lm(formula = log(infant_mortality) ~ ppgdp, data = df.un)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.61611 -0.48094 -0.07858  0.53930  2.17745 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  3.479e+00  6.537e-02   53.23   <2e-16 ***
## ppgdp       -4.595e-05  3.072e-06  -14.96   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7413 on 191 degrees of freedom
## Multiple R-squared:  0.5394,	Adjusted R-squared:  0.537 
## F-statistic: 223.7 on 1 and 191 DF,  p-value: < 2.2e-16
```

```r
summary(fit.mortality4)
```

```
## 
## Call:
## lm(formula = infant_mortality ~ log(ppgdp), data = df.un)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -38.239 -11.609  -2.829   8.122  82.183 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 155.7698     7.2431   21.51   <2e-16 ***
## log(ppgdp)  -14.8617     0.8468  -17.55   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 18.14 on 191 degrees of freedom
## Multiple R-squared:  0.6172,	Adjusted R-squared:  0.6152 
## F-statistic:   308 on 1 and 191 DF,  p-value: < 2.2e-16
```

Diagnostics plots for the model without transformed variables. 


```r
fit.mortality1 %>% 
  plot()
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-23-1.png" width="672" /><img src="24-assumptions_files/figure-html/unnamed-chunk-23-2.png" width="672" /><img src="24-assumptions_files/figure-html/unnamed-chunk-23-3.png" width="672" /><img src="24-assumptions_files/figure-html/unnamed-chunk-23-4.png" width="672" />

Residual plot using ggplot. 


```r
fit.mortality1 %>% 
  augment() %>% 
  clean_names() %>% 
  ggplot(data = .,
         mapping = aes(x = fitted,
                       y = resid)) + 
  geom_hline(yintercept = 0, 
             linetype = 2) + 
  geom_point() + 
  geom_smooth(color = "red",
              se = F)
```

```
## `geom_smooth()` using method = 'loess' and formula 'y ~ x'
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-24-1.png" width="672" />

Diagnostic plots for the log-log transformed model. 


```r
fit.mortality2 %>% 
  plot()
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-25-1.png" width="672" /><img src="24-assumptions_files/figure-html/unnamed-chunk-25-2.png" width="672" /><img src="24-assumptions_files/figure-html/unnamed-chunk-25-3.png" width="672" /><img src="24-assumptions_files/figure-html/unnamed-chunk-25-4.png" width="672" />

Model fit. 


```r
ggplot(data = df.un,
       mapping = aes(x = log(ppgdp),
                     y = log(infant_mortality))) + 
  geom_point() + 
  geom_smooth(method = "lm",
              color = "blue",
              fill = "blue",
              alpha = 0.1)
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-26-1.png" width="672" />

Illustration of the model predictions in the original scale. 


```r
fit.mortality2 %>% 
  ggpredict(terms = "ppgdp")
```

```
## Model has log-transformed response. Back-transforming predictions to original response scale. Standard errors are still on the log-scale.
```

```
## # Predicted values of infant_mortality
## # x = ppgdp
## 
##      x | Predicted |       95% CI
## ---------------------------------
##      0 |       Inf |             
##  15000 |      8.78 | [7.99, 9.65]
##  25000 |      6.41 | [5.73, 7.16]
##  40000 |      4.80 | [4.21, 5.46]
##  55000 |      3.94 | [3.42, 4.54]
##  70000 |      3.40 | [2.92, 3.95]
##  85000 |      3.01 | [2.57, 3.54]
## 110000 |      2.57 | [2.16, 3.05]
```

```r
fit.mortality2 %>% 
  ggpredict(terms = "ppgdp [exp]") %>% 
  plot()
```

```
## Model has log-transformed response. Back-transforming predictions to original response scale. Standard errors are still on the log-scale.
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-27-1.png" width="672" />

Model predictions for models with multiple predictors. 


```r
# with log transforms 
fit.mortality5 = lm(formula = log(infant_mortality) ~ log(ppgdp) + group,
                   data = df.un)

# without log transforms 
fit.mortality6 = lm(formula = infant_mortality ~ ppgdp + group,
                   data = df.un)

p1 = ggpredict(fit.mortality5,
          terms = c("ppgdp [exp]", "group")) %>% 
  plot() + 
  labs(title = "Prediction with log transform") +
  coord_cartesian(xlim = c(0, 20000))

p2 = ggpredict(fit.mortality6,
          terms = c("ppgdp", "group")) %>% 
  plot() + 
  labs(title = "Prediction without log transform") +
  coord_cartesian(xlim = c(0, 20000))

p1 + p2
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-28-1.png" width="672" />


### Non-parametric tests 

#### Mann-Whitney 


```r
df.ttest = tibble(group1 = rnorm(n = 20, mean = 10, sd = 1),
                  group2 = rnorm(n = 20, mean = 8, sd = 3)) %>% 
  pivot_longer(cols = everything()) %>% 
  mutate(participant = 1:n())
```



```r
ggplot(data = df.ttest,
       mapping = aes(x = name,
                     y = value)) + 
  geom_point(alpha = 0.3,
             position = position_jitter(width = 0.1)) + 
  stat_summary(fun.data = "mean_cl_boot")
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-30-1.png" width="672" />


```r
t.test(formula = value ~ name,
       data = df.ttest)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  value by name
## t = 3.3, df = 22.304, p-value = 0.003221
## alternative hypothesis: true difference in means is not equal to 0
## 95 percent confidence interval:
##  0.8300203 3.6319540
## sample estimates:
## mean in group group1 mean in group group2 
##            10.035302             7.804315
```


```r
wilcox.test(formula = value ~ name,
            data = df.ttest)
```

```
## 
## 	Wilcoxon rank sum exact test
## 
## data:  value by name
## W = 287, p-value = 0.01809
## alternative hypothesis: true location shift is not equal to 0
```

### Bootstrapping regressions 

This section is based on this post [here](https://ademos.people.uic.edu/Chapter12.html#4_what_about_a_more_problematic_example). 


```r
# make reproducible
set.seed(1)

n = 250 
df.turkey = tibble(turkey_time = runif(n = n, min = 0, max = 50),
                   nap_time = 500 + turkey_time ^ 2 + rnorm(n, sd = 16))
```

Visualize the data 


```r
ggplot(data = df.turkey, 
       mapping = aes(x = turkey_time,
                     y = nap_time)) + 
  geom_smooth(method = "lm") +
  geom_point()  
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-34-1.png" width="672" />

A simple linear regression doesn't fit the data well (not suprising since we included a squared predictor). 

Let's fit a simple linear model and print out the model summary.  


```r
fit.turkey = lm(formula = nap_time ~ 1 + turkey_time,
                data = df.turkey)

summary(fit.turkey)
```

```
## 
## Call:
## lm(formula = nap_time ~ 1 + turkey_time, data = df.turkey)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -212.82 -146.78  -55.17  125.74  462.52 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  15.4974    23.3827   0.663    0.508    
## turkey_time  51.5746     0.8115  63.557   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 172.4 on 248 degrees of freedom
## Multiple R-squared:  0.9422,	Adjusted R-squared:  0.9419 
## F-statistic:  4039 on 1 and 248 DF,  p-value: < 2.2e-16
```

A regression with a squared predictor would fit well. 


```r
fit.turkey2 = lm(formula = nap_time ~ 1 + I(turkey_time ^ 2),
                data = df.turkey)

summary(fit.turkey2)
```

```
## 
## Call:
## lm(formula = nap_time ~ 1 + I(turkey_time^2), data = df.turkey)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -45.611  -9.911  -0.652  11.137  43.008 
## 
## Coefficients:
##                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)      4.994e+02  1.575e+00   317.0   <2e-16 ***
## I(turkey_time^2) 1.001e+00  1.439e-03   695.3   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 16.23 on 248 degrees of freedom
## Multiple R-squared:  0.9995,	Adjusted R-squared:  0.9995 
## F-statistic: 4.835e+05 on 1 and 248 DF,  p-value: < 2.2e-16
```


```r
fit.turkey2 %>% 
  augment() %>% 
  clean_names() %>% 
  ggplot(data = ., 
       mapping = aes(x = i_turkey_time_2,
                     y = nap_time)) + 
  geom_line(mapping = aes(y = fitted),
            color = "blue") +
  geom_point()  
```

<img src="24-assumptions_files/figure-html/unnamed-chunk-37-1.png" width="672" />

Let's fit a bootstrap regression.


```r
boot.turkey = Boot(fit.turkey)
summary(boot.turkey)
```

```
## 
## Number of bootstrap replications R = 999 
##             original  bootBias bootSE bootMed
## (Intercept)   15.497 -1.130589 29.330  15.080
## turkey_time   51.575  0.023717  1.058  51.625
```

```r
fit.turkey %>% 
  tidy(conf.int = T) %>% 
  kable(digits = 2) %>% 
  kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> term </th>
   <th style="text-align:right;"> estimate </th>
   <th style="text-align:right;"> std.error </th>
   <th style="text-align:right;"> statistic </th>
   <th style="text-align:right;"> p.value </th>
   <th style="text-align:right;"> conf.low </th>
   <th style="text-align:right;"> conf.high </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> (Intercept) </td>
   <td style="text-align:right;"> 15.50 </td>
   <td style="text-align:right;"> 23.38 </td>
   <td style="text-align:right;"> 0.66 </td>
   <td style="text-align:right;"> 0.51 </td>
   <td style="text-align:right;"> -30.56 </td>
   <td style="text-align:right;"> 61.55 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> turkey_time </td>
   <td style="text-align:right;"> 51.57 </td>
   <td style="text-align:right;"> 0.81 </td>
   <td style="text-align:right;"> 63.56 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 49.98 </td>
   <td style="text-align:right;"> 53.17 </td>
  </tr>
</tbody>
</table>

```r
boot.turkey %>% 
  tidy(conf.int = T) %>% 
  kable(digits = 2) %>% 
  kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> term </th>
   <th style="text-align:right;"> statistic </th>
   <th style="text-align:right;"> bias </th>
   <th style="text-align:right;"> std.error </th>
   <th style="text-align:right;"> conf.low </th>
   <th style="text-align:right;"> conf.high </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> (Intercept) </td>
   <td style="text-align:right;"> 15.50 </td>
   <td style="text-align:right;"> -1.13 </td>
   <td style="text-align:right;"> 29.33 </td>
   <td style="text-align:right;"> -47.64 </td>
   <td style="text-align:right;"> 71.64 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> turkey_time </td>
   <td style="text-align:right;"> 51.57 </td>
   <td style="text-align:right;"> 0.02 </td>
   <td style="text-align:right;"> 1.06 </td>
   <td style="text-align:right;"> 49.35 </td>
   <td style="text-align:right;"> 53.61 </td>
  </tr>
</tbody>
</table>

We see that the confidence intervals using the bootstrap method are wider than the ones that use the linear regression model (particularly for the intercept). 

## Additional resources 

- [Assumptions of a linear regression](http://r-statistics.co-Linear-Regression.html)

## Session info 

Information about this R session including which version of R was used, and what packages were loaded.


```r
sessionInfo()
```

```
## R version 4.0.3 (2020-10-10)
## Platform: x86_64-apple-darwin17.0 (64-bit)
## Running under: macOS Catalina 10.15.7
## 
## Matrix products: default
## BLAS:   /Library/Frameworks/R.framework/Versions/4.0/Resources/lib/libRblas.dylib
## LAPACK: /Library/Frameworks/R.framework/Versions/4.0/Resources/lib/libRlapack.dylib
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
##  [1] forcats_0.5.1    stringr_1.4.0    dplyr_1.0.4      purrr_0.3.4     
##  [5] readr_1.4.0      tidyr_1.1.2      tibble_3.0.6     tidyverse_1.3.0 
##  [9] ggrepel_0.9.1    ggplot2_3.3.3    xtable_1.8-4     sjPlot_2.8.7    
## [13] stargazer_5.2.2  ggeffects_1.0.1  patchwork_1.1.1  janitor_2.1.0   
## [17] broom_0.7.3      car_3.0-10       carData_3.0-4    brms_2.14.4     
## [21] Rcpp_1.0.6       lme4_1.1-26      Matrix_1.3-2     tidybayes_2.3.1 
## [25] kableExtra_1.3.1 knitr_1.31      
## 
## loaded via a namespace (and not attached):
##   [1] utf8_1.1.4           tidyselect_1.1.0     htmlwidgets_1.5.3   
##   [4] grid_4.0.3           munsell_0.5.0        codetools_0.2-18    
##   [7] effectsize_0.4.3-1   statmod_1.4.35       DT_0.17             
##  [10] miniUI_0.1.1.1       withr_2.4.1          Brobdingnag_1.2-6   
##  [13] colorspace_2.0-0     highr_0.8            rstudioapi_0.13     
##  [16] stats4_4.0.3         bayesplot_1.8.0      labeling_0.4.2      
##  [19] emmeans_1.5.3        rstan_2.21.1         farver_2.1.0        
##  [22] bridgesampling_1.0-0 coda_0.19-4          vctrs_0.3.6         
##  [25] generics_0.1.0       TH.data_1.0-10       xfun_0.21           
##  [28] R6_2.5.0             markdown_1.1         gamm4_0.2-6         
##  [31] projpred_2.0.2       assertthat_0.2.1     promises_1.1.1      
##  [34] scales_1.1.1         nnet_7.3-15          multcomp_1.4-15     
##  [37] gtable_0.3.0         processx_3.4.5       sandwich_3.0-0      
##  [40] rlang_0.4.10         splines_4.0.3        checkmate_2.0.0     
##  [43] inline_0.3.17        yaml_2.2.1           reshape2_1.4.4      
##  [46] abind_1.4-5          modelr_0.1.8         threejs_0.3.3       
##  [49] crosstalk_1.1.1      backports_1.2.1      httpuv_1.5.5        
##  [52] rsconnect_0.8.16     Hmisc_4.4-2          tools_4.0.3         
##  [55] bookdown_0.21        ellipsis_0.3.1       RColorBrewer_1.1-2  
##  [58] ggridges_0.5.3       plyr_1.8.6           base64enc_0.1-3     
##  [61] ps_1.6.0             prettyunits_1.1.1    rpart_4.1-15        
##  [64] zoo_1.8-8            cluster_2.1.0        haven_2.3.1         
##  [67] fs_1.5.0             magrittr_2.0.1       data.table_1.13.6   
##  [70] ggdist_2.4.0         openxlsx_4.2.3       colourpicker_1.1.0  
##  [73] reprex_1.0.0         mvtnorm_1.1-1        sjmisc_2.8.6        
##  [76] matrixStats_0.57.0   hms_1.0.0            shinyjs_2.0.0       
##  [79] mime_0.10            evaluate_0.14        arrayhelpers_1.1-0  
##  [82] shinystan_2.5.0      jpeg_0.1-8.1         rio_0.5.16          
##  [85] sjstats_0.18.1       readxl_1.3.1         gridExtra_2.3       
##  [88] rstantools_2.1.1     compiler_4.0.3       V8_3.4.0            
##  [91] crayon_1.4.1         minqa_1.2.4          StanHeaders_2.21.0-7
##  [94] htmltools_0.5.1.1    mgcv_1.8-33          later_1.1.0.1       
##  [97] Formula_1.2-4        RcppParallel_5.0.2   lubridate_1.7.9.2   
## [100] DBI_1.1.1            sjlabelled_1.1.7     dbplyr_2.0.0        
## [103] MASS_7.3-53          boot_1.3-26          cli_2.3.0           
## [106] parallel_4.0.3       insight_0.13.1.1     igraph_1.2.6        
## [109] pkgconfig_2.0.3      foreign_0.8-81       xml2_1.3.2          
## [112] svUnit_1.0.3         dygraphs_1.1.1.6     webshot_0.5.2       
## [115] estimability_1.3     rvest_0.3.6          snakecase_0.11.0    
## [118] distributional_0.2.1 callr_3.5.1          digest_0.6.27       
## [121] parameters_0.12.0.1  rmarkdown_2.6        cellranger_1.1.0    
## [124] htmlTable_2.1.0      curl_4.3             shiny_1.6.0         
## [127] gtools_3.8.2         nloptr_1.2.2.2       lifecycle_1.0.0     
## [130] nlme_3.1-151         jsonlite_1.7.2       viridisLite_0.3.0   
## [133] fansi_0.4.2          pillar_1.4.7         lattice_0.20-41     
## [136] loo_2.4.1.9000       fastmap_1.1.0        httr_1.4.2          
## [139] pkgbuild_1.2.0       survival_3.2-7       glue_1.4.2          
## [142] xts_0.12.1           bayestestR_0.8.3.1   zip_2.1.1           
## [145] png_0.1-7            shinythemes_1.2.0    stringi_1.5.3       
## [148] performance_0.7.0.1  latticeExtra_0.6-29
```