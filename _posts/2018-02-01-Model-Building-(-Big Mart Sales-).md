---
published: true
---
## Model Building

We will be building a baseline model which will help us in comparing our future models.
I will be replacing the Item_Outlet_Sales column in test with the average Item_Outlet_Sales of train.

```
mean_sales <- mean(train$Item_Outlet_Sales)
test$Item_Outlet_Sales <- mean_sales
```

The Score with this model is 1861.44276283689. We will be using this baseline for comparing all our models. If the new model gives us a score below this then we will have to probably rebuild that model. The next step would be encode the factor variables in python in order to build a linear regression model in R. We will be using the labele encoder in order to perform encoding for all the factor variables

```
from pandas import *
import numpy as np
from matplotlib import *  

data_train = read_csv('C:/Users/devarsh patel/Desktop/New folder/alg1.csv')
data_test = read_csv('C:/Users/devarsh patel/Desktop/New folder/alg0.csv')

from sklearn.preprocessing import LabelEncoder
le = LabelEncoder()

#New variable for outlet
data_train['Outlet'] = le.fit_transform(data_train['Outlet_Identifier'])
var_mod = ['Item_Fat_Content','Outlet_Location_Type','Outlet_Size','Item_Type_Combined','Outlet_Type','Outlet','category_by_sales','Item_Identifier']
le = LabelEncoder()
for i in var_mod:
    data_train[i] = le.fit_transform(data_train[i])
    

#New variable for outlet
data_test['Outlet'] = le.fit_transform(data_test['Outlet_Identifier'])
var_mod = ['Item_Fat_Content','Outlet_Location_Type','Outlet_Size','Item_Type_Combined','Outlet_Type','Outlet','category_by_sales','Item_Identifier']
le = LabelEncoder()
for i in var_mod:
    data_test[i] = le.fit_transform(data_test[i])

data_test.to_csv('C:/Users/devarsh patel/Desktop/New folder/data_test.csv',header = True)

data_train.to_csv('C:/Users/devarsh patel/Desktop/New folder/data_train.csv',header = True)

```

Now, lets build a multiple Regression model using the encoded variables and try to find out the accuracy of our model. Initially we will include all the variables in our model, we will build a second regression model only using those varaibles that were shown to be significant in Model_Linear_1 .

```
Model_Linear_2 <- lm(Item_Outlet_Sales~ Item_MRP + Outlet_Establishment_Year + Outlet_Size + Outlet_Type + Avg_Item_Outlet_Sales ,data = data_linear_train)
Model_Linear_1 <- lm(Item_Outlet_Sales~ . ,data = data_linear_train)
```
```
summary(Model_Linear_1)
```
```
Call:
lm(formula = Item_Outlet_Sales ~ ., data = data_linear_train)

Residuals:
    Min      1Q  Median      3Q     Max 
-1995.9  -881.9  -138.8   793.9  2145.3 

Coefficients:
                            Estimate Std. Error t value Pr(>|t|)    
(Intercept)                449.56556  367.20483   1.224   0.2209    
Item_Identifier              0.03217    0.03994   0.806   0.4205    
Item_Weight                  1.13339    2.37803   0.477   0.6337    
Item_Fat_Content             3.59085   12.53496   0.286   0.7745    
Item_Visibility           -155.09528  233.14236  -0.665   0.5059    
Item_MRP                     1.69268    0.17753   9.535  < 2e-16 ***
Outlet_Establishment_Year   10.90554    1.50261   7.258 4.28e-13 ***
Outlet_Size                 52.61365   22.13794   2.377   0.0175 *  
Outlet_Location_Type        36.85788   33.12295   1.113   0.2658    
Outlet_Type                 41.47654   22.44069   1.848   0.0646 .  
Item_Type_Combined         -47.89596   34.83480  -1.375   0.1692    
Avg_Item_Outlet_Sales        0.43556    0.25660   1.697   0.0896 .  
category_by_sales           71.98106   41.43996   1.737   0.0824 .  
Outlet                       2.06372    7.91521   0.261   0.7943    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 1015 on 8509 degrees of freedom
Multiple R-squared:  0.01985,	Adjusted R-squared:  0.01836 
F-statistic: 13.26 on 13 and 8509 DF,  p-value: < 2.2e-16

```
```
summary(Model_Linear_2)
```
```
Call:
lm(formula = Item_Outlet_Sales ~ Item_MRP + Outlet_Establishment_Year + 
    Outlet_Size + Outlet_Type + Avg_Item_Outlet_Sales, data = data_linear_train)

Residuals:
    Min      1Q  Median      3Q     Max 
-1998.0  -882.5  -138.8   793.5  2117.4 

Coefficients:
                           Estimate Std. Error t value Pr(>|t|)    
(Intercept)               1.014e+03  1.893e+02   5.353 8.88e-08 ***
Item_MRP                  1.676e+00  1.773e-01   9.451  < 2e-16 ***
Outlet_Establishment_Year 1.024e+01  1.441e+00   7.102 1.33e-12 ***
Outlet_Size               3.206e+01  1.837e+01   1.745    0.081 .  
Outlet_Type               5.742e+01  1.437e+01   3.996 6.50e-05 ***
Avg_Item_Outlet_Sales     6.666e-02  1.405e-01   0.474    0.635    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 1015 on 8517 degrees of freedom
Multiple R-squared:  0.01976,	Adjusted R-squared:  0.01919 
F-statistic: 32.57 on 5 and 8517 DF,  p-value: < 2.2e-16

```

Lets compute the AIC of both the models and try to find out which one is better

```
AIC(Model_Linear_1)
```
```
[1] 142205.9
```
```
AIC(Model_Linear_2)
```
```
[1] 142212.4
```

Even though the second model has higher accuarcy, it doesnt imply that our second model is better. If you would have noticed the beta's for our second model have changed. This happened beacuse the beta's in second model tried to account for the varaince that was initally contributed by the insignificant variables. For this reason, it is always good to include even the insignificant variable while making predictions. Furthermore we cannot rank the variables based on the significane test conducted above because the scale is different for different variables. We would need to standadize the variables in order to rank them. 
The leaderboard score for Model_Linear_1 is 1790.82081009079, better than the baseline.
Our multiple regression model doesnt have high accuracy because we used encoding before building a model this might have resulted in multicollinearity or some kind of dummy varibale trap.

Lets try to correct the model for heteroskedasticity using the sandwhich and lmtest package.

```
library(sandwich) # For White correction 
library(lmtest) # More advanced hypothesis testing tools

```
coeftest(Model_Linear_1,vcov.=vcov) # Old school t test for significance (like summary)
```
t test of coefficients:

                             Estimate  Std. Error t value  Pr(>|t|)    
(Intercept)                449.565561  367.204831  1.2243   0.22088    
Item_Identifier              0.032170    0.039937  0.8055   0.42055    
Item_Weight                  1.133387    2.378034  0.4766   0.63365    
Item_Fat_Content             3.590851   12.534961  0.2865   0.77453    
Item_Visibility           -155.095279  233.142363 -0.6652   0.50592    
Item_MRP                     1.692681    0.177528  9.5347 < 2.2e-16 ***
Outlet_Establishment_Year   10.905541    1.502612  7.2577 4.283e-13 ***
Outlet_Size                 52.613648   22.137941  2.3766   0.01749 *  
Outlet_Location_Type        36.857876   33.122950  1.1128   0.26584    
Outlet_Type                 41.476540   22.440694  1.8483   0.06460 .  
Item_Type_Combined         -47.895962   34.834801 -1.3749   0.16918    
Avg_Item_Outlet_Sales        0.435563    0.256599  1.6974   0.08965 .  
category_by_sales           71.981064   41.439962  1.7370   0.08242 .  
Outlet                       2.063716    7.915210  0.2607   0.79431    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
```
coeftest(Model_Linear_1,vcov.=vcovHC) 
```
t test of coefficients:

                             Estimate  Std. Error t value  Pr(>|t|)    
(Intercept)                449.565561  366.982873  1.2250   0.22060    
Item_Identifier              0.032170    0.039856  0.8071   0.41960    
Item_Weight                  1.133387    2.358086  0.4806   0.63079    
Item_Fat_Content             3.590851   12.590149  0.2852   0.77549    
Item_Visibility           -155.095279  233.188132 -0.6651   0.50600    
Item_MRP                     1.692681    0.194735  8.6922 < 2.2e-16 ***
Outlet_Establishment_Year   10.905541    1.499138  7.2745 3.784e-13 ***
Outlet_Size                 52.613648   22.179867  2.3721   0.01771 *  
Outlet_Location_Type        36.857876   33.529478  1.0993   0.27168    
Outlet_Type                 41.476540   22.608560  1.8346   0.06661 .  
Item_Type_Combined         -47.895962   35.149924 -1.3626   0.17304    
Avg_Item_Outlet_Sales        0.435563    0.254865  1.7090   0.08749 .  
category_by_sales           71.981064   41.636644  1.7288   0.08388 .  
Outlet                       2.063716    7.945758  0.2597   0.79508    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
```

As we can see the significance did not change while correcting for heteroskedasticity. Hence our model is Homoskedastic. This means the variance of error is constant.
Lets try to use step wise regreesion and see if we can improve our score on the leaderboard.

```R
data_linear_test <- read.csv('data_test_python.csv')
library(RcmdrMisc)
Model_Linear_stepwise <- stepwise(Model_Linear, direction="backward",criterion = "AIC")
predicted_values_stepwise <- predict(Model_Linear_stepwise,data_linear_test)
data_linear_test$Item_Outlet_Sales <- predicted_values_stepwise
LinearModelSub_stepwise <- data_linear_test
write.xlsx(LinearModelSub_stepwise,'C:/Users/devarsh patel/Desktop/New folder/LinearModelSub_stepwise.xlsx')
summary(Model_Linear_stepwise)
```
```R
Call:
lm(formula = Item_Outlet_Sales ~ Item_MRP + Outlet_Establishment_Year + 
    Outlet_Size + Outlet_Type, data = data_linear_train)

Residuals:
    Min      1Q  Median      3Q     Max 
-1997.6  -881.5  -139.4   793.7  2116.2 

Coefficients:
                           Estimate Std. Error t value Pr(>|t|)    
(Intercept)               1099.7744    53.0923  20.714  < 2e-16 ***
Item_MRP                     1.6829     0.1766   9.528  < 2e-16 ***
Outlet_Establishment_Year   10.2342     1.4411   7.102 1.33e-12 ***
Outlet_Size                 32.0485    18.3675   1.745    0.081 .  
Outlet_Type                 57.4790    14.3691   4.000 6.38e-05 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 1015 on 8518 degrees of freedom
Multiple R-squared:  0.01874,	Adjusted R-squared:  0.01828 
F-statistic: 40.66 on 4 and 8518 DF,  p-value: < 2.2e-16
```
Our leaderboard score improved from 1790.82081009079 to 1787.94303604482 by using step wise regression because it used variables which were different from Model_Linear_1 based on the optimum AIC value.

Since the accuracy provided by multiple regression and stepwise regression is relatively low lets try to build a regression tree . We dont even have to encode the factor variables as the package(rpart) handles factor variables implicitly. we will first try to build a normal decision tree and then check if we can prune the tree for better accuracy.



```R
library(rpart)

train_decision_tree <- train
train_decision_tree <- subset(train_decision_tree,select = -c(Item_Identifier,Item_Type,category_by_sales))
set.seed(1234)
Model_Decision_Tree_1 <- rpart(Item_Outlet_Sales ~ . , data = train_decision_tree, method = 'anova')
pfit<- prune(Model_Decision_Tree_1, cp=0.010000)
printcp(pfit)
```
```R
printcp(Model_Decision_Tree_1)
```
```R
Regression tree:
rpart(formula = Item_Outlet_Sales ~ ., data = train_decision_tree, 
    method = "anova")

Variables actually used in tree construction:
[1] Item_MRP                  Outlet_Establishment_Year Outlet_Identifier         Outlet_Type              

Root node error: 2.4817e+10/8523 = 2911799

n= 8523 

        CP nsplit rel error  xerror      xstd
1 0.236626      0   1.00000 1.00018 0.0205952
2 0.163179      1   0.76337 0.76420 0.0160746
3 0.058926      2   0.60020 0.60076 0.0138319
4 0.032997      3   0.54127 0.54203 0.0114726
5 0.032966      5   0.47527 0.49374 0.0109148
6 0.013676      6   0.44231 0.44590 0.0104124
7 0.010984      7   0.42863 0.43271 0.0102626
8 0.010000      8   0.41765 0.42647 0.0098994

```
We built a regression tree and tried to find the best accuracy which was (1-0.41765) . Pruning of the tree gave us the same result .
This model works better than the earlier regression models because of the amazing power of classfication algorithms like regreesion trees. Our leaderboard score improved to 1309.73748928418 from 1787.94303604482 


Lets use ensemble methods like Random Forest in order to avoid overfitting of the data, which is one of the major drawbacks of decision trees.
```R
library(randomForest)
library(doSNOW)
library(caret)

data_random_forest <- train

str(data_random_forest)

data_random_forest <- subset(data_random_forest,select = - c(Item_Identifier,Item_Type))

response <- data_random_forest$Item_Outlet_Sales

predictors <- subset(predictors,select = -c(Avg_Item_Outlet_Sales))

bestmtry <- tuneRF(predictors,response)

```

Lets use tuneRF to find the optimal number of variables to select while building a single decision tree in a forests of decision trees

```R
bestmtry <- tuneRF(predictors,response)
```
```R
mtry = 4  OOB error = 1285013 
Searching left ...
mtry = 2 	OOB error = 1318870 
-0.02634776 0.05 
Searching right ...
mtry = 8 	OOB error = 1288806 
-0.002951793 0.05 
```
```R
bestmtry <- tuneRF(predictors,response,ntreeTry = 1000,improve = 0.06)
```
```R
mtry = 4  OOB error = 1231215 
Searching left ...
mtry = 2 	OOB error = 1272505 
-0.03353594 0.06 
Searching right ...
mtry = 8 	OOB error = 1236811 
-0.004545236 0.06 
```
I first tried to find the best mtry by giving default ntreeTry and improve parameters , 4 turned out ot be the best mtry beacuse of the minimum OOB error. Then i tried to increase the ntreeTry to 1000 (we dont have to worry about over-fitting and hence we can set this parameter high) and improve to 0.06, but even in this case the mtry turns out to be 4. 
Hence we will be using mtry = 4 and ntree = 1000 while buidling a random forests model. First we will try to take all the independent variables, after that using the varIMplot function we will filter out the variables which are not contributing towards the models accuarcy

```R
library(randomForest)
library(doSNOW)
library(caret)

data_random_forest <- train
set.seed(1234)
Model_Random_forest_1 <- randomForest(data_random_forest$Item_Outlet_Sales~., data = data_random_forest, mtry = 4,ntree = 1000)
varImpPlot(Model_Random_forest_1)
Model_Random_forest_1
```
```R
Call:
 randomForest(formula = data_random_forest$Item_Outlet_Sales ~      ., data = data_random_forest, mtry = 4, ntree = 1000) 
               Type of random forest: regression
                     Number of trees: 1000
No. of variables tried at each split: 4

          Mean of squared residuals: 1224107
                    % Var explained: 57.96

```
Lets select all the variables except Outlet_Size,category_by_sales and Item_Type, as they are not contributing much towards the accuracy.

```R
data_random_forest <- subset(data_random_forest,select = - c(Outlet_Size,category_by_sales,Item_Type))
set.seed(1234)
Model_Random_forest_2 <- randomForest(data_random_forest$Item_Outlet_Sales~., data = data_random_forest, mtry = 4,ntree = 1000)
```
```R
Model_Random_forest_2
```
```R
Call:
 randomForest(formula = data_random_forest$Item_Outlet_Sales ~      ., data = data_random_forest, mtry = 4, ntree = 1000) 
               Type of random forest: regression
                     Number of trees: 1000
No. of variables tried at each split: 4

          Mean of squared residuals: 1220853
                    % Var explained: 58.07
```
Since we removed some variables and then built Model_Random_forest_2, we maybe removed some kind of correlation present among our variables in our data
As we can notice that the accuracy of the model improved from 57.96 to 58.07, and the leaderboard score changed to 1175.09758415086 from 
1309.73748928418. 
Now the only thing left is to perform cross validation using caret and check if our model can give the same accuracy on unseen data.








