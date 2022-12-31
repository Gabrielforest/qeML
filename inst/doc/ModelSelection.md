
#  Clearing the Confusion: Overfitting and Feature Selection

*It can be said that the ability to avoid overfitting is what 
separates professionals from amateurs in ML.* -- 
Prof. Yaser Abu-Mostafa of Caltech

Plug the term *overfitting* into Google gave me 6,560,000 results!  And,
no matter what claims you might here, this is still an unsolved problem.
There is no foolproof way to choose the best model.  

## Definition

We will take as a definition that if Model 1 predicts less well than the
"similar" one, Model 2, we say the latter is overfit.  This may be
because Model 2 used too many predictors/features, or because Model 1
used a better combination of hyperparameters.  

In this document, we focus on the effects of having more or fewer
features in our model.  However, many of the points made here also apply
to selection of the best hyperparameter combination in nonparametric
models, e.g. minimum node size in random forests.

## The Bias-Variance Tradeoff

*Ceteris paribus*, the richer the model, the lesser the model bias but
the greater the sampling variance.  Typically, if one starts with a very
simple model and moves toward complexity, at first bias decreases
sharply, overcoming the increase in variance.  Later, the increase in
variance overpowers the by-now small reduction in bias.

Classically, this results in a U-shaped graph of mean prediction error
vs. number of features.  However, in recent years there has been
interesting work on a phenonemon called *double descent*, which involves
deliberate overfitting past "perfect fit" to the training set.
[Here](https://matloff.wordpress.com/2020/11/11/the-notion-of-double-descent/)
is an explanation and example.

## Cross-validation

The best general way to do model selection is to try many models on our
training data, then use each one to predict the holdout data.  Whichever
model does the latter best, we choose that model.  Generally we look at
many holdout sets, and computer the average prediction for each model
over the various holdout sets.

However, cross-validation is not foolproof either.  If we look at enough
combinations of hyperparameters, one or more of them are likely to be 
accidents, unrealistically good.  This is a *p-hacking* problem.

One can try to ameliorate that problem--in **qeML**, the **qeFT()**
function attempts this via Bonferroni confidence intervals--but again,
there is no perfect solution.

## Examples for parametric models (linear/generalized linear)

Again, our context here will be choosing features/predictors.  What can
be done with cross-validation in this regard?

First, note that it would generally be infeasible to look at all
2<sup>p</sup> subsets, where p is the number of predictors.  So, we may
try to limit our search to some singly-indexed quantity.  Here are a
few:

- The LASSO:  This methd is popular among many analysts because it
  automatically chooses a subset of the features.  By varying lambda,
  we obtain richer (small &lambda;) or simpler (large &lambda;) models.
  The **glmnet** package, wrapped by **qeLASSO()**, does this.

- PCA:  Predict Y from the first principal component, then from the
  first two, then the first three and so on.  Choose the set that
  predicts the holdout the best.  The **qePCA()** function alleviates
  you of most of the work.

- polynomial models:  Predict Y from a degree-1 model, then degree-2,
  then degree-3 and so on.  Choose the degree that predicts the holdout
  the best.
 
- p-values:  It has been widely recognized by many of us for years
  that p-values are seldom if ever helpful and often misleading.  In the last 
  few years, the American Statistical Association has begun to make 
  that view official, though it is still a controversy.  See
  [this writeup in my **regtools** package](https://github.com/matloff/regtools/blob/master/inst/NoPVals.md).

  It is classic practice for many analysts to fit the full model, with all
  p predictors, but then retain only the ones reported as "significant."
  This for instance is available by calling **summary()** to the object
  returned by **lm()**/**qeLin()**.

  As a longtime critic of p-values in general, I deplore this practice.
  Nevertheless, they are "going in the right direction," in the sense
  that the smaller the p-value for the estimated &beta;<sub>i</sub>,
  the more useful the corresponging predictor may be in prediction.

  The problem is that ,**the classical 0.05 cutoff is not telling you
  this.**  So, one might try a range of p-values.  One would first set
  the threshhold at the smallest p-value, thus setting a 1-predictor
  model, and test it on the holdout data.  Then one would set the
  threshhold at the second-smallest p-value, thus a 2-predictor model,
  then the third-smallest and so on.  Finally one would choose the model
  with the best performance on the holdout set.

  A variant would be to refit the model each time a predictor is added,
  but otherwise use the same approach.  This is essentially *stepwise
  regression*.  This used to be very popular, but now has fallen out of
  favor.  It's not my favorite by any means, but again, it would be
  usable as long as many different p-value threshholds are tried on a
  holdout set.

## A personal favorite

A new method (that I have a slight connection to) is
[FOCI](https://matloff.wordpress.com/2020/05/18/foci-a-new-method-for-feature-selection/).  It's easy to install and use; give it a try!

