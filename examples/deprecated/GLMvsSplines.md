Preparing data and logit

``` r
data <- HR_data
logit <- function(x) exp(x)/(1+exp(x))
```

Random forest for prediction of `left` variable

``` r
HR_rf_model <- randomForest(factor(left)~., data = data, ntree = 100)
```

Preparing model with `DALEX::explain`

``` r
explainer_rf  <- explain(HR_rf_model, data = data, y = data$left,
                         predict_function = function(model, newdata, ...)
                         predict(model, newdata, type = "prob")[,2])
```

Generalized linear model based on raw data

``` r
HR_glm_model <- glm(left~., data = data, family = "binomial")
```

Generalized linear model with transformed predictors

``` r
HR_spline_model <- build_spline_model(explainer_rf, data, "left")
```

What exactly `build_spline_model` do?

1.  Iteration across all predictors:

-   Calculate pdp curve based on built random forest (`explainer_rf`)
-   Approximate pdp with spline (here using `mgcv` package)
-   When variable is factor, ordered or has not enough unique values there is used identity (`I`) instead of spline

1.  Building glm model when each predictor is transformed with above function (identity or spline)

Let's compare two models:

AIC:
====

``` r
HR_glm_model$aic
```

    ## [1] 12887.9

``` r
HR_spline_model$aic
```

    ## [1] 8059.57

Based on AIC "spline" model is much better.

Accuracy:
=========

``` r
data$pred_glm <- round(logit(predict(HR_glm_model, data)))
data$pred_spline <- round(logit(predict(HR_spline_model, data)))
sum(data$pred_glm == data$left) / nrow(data)
```

    ## [1] 0.7923195

``` r
sum(data$pred_spline == data$left) / nrow(data)
```

    ## [1] 0.8951263

We can also see that spline model has higher accuracy.