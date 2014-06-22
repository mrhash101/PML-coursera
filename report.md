Modeling Engine for Accelerometer Data
========================================================
Hassan 21-06-2014

Executive Summary
======================
Activity data looks promising. Using the given parameters, a model was made to predict the outcome for a new user. Depending upon given factors the model will accurately judge whether the motion employed by the user is correct form or not. For this purpose the motion is divided into classes i.e class A, class B, and class C movements. Since the model has been trained on data from a variety of users, we can safely assume that the model will be able to help any and all users. Non linear techniques were applied using random forests and data was divided into 10-fold cross validaiton sets. The results came out to be 99% accurate and all the answers for test cases were satisfied 100% correctly. 

Handling Missing values and establishing Variable Importance
============================================================

Data was read using the following lines of code. All the variables with missing and NA values were removed from the dataset. Data was cleaned using the following code and then written to a file for k -fold cross validation


```r
data <- read.csv(file = "pml-training.csv")
y <- data[, !sapply(data, function(x) any(is.na(x)))]
z <- y[, !sapply(y, function(x) any(x == ""))]
z <- z[, -1]
head(z)
```

```
##   user_name raw_timestamp_part_1 raw_timestamp_part_2   cvtd_timestamp
## 1  carlitos           1323084231               788290 05/12/2011 11:23
## 2  carlitos           1323084231               808298 05/12/2011 11:23
## 3  carlitos           1323084231               820366 05/12/2011 11:23
## 4  carlitos           1323084232               120339 05/12/2011 11:23
## 5  carlitos           1323084232               196328 05/12/2011 11:23
## 6  carlitos           1323084232               304277 05/12/2011 11:23
##   new_window num_window roll_belt pitch_belt yaw_belt total_accel_belt
## 1         no         11      1.41       8.07    -94.4                3
## 2         no         11      1.41       8.07    -94.4                3
## 3         no         11      1.42       8.07    -94.4                3
## 4         no         12      1.48       8.05    -94.4                3
## 5         no         12      1.48       8.07    -94.4                3
## 6         no         12      1.45       8.06    -94.4                3
##   gyros_belt_x gyros_belt_y gyros_belt_z accel_belt_x accel_belt_y
## 1         0.00         0.00        -0.02          -21            4
## 2         0.02         0.00        -0.02          -22            4
## 3         0.00         0.00        -0.02          -20            5
## 4         0.02         0.00        -0.03          -22            3
## 5         0.02         0.02        -0.02          -21            2
## 6         0.02         0.00        -0.02          -21            4
##   accel_belt_z magnet_belt_x magnet_belt_y magnet_belt_z roll_arm
## 1           22            -3           599          -313     -128
## 2           22            -7           608          -311     -128
## 3           23            -2           600          -305     -128
## 4           21            -6           604          -310     -128
## 5           24            -6           600          -302     -128
## 6           21             0           603          -312     -128
##   pitch_arm yaw_arm total_accel_arm gyros_arm_x gyros_arm_y gyros_arm_z
## 1      22.5    -161              34        0.00        0.00       -0.02
## 2      22.5    -161              34        0.02       -0.02       -0.02
## 3      22.5    -161              34        0.02       -0.02       -0.02
## 4      22.1    -161              34        0.02       -0.03        0.02
## 5      22.1    -161              34        0.00       -0.03        0.00
## 6      22.0    -161              34        0.02       -0.03        0.00
##   accel_arm_x accel_arm_y accel_arm_z magnet_arm_x magnet_arm_y
## 1        -288         109        -123         -368          337
## 2        -290         110        -125         -369          337
## 3        -289         110        -126         -368          344
## 4        -289         111        -123         -372          344
## 5        -289         111        -123         -374          337
## 6        -289         111        -122         -369          342
##   magnet_arm_z roll_dumbbell pitch_dumbbell yaw_dumbbell
## 1          516         13.05         -70.49       -84.87
## 2          513         13.13         -70.64       -84.71
## 3          513         12.85         -70.28       -85.14
## 4          512         13.43         -70.39       -84.87
## 5          506         13.38         -70.43       -84.85
## 6          513         13.38         -70.82       -84.47
##   total_accel_dumbbell gyros_dumbbell_x gyros_dumbbell_y gyros_dumbbell_z
## 1                   37                0            -0.02             0.00
## 2                   37                0            -0.02             0.00
## 3                   37                0            -0.02             0.00
## 4                   37                0            -0.02            -0.02
## 5                   37                0            -0.02             0.00
## 6                   37                0            -0.02             0.00
##   accel_dumbbell_x accel_dumbbell_y accel_dumbbell_z magnet_dumbbell_x
## 1             -234               47             -271              -559
## 2             -233               47             -269              -555
## 3             -232               46             -270              -561
## 4             -232               48             -269              -552
## 5             -233               48             -270              -554
## 6             -234               48             -269              -558
##   magnet_dumbbell_y magnet_dumbbell_z roll_forearm pitch_forearm
## 1               293               -65         28.4         -63.9
## 2               296               -64         28.3         -63.9
## 3               298               -63         28.3         -63.9
## 4               303               -60         28.1         -63.9
## 5               292               -68         28.0         -63.9
## 6               294               -66         27.9         -63.9
##   yaw_forearm total_accel_forearm gyros_forearm_x gyros_forearm_y
## 1        -153                  36            0.03            0.00
## 2        -153                  36            0.02            0.00
## 3        -152                  36            0.03           -0.02
## 4        -152                  36            0.02           -0.02
## 5        -152                  36            0.02            0.00
## 6        -152                  36            0.02           -0.02
##   gyros_forearm_z accel_forearm_x accel_forearm_y accel_forearm_z
## 1           -0.02             192             203            -215
## 2           -0.02             192             203            -216
## 3            0.00             196             204            -213
## 4            0.00             189             206            -214
## 5           -0.02             189             206            -214
## 6           -0.03             193             203            -215
##   magnet_forearm_x magnet_forearm_y magnet_forearm_z classe
## 1              -17              654              476      A
## 2              -18              661              473      A
## 3              -18              658              469      A
## 4              -16              658              469      A
## 5              -17              655              473      A
## 6               -9              660              478      A
```


Variable importance can be established using the caret package. This can be done using the fscaret package as well, which makes it much more automatic and does not require the user to interfere. The following lines of R code were used to establish variable importance and to create a preliminary model. The processed data contained 60 variables in the database with 59 variables used as input.


Reading Files and PreProcessing Data
====================================

Increased variance and reduced bias in the training data leads to more accurate and reliable models. In order to achieve such a training set, the pre-processed training file was divided into 10-fold cross validation and then split into separate files for further use. Writing the split of data in separate files was done so that the experiment could be reproduced easily. The following purpose written R code was used to achieve 10-fold cross validation. 



```r
## file options
filename <- "~/Documents/Coursera/Data_science/Practical_machine_learning/assignment/pml-prepared-training.csv"
header <- TRUE
sep <- ","  #other options - ' ', ',', ';', '\t'
filename_t <- "t-pml_no"
filename_tr <- "pml_no"

# K value - how many fold do you want
K = 10

# DO NOT MODIFY BELOW
# ----------------------------------------------------------------------------------------------------

data <- read.csv(filename, header = header, sep = sep)

## create cv combinations from package cvTools
library(cvTools)
```

```
## Loading required package: lattice
## Loading required package: robustbase
```

```r

# the number of rows to be split
n = nrow(data)

# storing the results for further use
cv_index <- cvFolds(n, K, type = c("random"))

# Copying cv_index values in to a matrix to avoid any problems with loops
mat <- matrix(data = c(cv_index[[5]], cv_index[[4]]), ncol = 2, nrow = n, dimnames = list(c(), 
    c("df", "rownames")))

# ordering mat according to K number - do we need to do this here? mat<-
# mat[order(mat[,1], decreasing = FALSE),]

## merge the required rows with the rownames matrix - this works
data_x <- merge(cbind(data, row = row.names(data)), mat, by.x = "row", by.y = "rownames")

# subset the data for all K folds into separate dataframes - and write into
# files
for (i in 1:K) {
    
    # saving the subsetted data into temp_df
    test_df <- subset(data_x, data_x$df == i)
    train_df <- subset(data_x, data_x$df != i)
    
    # Omitting rownames and df labels
    write_test_df <- subset(test_df, select = -c(row, df))
    write_train_df <- subset(train_df, select = -c(row, df))
    
    # create unique filenames with every iteration
    filename_test <- paste0(filename_t, i, ".txt")
    filename_train <- paste0(filename_tr, i, ".txt")
    
    # write the subseted rows in test_df and train_df into different files
    # write.table(write_test_df, file = filename_test, sep = '\t', row.names =
    # FALSE, col.names = FALSE) write.table(write_train_df, file =
    # filename_train, sep = '\t', row.names = FALSE, col.names = FALSE)
}
```


Modeling Engine
================

With a variety of missing values, a good modeling choice would be Random Forests in R. Random Forests can be implemented by the randomForest package and as well as the caret package. It can accept values in numerics, characters, and factors. Therefore, all the date and timestamps were not pre-processed in the main database. In this study, the model was created using the caret package using the 'rf' parameter for the method function. The models were selected based on the best run of the 10-fold cross validation. Later on, internal validation with the 10-fold test sets were done which yielded excellent results. Model selection was done using RMSE values. External validation was done on the best model using the data provided by the instructors. All 20 answers came out to be correct. 


Results and Conclusion
======================

The results show that many using robust techniques, many variables are not required to establish a non linear relationship between different variables of data. The resulting models are giving good predictions based on the test sets derived from 10-fold cross validation. Random forests are easy to implement, require less computing resources, and easy to interpret. They are an excellent substitute to linear modeling on a local computer. In conclusion, 59 variables out of the provided data were sufficient to model the data correctly, with 99% accuracy levels with 10-fold cross validation. 


