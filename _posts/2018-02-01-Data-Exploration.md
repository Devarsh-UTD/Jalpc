---
published: true
---

### We’ll be performing some basic data exploration here and come up with inferences about the data. 
### We’ll try to figure out some irregularities and address them in the next section. 
### We will combine the test and train in order to perform our feature engineering  efficiently and later       divide them again.

Lets start of reading out test and train files into R.

```
library(xlsx)
train <- read.xlsx('Train_UWu5bXk.xlsx',sheetIndex = 1)
test <- read.xlsx('Test_u94Q5KV.xlsx', sheetIndex = 1)
library(xlsx)
temp <- data.frame(Item_Outlet_Sales=rep("None",nrow(test)),test[,])
full_data <- rbind(train,temp)
```
Lets perform some basic exploratory analysis of full_data.
```
str(full_data)
```
```
'data.frame':	14204 obs. of  12 variables:
     $ Item_Identifier          : Factor w/ 1559 levels "DRA12","DRA24",..: 157 9 663 1122 1298 759 697 739 441 991 ...
     $ Item_Weight              : num  9.3 5.92 17.5 19.2 8.93 ...
     $ Item_Fat_Content         : Factor w/ 5 levels "LF","low fat",..: 3 5 3 5 3 5 5 3 5 5 ...
     $ Item_Visibility          : num  0.016 0.0193 0.0168 0 0 ...
     $ Item_Type                : Factor w/ 16 levels "Baking Goods",..: 5 15 11 7 10 1 14 14 6 6 ...
     $ Item_MRP                 : num  249.8 48.3 141.6 182.1 53.9 ...
     $ Outlet_Identifier        : Factor w/ 10 levels "OUT010","OUT013",..: 10 4 10 1 2 4 2 6 8 3 ...
     $ Outlet_Establishment_Year: num  1999 2009 1999 1998 1987 ...
     $ Outlet_Size              : Factor w/ 3 levels "High","Medium",..: 2 2 2 NA 1 2 1 2 NA NA ...
     $ Outlet_Location_Type     : Factor w/ 3 levels "Tier 1","Tier 2",..: 1 3 1 3 3 3 3 3 2 2 ...
     $ Outlet_Type              : Factor w/ 4 levels "Grocery Store",..: 2 3 2 1 2 3 2 4 2 2 ...
     $ Item_Outlet_Sales        : chr  "3735.138" "443.4228" "2097.27" "732.38" ...
```
```
summary(full_data)
```
```
Item_Identifier  Item_Weight     Item_Fat_Content Item_Visibility                   Item_Type       Item_MRP     
 DRA24  :   10   Min.   : 4.555   LF     : 522     Min.   :0.00000   Fruits and Vegetables:2013   Min.   : 31.29  
 DRA59  :   10   1st Qu.: 8.710   low fat: 178     1st Qu.:0.02704   Snack Foods          :1989   1st Qu.: 94.01  
 DRB25  :   10   Median :12.600   Low Fat:8485     Median :0.05402   Household            :1548   Median :142.25  
 DRC25  :   10   Mean   :12.793   reg    : 195     Mean   :0.06595   Frozen Foods         :1426   Mean   :141.00  
 DRC27  :   10   3rd Qu.:16.750   Regular:4824     3rd Qu.:0.09404   Dairy                :1136   3rd Qu.:185.86  
 DRC36  :   10   Max.   :21.350                    Max.   :0.32839   Baking Goods         :1086   Max.   :266.89  
 (Other):14144   NA's   :2439                                        (Other)              :5006                   
 Outlet_Identifier Outlet_Establishment_Year Outlet_Size   Outlet_Location_Type            Outlet_Type   Item_Outlet_Sales 
 OUT027 :1559      Min.   :1985              High  :1553   Tier 1:3980          Grocery Store    :1805   Length:14204      
 OUT013 :1553      1st Qu.:1987              Medium:4655   Tier 2:4641          Supermarket Type1:9294   Class :character  
 OUT035 :1550      Median :1999              Small :3980   Tier 3:5583          Supermarket Type2:1546   Mode  :character  
 OUT046 :1550      Mean   :1998              NA's  :4016                        Supermarket Type3:1559                     
 OUT049 :1550      3rd Qu.:2004                                                                                            
 OUT045 :1548      Max.   :2009                                                                                            
 (Other):4894
 ```
Some useful observations
   1. Item_Visibility has a min value of zero. This makes no practical sense because when a product is being       sold in a store, the visibility cannot be 0.
   2. Outlet_Establishment_Years vary from 1985 to 2009. The values might not be apt in this form. Rather, 
      if we can convert them to how old the particular store is, it should have a better impact on sales.
   3. The lower ‘count’ of Item_Weight and Item_Outlet_Sales confirms the findings from the missing value         check.
Lets check for missing values in the data.
```
colnames(full_data)[colSums(is.na(full_data)) > 0]
```
```
"Item_Weight" "Outlet_Size"
```
Thus we have two columns with missing values, we will impute the missing data in data cleaning section.
Now lets look at the unique values present in each of the categorical columns.
```
unique_values <- apply(full_data, 2, function(x)length(unique(x)))
```
```
              Item_Identifier               Item_Weight          Item_Fat_Content           Item_Visibility 
                     1559                       416                         5                     13006 
                Item_Type                  Item_MRP         Outlet_Identifier Outlet_Establishment_Year 
                       16                      8052                        10                         9 
              Outlet_Size      Outlet_Location_Type               Outlet_Type         Item_Outlet_Sales 
                        4                         3                         4                      3494 
 ```                      
This tells us that there are 1559 products and 10 outlets/stores.Another thing that should catch attention is that Item_Type has 16 unique values. Let’s explore further using the frequency of different categories in each nominal variable. I’ll exclude the ID and source variables for obvious reasons
```
var1 <- sapply(full_data , is.factor)
cat_var <- full_data[var1]
cat_var <- subset(cat_var, select = - Item_Identifier)
unique_values1 <- apply(cat_var, 2, unique)
unique_values1
```

```
Item_Fat_Content
[1] "Low Fat" "Regular" "low fat" "LF"      "reg"    

Item_Type
 [1] "Dairy"                 "Soft Drinks"           "Meat"                  "Fruits and Vegetables"
 [5] "Household"             "Baking Goods"          "Snack Foods"           "Frozen Foods"         
 [9] "Breakfast"             "Health and Hygiene"    "Hard Drinks"           "Canned"               
[13] "Breads"                "Starchy Foods"         "Others"                "Seafood"              

Outlet_Identifier
 [1] "OUT049" "OUT018" "OUT010" "OUT013" "OUT027" "OUT045" "OUT017" "OUT046" "OUT035" "OUT019"

Outlet_Size
[1] "Medium" NA       "High"   "Small" 

Outlet_Location_Type
[1] "Tier 1" "Tier 3" "Tier 2"

Outlet_Type
[1] "Supermarket Type1" "Supermarket Type2" "Grocery Store"     "Supermarket Type3"
```
The output gives us following observations:
   1. Item_Fat_Content: Some of ‘Low Fat’ values mis-coded as ‘low fat’ and ‘LF’. Also, some of ‘Regular’  
      are mentioned as ‘regular’.
   2. Item_Type: Not all categories have substantial numbers. It looks like combining them can give better         results.
   3. Outlet_Type: Supermarket Type2 and Type3 can be combined.

# Data Cleaning

Herein we would inpute the missing values present in 'Item_Weight'  using rpart package.
```
temp1 <- full_data
temp1 <- subset(temp1, select = - Item_Outlet_Sales)
anova_mod <- rpart(Item_Weight ~ . , 
                   data=temp1[!is.na(temp1$Item_Weight), ], method="anova", na.action=na.omit)  
x <- predict(anova_mod,temp1[is.na(temp1$Item_Weight),])
full_data[is.na(full_data$Item_Weight),"Item_Weight"] <- x
colnames(full_data)[colSums(is.na(full_data)) > 0]
```
```
[1] "Outlet_Size"
```
We have successfully developed a decision tree model using the anova method in rpart  and used it to perform imputations on missing values in Item_Weight column.
I would be using the mice package to perform categorical imputations on 'Outlet_Size'
```
library(mice)
temp2 <- full_data
temp2 <- subset(temp2, select = - Item_Outlet_Sales)
miceMod <- mice(temp2[, !names(temp2) %in% "Item_Outlet_Sales"], method="rf")  
miceOutput <- complete(miceMod)
colnames(full_data)[colSums(is.na(full_data)) > 0]
```
```
character(0)
```
Hence we have filled the missing values in 'Outlet_Size'

# Feature Engineering

Lets see if we can combine the Outlet_Type categories. A quick way to check that could be to analyze the mean sales by type of store. If they have similar sales, then keeping them separate won’t help much.
```
library(sqldf)
sqldf('select AVG(Item_Outlet_Sales), Outlet_Type from new group by Outlet_Type ')
```
```
  AVG(Item_Outlet_Sales)       Outlet_Type
1               203.8971     Grocery Store
2              1389.8582 Supermarket Type1
3              1197.8155 Supermarket Type2
4              2215.4753 Supermarket Type3
```
Since there is a significant difference among the different types of  Outlet_Type, we will not combine them.
We noticed that the minimum value in Item_ Visibility was 0, which makes no practical sense. Lets consider it like missing information and impute it with mean visibility of that product.
```
sqldf('select COUNT(Item_Visibility) from new where Item_Visibility = 0 ')
```
```
          COUNT(Item_Visibility)
1                    879
```
```
new$Item_Visibility[new$Item_Visibility == 0] <- mean(new$Item_Visibility)
sqldf('select COUNT(Item_Visibility) from new where Item_Visibility = 0 ')
```
```
           COUNT(Item_Visibility)
1                      0
```
Item_Type variable has 16 categories which might prove to be very useful in analysis. So its a good idea to combine them. If you look at the Item_Identifier, i.e. the unique ID of each item, it starts with either FD, DR or NC. If you see the categories, these look like being Food, Drinks and Non-Consumables.
```
get_first_2_char <-sapply(new[,'Item_Identifier'], substring, 1, 2)
length(get_first_2_char)
temp3 <- ''
for(i in 1:length(get_first_2_char)){
   if(get_first_2_char[i] == 'FD'){
     temp3[i] <- 'Food' 
   }else if(get_first_2_char[i] == 'DR'){
     temp3[i] <- 'Drinks'
   }else{
     temp3[i] <- 'Non-Consumable'
   }
 }
new$Item_Type_Combined <-temp3
```
```
head(new$Item_Type_Combined)
[1] "Food"           "Drinks"         "Food"           "Food"           "Non-Consumable" "Food"          
```
Lets determine the 'Outlet_Establishment_Year' of a store in terms of differences rather than years because it would be more meaningful and add a column for 'average item outlet sales'.

```
new_full_data['Outlet_Establishment_Year'] <- 2013 - new_full_data$Outlet_Establishment_Year
s <- sqldf('select AVG(Item_Outlet_Sales),Item_Type from new group by Item_Type')
new_full_data <- sqldf('select * from new n INNER JOIN s on n.Item_Type = s.Item_Type ')
summary(new_full_data)
```
```
Item_Identifier  Item_Weight     Item_Fat_Content Item_Visibility                    Item_Type   
 DRA24  :   10   Min.   : 4.555   LF     : 522     Min.   :0.003575   Fruits and Vegetables:2013  
 DRA59  :   10   1st Qu.: 8.600   low fat: 178     1st Qu.:0.033143   Snack Foods          :1989  
 DRB25  :   10   Median :12.500   Low Fat:8485     Median :0.062347   Household            :1548  
 DRC25  :   10   Mean   :12.795   reg    : 195     Mean   :0.070034   Frozen Foods         :1426  
 DRC27  :   10   3rd Qu.:16.750   Regular:4824     3rd Qu.:0.094037   Dairy                :1136  
 DRC36  :   10   Max.   :21.350                    Max.   :0.328391   Baking Goods         :1086  
 (Other):14144                                                        (Other)              :5006  
    Item_MRP      Outlet_Identifier Outlet_Establishment_Year Outlet_Size   Outlet_Location_Type
 Min.   : 31.29   OUT027 :1559      Min.   : 4.00             High  :1591   Tier 1:3980         
 1st Qu.: 94.01   OUT013 :1553      1st Qu.: 9.00             Medium:5588   Tier 2:4641         
 Median :142.25   OUT035 :1550      Median :14.00             Small :7025   Tier 3:5583         
 Mean   :141.00   OUT046 :1550      Mean   :15.17                                               
 3rd Qu.:185.86   OUT049 :1550      3rd Qu.:26.00                                               
 Max.   :266.89   OUT045 :1548      Max.   :28.00                                               
                  (Other):4894                                                                  
            Outlet_Type   Item_Outlet_Sales Item_Type_Combined Avg_Item_Outlet_Sales                 Item_Type   
 Grocery Store    :1805   None     :5681    Length:14204       Min.   :1163          Fruits and Vegetables:2013  
 Supermarket Type1:9294   958.752  :  17    Class :character   1st Qu.:1247          Snack Foods          :1989  
 Supermarket Type2:1546   1342.2528:  16    Mode  :character   Median :1328          Household            :1548  
 Supermarket Type3:1559   1845.5976:  15                       Mean   :1309          Frozen Foods         :1426  
                          703.0848 :  15                       3rd Qu.:1374          Dairy                :1136  
                          1230.3984:  14                       Max.   :1673          Baking Goods         :1086  
                          (Other)  :8446                                             (Other)              :5006  
```
We noted that 'Item_Fat_Content' had some irregularities because LF,lowfat and Low Fat indicate the same categorical level but have been treated differently in the data set, same goes for 'reg' and 'Regular'. Lets try to combine these levels .
```
temp4 <- new_full_data

for(i in 1:nrow(temp4)){
  if(temp4[i,'Item_Fat_Content'] == 'LF' || temp4[i,'Item_Fat_Content'] == 'low fat' || temp4[i,'Item_Fat_Content'] == 'Low Fat' ){
    
    temp4[i,'Item_Fat_Content'] <- 'Low Fat'
  }else{
    temp4[i,'Item_Fat_Content'] <- 'Regular'
  }
}
new_full_data <- temp4
summary(temp4)
```
```R
Item_Identifier  Item_Weight     Item_Fat_Content Item_Visibility                    Item_Type   
 DRA24  :   10   Min.   : 4.555   LF     :   0     Min.   :0.003575   Fruits and Vegetables:2013  
 DRA59  :   10   1st Qu.: 8.600   low fat:   0     1st Qu.:0.033143   Snack Foods          :1989  
 DRB25  :   10   Median :12.500   Low Fat:9185     Median :0.062347   Household            :1548  
 DRC25  :   10   Mean   :12.795   reg    :   0     Mean   :0.070034   Frozen Foods         :1426  
 DRC27  :   10   3rd Qu.:16.750   Regular:5019     3rd Qu.:0.094037   Dairy                :1136  
 DRC36  :   10   Max.   :21.350                    Max.   :0.328391   Baking Goods         :1086  
 (Other):14144                                                        (Other)              :5006  
    Item_MRP      Outlet_Identifier Outlet_Establishment_Year Outlet_Size   Outlet_Location_Type
 Min.   : 31.29   OUT027 :1559      Min.   : 4.00             High  :1591   Tier 1:3980         
 1st Qu.: 94.01   OUT013 :1553      1st Qu.: 9.00             Medium:5588   Tier 2:4641         
 Median :142.25   OUT035 :1550      Median :14.00             Small :7025   Tier 3:5583         
 Mean   :141.00   OUT046 :1550      Mean   :15.17                                               
 3rd Qu.:185.86   OUT049 :1550      3rd Qu.:26.00                                               
 Max.   :266.89   OUT045 :1548      Max.   :28.00                                               
                  (Other):4894                                                                  
            Outlet_Type   Item_Outlet_Sales Item_Type_Combined Avg_Item_Outlet_Sales                 Item_Type   
 Grocery Store    :1805   None     :5681    Length:14204       Min.   :1163          Fruits and Vegetables:2013  
 Supermarket Type1:9294   958.752  :  17    Class :character   1st Qu.:1247          Snack Foods          :1989  
 Supermarket Type2:1546   1342.2528:  16    Mode  :character   Median :1328          Household            :1548  
 Supermarket Type3:1559   1845.5976:  15                       Mean   :1309          Frozen Foods         :1426  
                          703.0848 :  15                       3rd Qu.:1374          Dairy                :1136  
                          1230.3984:  14                       Max.   :1673          Baking Goods         :1086  
                          (Other)  :8446                                             (Other)              :5006 
```
As you can notice above that even though we have replaced LF, lowfat and Low Fat with the same name but the factor levels are still different, lets handle this problem
```
temp4$Item_Fat_Content <- as.character(temp4$Item_Fat_Content)
temp4$Item_Fat_Content <- as.factor(temp4$Item_Fat_Content)
summary(temp4)
```
```
 Item_Identifier  Item_Weight     Item_Fat_Content Item_Visibility                    Item_Type   
 DRA24  :   10   Min.   : 4.555   Low Fat:9185     Min.   :0.003575   Fruits and Vegetables:2013  
 DRA59  :   10   1st Qu.: 8.600   Regular:5019     1st Qu.:0.033143   Snack Foods          :1989  
 DRB25  :   10   Median :12.500                    Median :0.062347   Household            :1548  
 DRC25  :   10   Mean   :12.795                    Mean   :0.070034   Frozen Foods         :1426  
 DRC27  :   10   3rd Qu.:16.750                    3rd Qu.:0.094037   Dairy                :1136  
 DRC36  :   10   Max.   :21.350                    Max.   :0.328391   Baking Goods         :1086  
 (Other):14144                                                        (Other)              :5006  
    Item_MRP      Outlet_Identifier Outlet_Establishment_Year Outlet_Size   Outlet_Location_Type
 Min.   : 31.29   OUT027 :1559      Min.   : 4.00             High  :1591   Tier 1:3980         
 1st Qu.: 94.01   OUT013 :1553      1st Qu.: 9.00             Medium:5588   Tier 2:4641         
 Median :142.25   OUT035 :1550      Median :14.00             Small :7025   Tier 3:5583         
 Mean   :141.00   OUT046 :1550      Mean   :15.17                                               
 3rd Qu.:185.86   OUT049 :1550      3rd Qu.:26.00                                               
 Max.   :266.89   OUT045 :1548      Max.   :28.00                                               
                  (Other):4894                                                                  
            Outlet_Type   Item_Outlet_Sales Item_Type_Combined Avg_Item_Outlet_Sales                 Item_Type   
 Grocery Store    :1805   None     :5681    Length:14204       Min.   :1163          Fruits and Vegetables:2013  
 Supermarket Type1:9294   958.752  :  17    Class :character   1st Qu.:1247          Snack Foods          :1989  
 Supermarket Type2:1546   1342.2528:  16    Mode  :character   Median :1328          Household            :1548  
 Supermarket Type3:1559   1845.5976:  15                       Mean   :1309          Frozen Foods         :1426  
                          703.0848 :  15                       3rd Qu.:1374          Dairy                :1136  
                          1230.3984:  14                       Max.   :1673          Baking Goods         :1086  
                          (Other)  :8446                                             (Other)              :5006 
```
Hang on we saw there were some non-consumables as well and a fat-content should not be specified for them. So we can also create a separate category for such kind of observations.
```
temp5 <- new_full_data
temp5$Item_Fat_Content <- as.character(temp5$Item_Fat_Content)
for(i in 1:nrow(temp5)){
  if(temp5[i,'Item_Type_Combined'] == 'Non-Consumable'){
    temp5[i,'Item_Fat_Content'] <- 'Non-Edible'
  }else{
    next
  }
}

temp5$Item_Fat_Content <- as.factor(temp5$Item_Fat_Content)
temp5 <- temp5 %>% subset(., select=which(!duplicated(names(.)))) 
temp5$Item_Type_Combined <- as.factor(temp5$Item_Type_Combined)
summary(temp5)
```
```
Item_Identifier  Item_Weight       Item_Fat_Content Item_Visibility                    Item_Type       Item_MRP     
 DRA24  :   10   Min.   : 4.555   Low Fat   :6499    Min.   :0.003575   Fruits and Vegetables:2013   Min.   : 31.29  
 DRA59  :   10   1st Qu.: 8.600   Non-Edible:2686    1st Qu.:0.033143   Snack Foods          :1989   1st Qu.: 94.01  
 DRB25  :   10   Median :12.500   Regular   :5019    Median :0.062347   Household            :1548   Median :142.25  
 DRC25  :   10   Mean   :12.795                      Mean   :0.070034   Frozen Foods         :1426   Mean   :141.00  
 DRC27  :   10   3rd Qu.:16.750                      3rd Qu.:0.094037   Dairy                :1136   3rd Qu.:185.86  
 DRC36  :   10   Max.   :21.350                      Max.   :0.328391   Baking Goods         :1086   Max.   :266.89  
 (Other):14144                                                          (Other)              :5006                   
 Outlet_Identifier Outlet_Establishment_Year Outlet_Size   Outlet_Location_Type            Outlet_Type   Item_Outlet_Sales
 OUT027 :1559      Min.   : 4.00             High  :1591   Tier 1:3980          Grocery Store    :1805   None     :5681   
 OUT013 :1553      1st Qu.: 9.00             Medium:5588   Tier 2:4641          Supermarket Type1:9294   958.752  :  17   
 OUT035 :1550      Median :14.00             Small :7025   Tier 3:5583          Supermarket Type2:1546   1342.2528:  16   
 OUT046 :1550      Mean   :15.17                                                Supermarket Type3:1559   1845.5976:  15   
 OUT049 :1550      3rd Qu.:26.00                                                                         703.0848 :  15   
 OUT045 :1548      Max.   :28.00                                                                         1230.3984:  14   
 (Other):4894                                                                                            (Other)  :8446   
      Item_Type_Combined Avg_Item_Outlet_Sales
 Drinks        : 1317    Min.   :1163         
 Food          :10201    1st Qu.:1247         
 Non-Consumable: 2686    Median :1328         
                         Mean   :1309         
                         3rd Qu.:1374         
                         Max.   :1673     
```
Hence we have successfully converted 'Item_Fat_Content' having three factor levels namely Low Fat, Regular and Non-Edible and converted 'Item_Type_Combined' into a factor. Thus we have completed the Data Exploration and cleaning process and made our dataset ready for model building. Let's divide the dataset into training and testing again.

```
train <- temp5[which(!temp5$Item_Outlet_Sales == 'None'),]
test <- temp5[which(temp5$Item_Outlet_Sales == 'None'),]
test <- subset(test, select = - Item_Outlet_Sales)
```
