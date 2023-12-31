GAM example

Now that we understand why GAMs are such a useful tool in our statistical toolbelt lets have a look at a more relevant dataset.
First we need to load the mgcv package.

```{r, eval=TRUE,echo = FALSE}
library(mgcv)
```
If you haven't already you will also need to download the csv file from the repository "GAM_example.csv".
Now you can import this dataset and inspect the data 

```{r, eval=TRUE, echo = FLASE}
data = read.csv("GAM_example.csv",fileEncoding = "UTF-8-BOM")
head(data)
str(data)
```
You will notice that microcosm and treatment are specified as character variables in our dataset. The mgcv package does not recognise character variables and thus we must change these to factor variables.

```{r, eval=TRUE, echo = FLASE}
data$Microcosm = as.factor(data$Microcosm)
data$Treatment = as.factor(data$Treatment)
str(data)
```

Similar to before we will first fit a simple linear model and inspect a plot of this overlaid on the actual data
```{r, eval=TRUE, echo = FLASE}
lm_mod = lm(Y ~ Day, data = data)
termplot(lm_mod, partial.resid = TRUE, se = TRUE)
```

No surprises here, our linear model does a poor job of explaining the relationship between x and y. Lets fit a simple gam model now

```{r, eval=TRUE, echo = FLASE}
gam_mod <- gam(Y ~ s(Day), 
               family = gaussian (), method = "REML", data = data)

```
This is the same as the linear model but Day is fit as a smooth term. 
You will notice I have added a few extra pieces of imformation to our model which weren't included in the introduction.
First "family = gaussian" as we are assuming this data fits a normal distribution, however there are several other families which may be used depending on your data (e.g. count, bionomial, non-normal distribution) 
The second is "method = "REML"," this involves the selection of the best fit smoothing parameter and you can read more about this here (S. N. Wood 2006), however for the purpose of running simple GAMs and our tutorial here we will use REML.

Now lets have a look at the plot. there are many ways to do this first a simple plot using R's default graphics package, the mgcViz package and itsadug package.
It is very importat to visualise GAMs as we will find out later, so it is good to understand the different ways to plot your results
```{r, eval=TRUE, echo = FLASE}
plot(gam_mod, residuals = TRUE, pch = 1)

library(mgcViz)
plot(gam_mod, residuals = TRUE, pch = 1, cex = 1, shade = TRUE, shade.col = "lightblue", 
     pages = 2, all.terms = TRUE, seWithMean = TRUE)

library(itsadug)
plot_smooth(gam_mod, view = "Day", main = "intercept + s(Day)")
```
Although it is much better than our linear model it is far from perfect. You will also notice all the plots look relatively similar but as our model gets more complex you will undertsand why we have various plotting packages



Lets begin by adding Treatment, note we will had this as a "linear" or non smooth term because it is a factor variable
```{r, eval=TRUE, echo = FLASE}
gam_mod1 <- gam(Y ~ s(Day) + Treatment, 
                family = gaussian (), method = "REML", data = data)

plot(gam_mod1, residuals = TRUE, pch = 1, cex = 1, shade = TRUE, shade.col = "lightblue", 
     pages = 2, all.terms = TRUE, seWithMean = TRUE)
```
This plot puts our partial residuals on the plot, smoother of time (shaded ad coloured light blue) and standard errors of the partial effect term combined with the standard errors of the model intercept.
This is an improvement on our previous model but there is still some data  which are not being accounted for. Likely because we still only have one smooth function for the relationship between x and y.


In the next model we will let the smooth function vary by each level of treatment but we will still keep Treatment as an additive variable as this allows the smoothers to vary by intercept for each level of Treatment.
```{r, eval=TRUE, echo = FLASE}
gam_mod2 <- gam(Y ~ s(Day, by = Treatment) + Treatment, 
                family = gaussian (), method = "REML", data = data)

plot(gam_mod2, residuals = TRUE, pch = 1, cex = 1, shade = TRUE, shade.col = "lightblue", 
     pages = 1, all.terms = TRUE, seWithMean = TRUE)
```
Now we have three smooth functions, one for each treatment, modelling the relationship between x an y and our additive variable allowing the treatments to vary by intercept

There is still one variable which we havent discussed Microcosm. Because we sample from each microcosm several times we have temporal pseudo replication this needs to be accounted for in our model and we can do this by adding Microcosm as a random effect. For those with experience in linear mixed effects models you will know that there are several types of random effects random intercepts, random slopes and in gams random smooths as well. When we add a random variable to a GAM it is called a Generalised Additive Mixed Model or GAMM.
In order to help us visualise what is occurring when adding the random effects we will run these models without treatment fitted 

In our first example we will fit microcosm as a random intercept, that is to say we will let the smooth of each microcosm vary by the point at which they intersept the y axis. You will see in the plot that the smooths for each microcosm are exatly the same except for the point at which they intercept the y axis.
You will notice our plot produced with mgcViz isn't very helpful here but plots using itsadug are.
```{r, eval=TRUE, echo = FLASE}
gam_mod3 <- gam(Y ~ s(Day) + s(Microcosm, bs = "re"),
                family = gaussian (), method = "REML", data = data)

plot(gam_mod3, residuals = TRUE, pch = 1, cex = 1, shade = TRUE, shade.col = "lightblue", 
     pages = 1, all.terms = TRUE, seWithMean = TRUE)

# Plot the summed effect of Day (without random effects)
plot_smooth(gam_mod3, view = "Day", rm.ranef = TRUE, main = "intercept + s(Day)")
# Plot each level of the random effect
plot_smooth(gam_mod3, view="Day", plot_all="Microcosm", rm.ranef=F, xlab = "Day", ylab = "Y")
```

Now we will let the slope of microcosm vary but we have a fixed intercept or start point
```{r, eval=TRUE, echo = FLASE}
gam_mod4 <- gam(Y ~ s(Day) + s(Day,Microcosm, bs = "re"),
                family = gaussian (), method = "REML", data = data)

plot_smooth(gam_mod4, view="Day", plot_all="Microcosm", rm.ranef=F, xlab = "Day", ylab = "Y")
```

Now we can let the intercept and slope vary. You'll notice there is almsot no difference between this plot and the plot for model4 as all microcosms started from the same source/body of water and therefore had the same y values to begin with
```{r, eval=TRUE, echo = FLASE}
gam_mod5 <- gam(Y ~ s(Day) + s(Day,Microcosm, bs = "re")+ s(Day,Microcosm, bs = "re"),
                family = gaussian (), method = "REML", data = data)

plot_smooth(gam_mod5, view="Day", plot_all="Microcosm", rm.ranef=F, xlab = "Day", ylab = "Y")
```

Finally we will let each level of the random effect (microcosm) have its own smooth.
```{r, eval=TRUE, echo = FLASE}
gam_mod5 <- gam(Y ~ s(Day) + s(Day,Microcosm, bs = "fs"),
                family = gaussian (), method = "REML", data = data)

plot_smooth(gam_mod5, view="Day", plot_all="Microcosm", rm.ranef=F, xlab = "Day", ylab = "Y")
```

We aren't actually interested in differences between microcosms though... we simply want to account for this in our models. Now that we have visualised the way different random effects influence the fit of our model we can re-run our final model and add in our treatments. If you compare the plot of this model to our first model you will notice we capture alot more of the variation here.
```{r, eval=TRUE, echo = FLASE}
gam_mod6 <- gam(Y ~ s(Day, by = Treatment) + Treatment + s(Day,Microcosm, bs = "fs"),
                family = gaussian (), method = "REML", data = data)
plot_smooth(gam_mod6, view="Day", plot_all="Treatment", rm.ranef=F, xlab = "Day", ylab = "Y")
```

Lets compare this to our actual data or averages for each treatment
```{r, eval=TRUE, echo = FLASE}
avg_Y <- aggregate(Y ~ Treatment + Day, data = data, FUN = mean)

ggplot(data = avg_Y, aes(x = Day, y = Y, color = Treatment)) +
  geom_line() +
  geom_point() +
  labs(title = "Average Y by Treatment",
       x = "Day",
       y = "Average Y")
```

It looks like our gam may be modelling some noise, see the control after day 5. Lets check our model and some of the assumptions of GAMs/GAMMS, we can do this using gam.check
```{r, eval=TRUE, echo = FLASE}
par(mfrow = c(2, 2))
gam.check(gam_mod6)
```
Looking at the plots you'll notice a few things, our model struggles to fit assumptions of gaussian data, mainly a normal dsitribution (top right plot). The other plots all look good
In the text you'll see significant p values, in this space small p values indicate non-random distribution and suggests the model is not using enough basis functions. Basis functions are the functions which make up our smooth terms, to many and you will overfit the model and include noise, to little and you get a linear model.
There are two things that can help with this the first is transforming your data to improve the fit and the second is specifying the number of basis functions or knots. In our case we should have as many knots as we have days/measurements. Knots are specified using "k=" in your smooth term for x
```{r, eval=TRUE, echo = FLASE}
gam_mod7 <- gam((Y) ~ s(Day, by = Treatment, k =12) + Treatment + s(Day,Microcosm, bs = "fs"),
                family = gaussian (), method = "REML", data = data)

par(mfrow = c(2, 2))
gam.check(gam_mod7)
```
Looking again at our gam.check you will see the fit is relatively good for all except the top left. You may chose to transform this data however for this example you'll notice no significant improvement in the model fit, so we will leave it as is. The other option is to change the "family" argument in your model to something that is better equipped to handle non-gaussian data e.g. "scat".


We must also check for concurvity. This checks to see if one of our smooth terms is the same as another smooth term. This is similar to colinearity in linear models.
Ideally we want values less than 0.8 in the worst row, however this doesn't suit our model. So we will need to look at full=FALSE to look at returns matrices of pairwise concurvities. These show the degree to which each variable is predetermined by each other variable, rather than all the other variables.
You will notice that our our random effect was causing issues with the previous model but you can see when specifying full=FALSE our assumptions for each treatment is met
```{r, eval=TRUE, echo = FLASE}
concurvity(gam_mod6, full = TRUE)
concurvity(gam_mod6, full = FALSE)
```

Finally we can have a look at the model summary now
```{r, eval=TRUE, echo = FLASE}
summary(gam_mod6)
```

There is a lot to look at here but essentially the parametric coefficients explain our linear terms, in our case the additive term Treatment. But the Approximate significance of smooth terms is what we are really interested. edf is the effective degrees of freedom with 1 = straight line and the higher the number the more wiggly the smooth function is ref.df and f are test statistics used in anova but these are only approximate and it is best to check visually for significance. Finally our p value is showing statistical significance of each term, however this is approximate only and it is recomended to a) visually check this and b) compare several models via AIC values to establish the significance of variables


There are four main scenarios we would like to assess with our significance testing
1. the treatment had no significant effect on the dependent variable
2. the treatment had an effect on the timing of changes to the dependent variable
3. the treatment had an effect on the absolute values of the dependent variable
4. the treatment had an effect on the timing and absolute values of the dependent variable.

In order to assess these we need to vary how the independent variable is entered into our model this was discussed above but we will go through it again here;

1. gam_1 assumes Treatment is not significant and therefore we don't need to include it in the model
2. gam_2 assumes the Treatment has a significant influence on the timing of the dependent variable, thus we let Day vary by Treatment
3. gam_3 assumes the Treatment has a significant influence on the absolute value of the dependent variable, thus we add treatment as an additive variable
4. gam_4 assumes the Treatment has a significant influence on the absolute value and timing of the dependent variable, thus we add treatment as an additive variable and let day vary by treatment

```{r, eval=TRUE, echo = FLASE}
gam_1 <- gam((Y) ~ s(Day, k =12) + s(Day,Microcosm, bs = "fs"),
                family = gaussian (), method = "REML", data = data)
par(mfrow = c(1, 1))
plot(gam_1, residuals = TRUE, pch = 1, cex = 1, shade = TRUE, shade.col = "lightblue", 
     pages = 1, all.terms = TRUE, seWithMean = TRUE)
plot_smooth(gam_1, view="Day", plot_all="Treatment", rm.ranef=F, xlab = "Day", ylab = "Y")
# you will notice an error here saying that the two smooth terms are essentially repeating the same information or are highly correlated. this is okay as we know this is likely the case

gam_2 <- gam((Y) ~ s(Day, by = Treatment, k =12) + s(Day,Microcosm, bs = "fs"),
             family = gaussian (), method = "REML", data = data)
plot(gam_2, residuals = TRUE, pch = 1, cex = 1, shade = TRUE, shade.col = "lightblue", 
     pages = 1, all.terms = TRUE, seWithMean = TRUE)
plot_smooth(gam_2, view="Day", plot_all="Treatment", rm.ranef=F, xlab = "Day", ylab = "Y")

gam_3 <- gam((Y) ~ s(Day, k =12) + s(Day,Microcosm, bs = "fs") + Treatment,
             family = gaussian (), method = "REML", data = data)
plot(gam_3, residuals = TRUE, pch = 1, cex = 1, shade = TRUE, shade.col = "lightblue", 
     pages = 1, all.terms = TRUE, seWithMean = TRUE)
plot_smooth(gam_3, view="Day", plot_all="Treatment", rm.ranef=F, xlab = "Day", ylab = "Y")

gam_4 <- gam((Y) ~ s(Day,by =Treatment, k =12) + s(Day,Microcosm, bs = "fs") + Treatment,
             family = gaussian (), method = "REML", data = data)
plot(gam_4, residuals = TRUE, pch = 1, cex = 1, shade = TRUE, shade.col = "lightblue", 
     pages = 1, all.terms = TRUE, seWithMean = TRUE)
plot_smooth(gam_4, view="Day", plot_all="Treatment", rm.ranef=F, xlab = "Day", ylab = "Y")
```

Now we can compare our models using AIC and R square values.
```{r, eval=TRUE, echo = FLASE}
# Create a data frame to store the results
model_results <- data.frame(Model = character(),
                            AIC = numeric(),
                            R_squared = numeric(),
                            stringsAsFactors = FALSE)

# Function to extract AIC and R-squared values from a model
extract_model_info <- function(model) {
  AIC_value <- round(AIC(model),3)
  R_squared <- round(summary(model)$r.sq, 3)
  return(c(AIC = AIC_value, R_squared = R_squared))
}

# Apply the function to each model and populate the data frame
model_results[1, ] <- c("gam_1", extract_model_info(gam_1))
model_results[2, ] <- c("gam_2", extract_model_info(gam_2))
model_results[3, ] <- c("gam_3", extract_model_info(gam_3))
model_results[4, ] <- c("gam_4", extract_model_info(gam_4))

# Print the table
print(model_results)
```

You will notice that gam_2 has the lowest AIC value but has the same R squared value as gam_4. Furthermore the difference in AIC values is < 2 which is a general threshold for determining significant differences between models. Thus you could safely pick either model 2 or 4 in this case, but first we should inspect the plots
```{r, eval=TRUE, echo = FLASE}
par(mfrow = c(1, 1))
plot_smooth(gam_2, view="Day", plot_all="Treatment", rm.ranef=F, xlab = "Day", ylab = "Y")
plot_smooth(gam_4, view="Day", plot_all="Treatment", rm.ranef=F, xlab = "Day", ylab = "Y")
ggplot(data = avg_Y, aes(x = Day, y = Y, color = Treatment)) +
  geom_line() +
  geom_point() +
  labs(title = "Average Y by Treatment",
       x = "Day",
       y = "Average Y")
```

There is an obvious limitation to our model comparison which becomes aparent when visualising the data. You will notice that gam_2 underestimates the difference in the slope of the relationship between Y and Days, particularly for the equilibrated treatment. This data is an example of dissolved inorganic nutrient data thus we would expect there to be no difference in y (the amounts/concentrations) between treatments. When we comapre the start and end values given for "y" this is true. However our model comparison shows that although this is true gam_4 provides a better fit to the actual data. This is because in gam_2 the exclusion of treatment as an additive effect forces the treatments to have identical absolute values over the experimental treatment which alters the fit of the smoother, in particular the start and end value, contorting our data. Therefore under this scenario we should go with gam_4. It is important to note that this is not always the case e.g. Chla can vary by absolute values between treatments as to can abundance etc. However if you have a good understanding of the parameter and follow this tutorial you should be able to appropriately select the best model and infer significance from this and visual inspection of the model.


