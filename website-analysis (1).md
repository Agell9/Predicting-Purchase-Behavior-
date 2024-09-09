```R
data <- read.csv("customer_purchase_data.csv")
# Convert data types
data$Age <- as.integer(data$Age)
data$Gender <- factor(data$Gender, levels = c(0, 1), labels = c("Female", "Male"))
data$AnnualIncome <- as.numeric(data$AnnualIncome)
data$NumberOfPurchases <- as.integer(data$NumberOfPurchases)
data$ProductCategory <- factor(data$ProductCategory, levels = c(0, 1, 2, 3, 4), 
                                labels = c("Electronics", "Clothing", "Home Goods", "Beauty", "Sports"))
data$TimeSpentOnWebsite <- as.numeric(data$TimeSpentOnWebsite)
data$LoyaltyProgram <- factor(data$LoyaltyProgram, levels = c(0, 1), labels = c("No", "Yes"))
data$DiscountsAvailed <- as.integer(data$DiscountsAvailed)
data$PurchaseStatus <- as.numeric(data$PurchaseStatus)  # Used as numeric for correlation, 
```


```R
# Handle missing values
# Replace missing values with median or mean as appropriate
data$TimeSpentOnWebsite[is.na(data$TimeSpentOnWebsite)] <- median(data$TimeSpentOnWebsite, na.rm = TRUE)
data$DiscountsAvailed[is.na(data$DiscountsAvailed)] <- 0 # Assuming 0 means no discounts availed
```


```R
# Create interaction term between TimeSpentOnWebsite and DiscountsAvailed
data$TimeSpent_DiscountInteraction <- data$TimeSpentOnWebsite * data$DiscountsAvailed

# Binning Age into groups
data$AgeGroup <- cut(data$Age, 
                     breaks = c(0, 18, 25, 35, 50, 65, Inf), 
                     labels = c("0-17", "18-24", "25-34", "35-49", "50-64", "65+"), 
                     right = FALSE)
```


```R
# Distribution of Time Spent on Website
ggplot(data, aes(x=TimeSpentOnWebsite)) + 
    geom_histogram(binwidth=5, fill="skyblue", color="black") + 
    theme_minimal() +
    labs(title="Distribution of Time Spent on Website", x="Time Spent (minutes)", y="Count")

# Time Spent vs Purchase Probability
ggplot(data, aes(x=TimeSpentOnWebsite, y=PurchaseStatus)) + 
    geom_point() + 
    geom_smooth(method="glm", method.args=list(family="binomial"), se=FALSE) + 
    theme_minimal() +
    labs(title="Time Spent on Website vs Purchase Probability")

# Age Group vs Time Spent on Website
ggplot(data, aes(x=AgeGroup, y=TimeSpentOnWebsite)) + 
    geom_boxplot(fill="lightgreen", color="black") + 
    theme_minimal() +
    labs(title="Time Spent on Website by Age Group", x="Age Group", y="Time Spent (minutes)")

# Impact of Discounts on Purchase Status
ggplot(data, aes(x=DiscountsAvailed, y=PurchaseStatus)) + 
    geom_point() + 
    geom_smooth(method="glm", method.args=list(family="binomial"), se=FALSE) + 
    theme_minimal() +
    labs(title="Impact of Discounts on Purchase Status", x="Discounts Availed", y="Purchase Probability")
```


```R
# Split data into training and testing sets
set.seed(123)  # For reproducibility
trainIndex <- sample(1:nrow(data), 0.8 * nrow(data))
trainData <- data[trainIndex, ]
testData <- data[-trainIndex, ]

# Fit logistic regression model
model <- glm(PurchaseStatus ~ TimeSpentOnWebsite + DiscountsAvailed + TimeSpent_DiscountInteraction, 
             data=trainData, family=binomial)
```


```R
# Predictions on test data
predictions <- predict(model, newdata=testData, type="response")
predicted_classes <- ifelse(predictions > 0.5, 1, 0)

# Confusion Matrix
confusion_matrix <- table(Predicted = predicted_classes, Actual = testData$PurchaseStatus)
print(confusion_matrix)

# Calculate accuracy, precision, recall
accuracy <- sum(diag(confusion_matrix)) / sum(confusion_matrix)
precision <- confusion_matrix[2,2] / sum(confusion_matrix[2,])
recall <- confusion_matrix[2,2] / sum(confusion_matrix[,2])

# Print metrics
print(list(accuracy = accuracy, precision = precision, recall = recall))

# ROC Curve and AUC
library(ROCR)
pred <- prediction(predictions, testData$PurchaseStatus)
perf <- performance(pred, measure = "tpr", x.measure = "fpr")
plot(perf, main="ROC Curve")
auc <- performance(pred, measure = "auc")
auc_value <- auc@y.values[[1]]
print(auc_value)

```


```R
# Summary of the logistic regression model
summary(model)
```


```R
# Plot Time Spent on Website vs Purchase Probability
ggplot(data, aes(x=TimeSpentOnWebsite, y=PurchaseStatus)) + 
    geom_point() + 
    geom_smooth(method="glm", method.args=list(family="binomial"), se=FALSE) + 
    theme_minimal() +
    labs(title="Time Spent on Website vs Purchase Probability")

# ROC Curve Visualization
plot(perf, main="ROC Curve")
```
