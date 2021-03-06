# House prices: Ensemble and a comprehensive EDA 
## Kaggle Competition Rank Top 0.2% (12/5410)
https://www.kaggle.com/huiziren1117

## 1. Introduction

Kaggle describes this competition as follows:

Ask a home buyer to describe their dream house, and they probably won’t begin with the height of the basement ceiling or the proximity to an east-west railroad. But this playground competition’s dataset proves that much more influences price negotiations than the number of bedrooms or a white-picket fence.

With 79 explanatory variables describing (almost) every aspect of residential homes in Ames, Iowa, this competition challenges you to predict the final price of each home.

## 2. Executive Summary:

I started the competition by focusing on detailed data exploration just to have a great grasp of the dataset, which is very useful for dealing with missing value and conducting effective feature engineering. EDA process will be introduced here with many visualizations. This project includes

* **Exploratory Data Analysis**

*	**Feature Engineering**

* **Build Machine Learning Base Models in Pipeline** 

* **Grid Search** parameters to optimize a machine learning model

* **Random Forest, XGBoost, Lasso, Elastic Net Regression, Gradient Boosting Regression**

*	**Ensemble**

## 3. Exploratory Data Analysis

**First, Let's start with loading libraries from python**
![1](https://user-images.githubusercontent.com/38633055/40275135-0c473a28-5bb4-11e8-9ea5-3d86c37fb3f4.png)
Next, we load both train and test datasets and merge them in a dataframe called "df", from the shape of which we can see that there are 2917 records and 81 features. Our response variable is "SalePrice".
![134](https://user-images.githubusercontent.com/38633055/40679535-f557d644-6372-11e8-8481-d0e37ff87779.PNG)

After loading the dataset, we would like to do some visualization just to have a clear view of our dataset . The first thing I'm interested in is to see how many missing value in each feature. Because our number of features is not small, for a clear view I ordered the number of missing values in a descending sequence.
![3](https://user-images.githubusercontent.com/38633055/40275191-ba4f4e4c-5bb6-11e8-890a-74ec51254d67.png)

**SalePrice is the variable we need to predict and because variables like price in reality is often right skewed, so Let's do some analysis on it**. 

Basic statistics are shown here:
![4](https://user-images.githubusercontent.com/38633055/40275317-693e84d8-5bba-11e8-8400-569977c53993.png)

We can also use qqplot to check it's normality
![5](https://user-images.githubusercontent.com/38633055/40275327-9741b670-5bba-11e8-9078-51a972d7afc0.png)

As expected the price variable **"SalePrice"** is right skewed:
![6](https://user-images.githubusercontent.com/38633055/40275350-4e1ed0f8-5bbb-11e8-9582-bd8d98902ce1.png)
![7](https://user-images.githubusercontent.com/38633055/40275361-9c2fa3da-5bbb-11e8-86cd-5daee85a3259.png)

As we can see here, the variable "SalePrice" is right skewed and does not show normality in distribution, however most regression models would require normal distribution as their assumptions. We need to add the logrithm to transform the target variable.

Now it looks normal! Both qqplot and density plot reflect that the transformation works
![8](https://user-images.githubusercontent.com/38633055/40275470-9b462974-5bbd-11e8-919d-c66897eb8ab5.png)
![9](https://user-images.githubusercontent.com/38633055/40282234-4a522a4a-5c3a-11e8-8a87-39e9661e6356.png)

**Correlation Matrix:**

![10](https://user-images.githubusercontent.com/38633055/40322961-fd4b7e54-5d01-11e8-852e-3b92b56c7f77.png)

The correlation matrix may still look a bit unclear to you so I extracted top 10 correlated pairs
![12](https://user-images.githubusercontent.com/38633055/40282711-6e1cad9a-5c41-11e8-8c57-7034fb9a4a49.png)

According to the correlation analysis these are the variables mostly correlate with "SalePrice". We can see that **OverallQual** is highly mostly correlated with **SalePrice** with coefficient 0.79, followed by "GrLivArea" with coefficient 0.7. Features like "GarageCars", "GarageArea" and "TotalBsmtSF" are less but still highly correlated with prediction variable with coeffient at 0.64, 0.62 and 0.61. Since the total 5 variables are all highly correlated with house price, multicolinearity may exist.

**Pairwise plot with houseprice:**
![13](https://user-images.githubusercontent.com/38633055/40316986-0ab92f1e-5cee-11e8-9300-717b41c14b49.png) 
![14](https://user-images.githubusercontent.com/38633055/40317241-e84d242a-5cee-11e8-8a4c-eb7d91c37b72.png)

From figures above it seems that overall quality of the house and house price have strong linear relation. "OverallQual" is an orderal variable ranked from 1 to 10. 10 means highest quality which makes sense as normally the higher the quality the higher the price. The linear correlation between "GrLivArea"(ground living area square feet) and house price is less obvious and most datapoints are concentrated at "GrLivArea"=[1000,3000]. "GrLivArea" is the second highest correlated variable which makes sense as expensive houses should have big living area. **Here you might already noticed that the two data points at the right bottom corner seem counterintuitive. It's unreasonable for these two houses having very big living area (around 5000 square feet) but very low price.** I dropped these two data points later and this step do improve my final model error!

![15](https://user-images.githubusercontent.com/38633055/40317293-0854ed70-5cef-11e8-82e0-210736767725.png)

It seems "GarageCars"(size of garage in car capacity) and house price have great linear correlation. "GarageCarS" is an orderal variable ranked from 0 to 4. 4 means biggest size which in general makes sense as usually it takes higher price to have a bigger garage. However we also notice that most size of garage is concentrated between 0 and 3, and we don't see such linear correlation in size value 4. Also there are more outliers in size 3 and 4 than smaller size.

## 4. Feature Engineering
Next I will do some feature engineering to the existing dataset. Since in reality even structured datasets will have some missing value, not clean data formating issues, and further transformation needed. This section will therefore include imputing missing data, re-defining categorical & numerical variables, getting dummy variables, dropping existing irrelavant features, or adding & merging new features.

### 4.1 Missing Values
Usually dataset will contain a lot of missing data. Some of them are not really "missing" just because they are none in nature.For example, quality measurements of a garage can have **NA** when there is no garage. In this case there is no need to impute the missing data. However there are other types of missing value requiring filling in order to maintain effectiveness of data.

First of all, I would like to see which variables have missing value and how many are missing:

![16](https://user-images.githubusercontent.com/38633055/40322576-cb566126-5d00-11e8-9502-8b6e7c7cf76d.png)

So there are 19 variables having missing data and I'm going to deal with them one by one. 
#### PoolQC
As its name suggests, this is quality of the pool of houses. Based on data description, NA simply implies there is no pool in the house.
![17](https://user-images.githubusercontent.com/38633055/40323413-4a67e514-5d03-11e8-9615-7d241a4cfeff.png)

#### MiscFeature
This indicates miscellaneous feature not covered in other categories. It is a categorical variable and to see it's impact on house price I apply box plot to check
![18](https://user-images.githubusercontent.com/38633055/40323953-01a00986-5d05-11e8-9e19-31d1cb835bb0.png)
![19](https://user-images.githubusercontent.com/38633055/40323850-bc9cf7c2-5d04-11e8-8563-1610ec834ddf.png)

I decided to drop this variable because I don't see pattern in this box plot and this type of irrelavant variable will distract final model fitting

#### Alley
This indicates type of alley access to property. It is categorical so we still use boxplot to check:
![20](https://user-images.githubusercontent.com/38633055/40324215-dded8792-5d05-11e8-92a6-856bfe6ba4d7.png)
![21](https://user-images.githubusercontent.com/38633055/40324370-538a2fa0-5d06-11e8-9fc1-1877ca77de41.png)

NA of this variable means that there is no alley access so I assign string "other" to replace NA. As suggested in the boxplot(the two in the middle of above figure), type "Gravel" and type "Paved" seem to have very similar impact so I assign and aggregate both "Grvl" and "Pave" as "Both"

#### Fence
This indicates fence quality and like most quality measurement, NA implies no fence. "Fence" is also a categorical variable I still use boxplot to check if we need to keep this variable. In order to do this I need to see how many distinct value in this feature:
![22](https://user-images.githubusercontent.com/38633055/40325028-62dad9d0-5d08-11e8-8d47-bfee508414cf.png)
![23](https://user-images.githubusercontent.com/38633055/40325580-4da7e010-5d0a-11e8-82ac-5358ad9d7e40.png)

All levels seem stable and even, which means fence quality doesn't have noticable impact on house price, so I dicided to drop this feature

#### FireplaceQu
This indicates fireplace quality and similarly NA means no fireplace, so I replace NA with "None". Again, we use boxplot to check it's impact on house price
![25](https://user-images.githubusercontent.com/38633055/40325978-b2f07dc8-5d0b-11e8-84d0-295a660b8a1e.png)
![24](https://user-images.githubusercontent.com/38633055/40325935-8c8594e8-5d0b-11e8-953b-6f84dfb72b39.png)
![34](https://user-images.githubusercontent.com/38633055/40333973-30dd52fc-5d29-11e8-9616-99a9937804d3.png)

As suggested by the plot above the boxes with better quality have higher sale price, which makes sense. Even though there are some outliers in some categories(the left three boxes), I still decided to keep this variable.

#### LogFrontage
This means linear feet of street connected to property. I decided to drop this because the correlation coeffcient to sale price is low and linear plot also shows weak correlation

![26](https://user-images.githubusercontent.com/38633055/40326503-79a2d244-5d0d-11e8-9e68-359dd2a4c19e.png)

#### GarageCond
As it's name suggested this means the condition of the garage. This is also a categorical variable so I use boxplot to visualize the data. I decided to drop this because all levels seem on a horizontal line and this variable doesn't contribute much information to sale price

![27](https://user-images.githubusercontent.com/38633055/40326794-8b0247f8-5d0e-11e8-9bf1-c2ec73755377.png)

#### Other Garage-Related Variables: GarageType, GarageYrBlt, GarageFinish, GarageQual
These four garage-related variables are very similar and NA indicates no garage. So I replaced NA with string "None"

**Drop variables**

After imputing missing data I still need to clean the data format, because we have some categorical variables that should be numerical and some numerical variables that are essentially categorical. 

Let's first take a look at what are the current numerical variables:

![40](https://user-images.githubusercontent.com/38633055/40335686-f5cc0d70-5d32-11e8-9b25-bc9acd841019.png)

I have gone through these numerical features one by one and since we have too many of these variables I will pick up key transformation and summarize the final results
#### MSSubClass
This variable identifies the type of dwelling involved in the sale. I will change this into a categorical variable as seen from the figure below different levels of MSSubClass do have impact on sale.![28](https://user-images.githubusercontent.com/38633055/40327044-6ea67240-5d0f-11e8-8abf-fd1cca8476ee.png)

#### Basement-related Variables:
Similarly to garage-related variables, these five basement-related variables are very similar and NA indicates no basement. So I replaced NA with string "None"
![29](https://user-images.githubusercontent.com/38633055/40327275-4699438a-5d10-11e8-905f-20cbc206b32c.png)

#### MasVnrType
According to the feature definition, this is masonry veneer type and NA means there is no masonry. Since this is also a categorical variable the boxplot is applied. I will keep this variable and fill NA with "None"
![30](https://user-images.githubusercontent.com/38633055/40333550-0c67d264-5d27-11e8-8f8b-34069b85630c.png)
![31](https://user-images.githubusercontent.com/38633055/40333566-27dba958-5d27-11e8-92f1-a342373aadab.png)

#### MasVnrArea
This variable denotes masonry veneer area in square feet and similar to MasVnrType NA means none masonry actually. This is a numerical variable and in order to better visualize it, I remove the missing value.
![33](https://user-images.githubusercontent.com/38633055/40333722-f42973d2-5d27-11e8-9c68-03ce9e8b3436.png)

We can see that linear correlation exists and I fill the missing value with the average of none-NA value.

#### Electrical
This variable represents electrical system.Since we only have 1 missing value in this variable, I replace the NA with the highest-frequency category. 
![35](https://user-images.githubusercontent.com/38633055/40334890-a29ab65a-5d2e-11e8-9e78-050aa71a9f2a.png)
![39](https://user-images.githubusercontent.com/38633055/40335416-81a25068-5d31-11e8-9bce-48412a10bc09.png)

The most frequent electrical system is "SBrkr" with frequency 1334.

Up to now we have filled with all missing value!
![38](https://user-images.githubusercontent.com/38633055/40335194-6ddb68fe-5d30-11e8-85e8-08b0139d9ae8.png)

### 4.2 Continue Feature Engineering: 
This section will introduce following data methods:

* **Transform Numerical Variables Into Categorical Variables**
* **Tranform date and time**
* **Merge certain variables**

![101](https://user-images.githubusercontent.com/38633055/40367504-1d21e1a6-5dc9-11e8-95d8-02b92e59c8b7.png)

#### OverallCond
It rates the overall material and finish of the house. It's rated from 1 to 10 while 1 means "very poor" and 10 means "Very Excellent". This feature is turned into categorical and for visualization I draw a regression plot of this data to house price.
![102](https://user-images.githubusercontent.com/38633055/40367878-f3db3e18-5dc9-11e8-9813-63eae3fec6b9.png)

We can see the upward trend from rating 1 to 10.

#### YearRemodAdd
This indicates remodel date (same as construction date if no remodeling or additions)
![103](https://user-images.githubusercontent.com/38633055/40368630-c7d6e78e-5dcb-11e8-89e9-d5149638b026.PNG)

Here you can see there is an upward trend in the regression plot that the most recent constructed houses in general have the highest price, but you might also noticed that something strange is happening at year 1950. All the data points before year 1950 are truncated and possibly classified into year 1950, so I replace all the data points before 1950 with another variable "YearBuilt". "YearBuilt" denotes when the houses were built. **This transformation helps me improve the model error!** Figures below show updated variable "YearRemodAdd" and it's regression results 
![104](https://user-images.githubusercontent.com/38633055/40369457-8db96fe8-5dcd-11e8-8b84-a37c16c32c17.PNG)
![105](https://user-images.githubusercontent.com/38633055/40369501-9ec05ab8-5dcd-11e8-8c31-1aefbb1fe444.png)

#### YearBuilt
As explained in the last session, this feature illustrates original construction year. Normally it is not the best to put time variable direclty in dataset. Instead, I will transform this absolute time variable into time difference, which is the age of the house. I do the same for transformed variable "YearRemodAdd"
![106](https://user-images.githubusercontent.com/38633055/40370473-d9ae8bac-5dcf-11e8-9cf3-7061549ed1cd.PNG)

#### Mosold
This indicates the month when the house was sold. I turn this feature into a categorical variable because months are not ordinal. We cannot say Feburary is larger than January. At last, I tranform three variables mentioned above at once.
![111](https://user-images.githubusercontent.com/38633055/40375078-07419d70-5dda-11e8-94d7-84e13d0cd092.PNG)

#### Add "TotalSF" variable
It makes sense to me to see expensive houses own big area. To build a powerful variable indicating the total area. I aggregate total square feet for basement area, first floor and second floor.
![107](https://user-images.githubusercontent.com/38633055/40371854-eba71cd6-5dd2-11e8-9ce7-500d3c799dd9.PNG)
![108](https://user-images.githubusercontent.com/38633055/40371878-fddc62da-5dd2-11e8-811f-8534307b61ee.png)

It's great to see such strong correlation between total area and sale price!

#### Add "TotalBath" variable
Similar to aggregating area features, I also find several features for numbers of bath rooms. They are 
* BsmtFullBath: Basement full bathrooms
* BsmtHalfBath: Basement half bathrooms
* FullBath: Full bathrooms above grade
* HalfBath: Half baths above grade
![109](https://user-images.githubusercontent.com/38633055/40374297-4e867a22-5dd8-11e8-8d5b-524dd4a7335d.PNG)

#### Drop variables
Based on analysis in section 4.1 and 4.2, I will drop variables below:
![110](https://user-images.githubusercontent.com/38633055/40374838-7c0e0c34-5dd9-11e8-8fc7-ac91b5542c3a.PNG)

### 4.3 Transform categorical features into numerical data
In this dataset, we have quite a few categorical data related to quality or condition. These types of data will often reveal ordinal information. *For example, variable "PoolQC" has five levels:* 
* *Ex: Excellent*
* *Gd: Good*
* *TA: Average/Typical*
* *Fa: Fair*
* *NA: No pool*

It's not hard to notice that these five levels are ordinal actually so I assign value 4 to 0 to represent "Ex", "Gd", "TA", "Fa" and NA, respectively. people might argue that even if we leave this variable as category we would still be able to use this feature. I agree, but if we leave it categorical we have to either factorize it or set it as dummy variables, which will greatly increase the variable size in the final dataset and drag down the calculation speed in later modeling part. It's my personal habit to transform certain categorical variables into numbers before getting dummy variables for all categorical data. 

I went through categorical variables and turn the following data into numerical features:
* Utilities
* ExterQual
* ExterCond
* HeatingQC
* KitchenQual
* BsmtQual
* BsmtCond
* PoolQC
* BsmtFinType1
* BsmtFinType2
* FireplaceQu
* GarageQual
![113](https://user-images.githubusercontent.com/38633055/40389088-f1590f1c-5e00-11e8-86da-8512dc4f87c8.PNG)

Besides specifying the transformation in this dataset, I also write another class function to generalize the solution. One disadvantage in python method "label_encoder" is that it randomly assigns numbers to each category, by doing which we might not get the right sequence of numbers coorespondent to right categories. The idea of my solution is that for categorical variables that have obvious ordinal information, label encoder can be applied to these variables and the number is assigned according to it's impact on predicted variable. Not only numbers will be assigned correctly but also each encoder transformation for each variable will be recorded so that numbers can be transferred back to categories.
![115](https://user-images.githubusercontent.com/38633055/40390424-bd3c8020-5e04-11e8-844e-4883d8397762.PNG)

### 4.4 Turn categorical data into dummy variables
At the end, I split sale price from training set as the variable of prediction and I get dummy variables for all the categorical features. 
![114](https://user-images.githubusercontent.com/38633055/40389261-6c8d207e-5e01-11e8-8579-a544d8da8b2e.PNG)

## 5. Modelling   

Load Libraries
![116](https://user-images.githubusercontent.com/38633055/40429348-08e557bc-5e92-11e8-98c4-d7c3ce3a0437.PNG)




### 5.1 Base Machine Learning Models




All the base models are packaged in **pipelines**. Pipelines help automating the workflow of data pre-processing, model training and fitting. In this section, I propose some base models to use and I will optimize the performance of each base model by selecting the most suitable parameters, which is so called gridsearch. 

Before running models, Let's first define the cross validation function (**why do we need this??**). I use the **cross_val_score** function from scikit-learn but this function do not have a shuffle attribute, so I add a line of code **KFold** in order to shuffle the dataset before cross validation.
![117](https://user-images.githubusercontent.com/38633055/40432388-1d6599ac-5e99-11e8-86a8-fc476fb5821d.PNG)

I have also customized a scorer to be used in *GridSearchCV()*
![118](https://user-images.githubusercontent.com/38633055/40432547-7d4e175e-5e99-11e8-9122-75104dfdb704.PNG)

**Base Models**

* **Random Forest**

![119](https://user-images.githubusercontent.com/38633055/40433216-fada2130-5e9a-11e8-8de7-4a1f462f0762.PNG)

* **Lasso**

Penalized regression models like Lasso or Ridge are sensitive to outliers, so we need to robust the dataset before running the model

![120](https://user-images.githubusercontent.com/38633055/40433528-b95ec890-5e9b-11e8-8f3f-7ebe785fd3b4.PNG)

* **Gradient Boosting Regression**

To avoid the model sensitivity to ourtliers, robust scaler is also applied.

![121](https://user-images.githubusercontent.com/38633055/40434766-98dd2690-5e9e-11e8-8984-d53d67ccbd2f.PNG)

* **Elastic Net Regression**

Robust scaler is also applied

![122](https://user-images.githubusercontent.com/38633055/40434863-daf6da3a-5e9e-11e8-9e0c-b4af324f96eb.PNG)





### 5.2 Base Model Scores & Model Selection




Now, Let's see how each base model performs on our dataset by evaluating individual's cross-validation score. Also we will focus on how to optimize each base model based on these scores.

* **1) Elastic Net Regression**

Initial score is:
![124](https://user-images.githubusercontent.com/38633055/40436052-9cea476a-5ea1-11e8-9434-04a2799a3756.PNG)

Elastic Net Regression has two main parameters: alpha and l1 ratio. Then I define two possible sets of data to grid search the best parameter combination
![135](https://user-images.githubusercontent.com/38633055/40683298-b4eac426-637d-11e8-88c1-bf4f9ae203f7.PNG)

From figure above we can see updated scores under each pair of parameters. After selecting the best estimator, Elastic Net Regression model now has improved both in terms of mean and standard deviation. We run the score again

![136](https://user-images.githubusercontent.com/38633055/40683498-53d51316-637e-11e8-8294-0e0af84ce090.PNG)

* **2) Gradient Boosting Regression**

The initial score is:
![128](https://user-images.githubusercontent.com/38633055/40445359-6b56fd64-5ebb-11e8-96f1-30458ef816cb.PNG)

We have six main parameters in Gradient Boosting Regression model: *n_estimators, learning_rate, max_depth, min_samples_leaf, max_features, min_samples_split*, so I arbitrarily define six sets of data to pick up the parameters which will give us the best model score. After updating parameters, I re-run the model and this time score decreases to 0.1118!
![138](https://user-images.githubusercontent.com/38633055/40684045-c9b7521e-637f-11e8-8d12-73475dfce2fc.PNG)

* **3) Lasso**

Lasso performs best initially.
![139](https://user-images.githubusercontent.com/38633055/40684186-3e1ade00-6380-11e8-9fed-88716de37751.PNG)

Lasso has the parameter alpha to control the severity of panelty term and it is also the model driver. After trying a few value of alpha just to get idea about what the suitable range would be, I have winded on an interval [0.0005,0.0006] to search for the best alpha value.

![140](https://user-images.githubusercontent.com/38633055/40684331-8c3a18c6-6380-11e8-9e21-80b45d5350b2.PNG)

* **4) Random Forest**

The initial score:
![127](https://user-images.githubusercontent.com/38633055/40437343-9961d7ae-5ea4-11e8-9994-5ee94e1e6dae.PNG)

After gridsearching the parameter "max_feature" in RandomForest and find value XX is the most suitable.





### 5.3 Model Stacking




After optimizing individual base model, I will apply stacking approach to unite these models for more accurate prediction. I start with a simple approach averaging the prediction value from base models. Here is the class I use to stack

![132](https://user-images.githubusercontent.com/38633055/40450894-817b3a0e-5ecc-11e8-8e29-94629bea5e73.PNG)

From last session we find out that Lasso, Gradient Boosting Regression and Elastic Net Regression perform best, so here I stack these models by simply taking average.

![141](https://user-images.githubusercontent.com/38633055/40684415-ce434f4e-6380-11e8-8fec-fbd99f5deb86.PNG)


## Conclusion

Cross-validation score is 0.1080 after ensembling Elastic Nets Regression, Lasso and Gradient Boosting. We apply this model to fit the test set and get final test score 0.10985

