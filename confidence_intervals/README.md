Interpretation of confidence intervals
======================================

This dumb simulation should help understanding the interpretation of confidence intervals. First,
we simulate *n* experiments independent of each other but generated by the same underlying 
paramaters, *e.g.* **y ~ N(mu, sd)**. As in real life, we imaging that only the first experiment is
actually performed, the other *n-1* are just hypothesised. 

So we ask the question, having performed the (first) experiment and having calcultaed mean and CI, what
does the CI actually means? The answer is that CI tell something about the accuracy of the current 
experiment in estimating the parameters (the *mean* here). CI are not about the estimate of the parameters.

Simulate the data, a matrix where each column is an experiment:

```
true_mu<- 10 ## Parameter unknown in real life and to be estimated

mat<- data.frame(matrix(data= rnorm(mean= true_mu, n= 20 * 100), ncol= 100))

round(mat[1:10, 1:5], 2)
#    X1    X2    X3    X4    X5 ...
# 10.45 10.78 11.92 10.08 10.97
#  8.92 11.73  9.67  9.46  9.35
# 10.48  9.24 10.46  9.17  9.12
# 10.17  9.77  9.78 10.62  9.34
#  9.37 10.06  9.87 11.52  9.99
#  9.46 10.30  9.06 10.38 11.58
# 11.36  8.25  8.86 10.57  8.66
# 10.21 10.11  9.69  8.55  9.49
#  8.89 10.80 10.33 10.84  8.85
#  9.75 10.45  9.66  8.54  9.27
# ...
```

This function calculates for vector *x* the mean and CI with confidence level *alpha*.

```
mean_ci<- function(x, alpha= 0.8){
    mu<- mean(x)
    p<- (1 - alpha)/2 + alpha
    xint<- qt(p, length(x) - 1) * sd(x) / sqrt(length(x))
    ci<- list(mu= mu, low= mu - xint, high= mu +xint)
    return(ci)
}
```

Now we analysed each experiment independently and for each we calculate mean and CI in the same way. 
In real life, only the first experiment is performed:

```
IN<- 0
OUT<- 0
dat<- data.frame(mu= rep(NA, ncol(mat)), low= rep(NA, ncol(mat)), high= rep(NA, ncol(mat)))
for(i in 1:ncol(mat)){
    ci<- mean_ci(mat[, i], 0.8)
    dat[i, ]<- ci
    if(ci$low > true_mu || ci$high < true_mu){
        OUT<- OUT+1
    } else {
        IN<- IN+1
    }
}
png('mean_ci.png', width= 24, height= 12, units= 'cm', res= 360)
par(las= 1, mgp= c(2, 0.5, 0), mar= c(3, 3, 0.5, 0.1))
plot(1:nrow(dat), dat$mu, pch= 19, ylim= range(unlist(dat)), xlab= 'Experiment number', 
    ylab= 'Experiment mean and CI', cex= 0.75)
segments(x0= 1:nrow(dat), y0= dat$low, x1= 1:nrow(dat), y1= dat$high)
segments(x0= 1, y0= dat$low[1], x1= 1, y1= dat$high[1], col= 'blue', lwd= 2)
abline(h= true_mu, lwd= 2, col= 'red')
dev.off()
```

As expected, in the long run *i.e.* if the experiment were to be repeated many times, the confidence
intervals would overlap the true mean in *alpha* % of the cases (here ~80 %):

```
print(c(n= ncol(mat), IN= IN, OUT= OUT, PCT= IN / (IN + OUT)))
#     n     IN    OUT    PCT 
#100.00  79.00  21.00   0.79 
```

This plot shows the mean and CI for each experiment where, as expected, about 80 % of the CI overlaps 
the true mean. Let's say that the first experiment (blue) is the one actually performed and the others
are imaginary, meaning: *if I repeat the blue experiment another 99 times, what would I see?*

<img src=mean_ci.png width="800">

So, having performed the (first) experiment and getting mean 10.4 and CI 10.1-10.7 (at level 80 %)
we can say that repeating this experiment another *n* times, we would hit the true mean ~80 % of the
time with at this level of confidence. 

We cannot say that 10.1-10.7 contains the true mean 80 % of the time. In fact, it doesn't make sense to
say that. As shown in the plot, either the true mean is overlapped or not.
