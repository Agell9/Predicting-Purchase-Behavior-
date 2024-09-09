# Predicting-Purchase-Behavior-
In today’s competitive e-commerce environment, understanding and predicting customer purchase behavior is crucial for optimizing marketing efforts and improving conversion rates. Companies need to identify the factors that most influence a customer's decision to buy 


# Objective

The primary objective of this project is to predict customer purchase behavior based on their interaction with the website. By analyzing key variables such as TimeSpentOnWebsite, DiscountsAvailed, and demographic data, the goal is to develop a model that can identify high-potential customers and optimize conversion strategies.

# Dataset

The dataset has 10 columns and 15,000 rows consisting of 

- Age, 
- Gender, 
- Annual income, 
- NumberOfpurchases, 
- productCatergory, 
- TimespentonWebsite, 
- Loyalty Program,
- discounts Avail, 
- Purchase Status

# Preparation 

The initial data cleaning was conducted in Power BI to ensure accurate representation and facilitate effective visualizations. Several transformations were applied:

Gender: The gender column was duplicated and converted from binary values to text, where 1 was changed to "Male" and 0 to "Female."

Product Category: The product category column was also duplicated and reformatted from numeric values to text labels:

0 = Electronics
1 = Clothing
2 = Home Goods
3 = Beauty
4 = Sports

**Time Spent on Website**: The TimeSpentOnWebsite column, represented in minutes, was rounded to make the values more interpretable. This column was not duplicated.

**Loyalty Program:** The loyalty program column was duplicated and reformatted from binary values to text, where 0 represents "No" (not part of the program) and 1 represents "Yes" (part of the program).

**Purchase Status:** The purchase status column was duplicated and converted from numeric values to text, where 1 is "Made Purchase" and 0 is "No Purchase."

**Handling Missing Values:** Missing values in the TimeSpentOnWebsite column were imputed using the median value. The DiscountsAvailed column was filled with 0, assuming that missing values represent no discounts availed.

**Feature Engineering:**
An interaction term was created between TimeSpentOnWebsite and DiscountsAvailed to capture the combined impact on purchasing behavior.

**Age Grouping:** The Age variable was binned into meaningful categories for analysis ("18-24", "25-34", "35-45", "46-55", 55+)


These changes were necessary to ensure that the dataset was properly structured for creating accurate visualizations in Power BI, and to perform aggregate functions needed for analysis across the correct categories.  

# Limitations

**Lack** of Product-Level Details:
The dataset does not contain detailed information about individual products purchased, such as price, quantity, or specific product attributes. This limits the depth of analysis that can be performed regarding product preferences and sales patterns.

**Potential** Data Imbalance:
There could be an imbalance in the PurchaseStatus variable (more 'No Purchase' entries compared to 'Made Purchase'), which might skew the model's predictions and affect the accuracy of insights. Handling this imbalance through techniques like resampling may be required.

**Limited** Discount Information:**
The dataset includes a column for DiscountsAvailed, but it does not specify the types or amounts of discounts applied. Without this information, it’s challenging to analyze the true impact of specific discount strategies on purchasing behavior.

**Simplified** Demographic Data:**
The dataset includes only basic demographic data such as Age and Gender. Other potentially impactful factors like, geographic location, or marital status are missing, which may limit the analysis of customer segmentation and behavior trends.

**Time-Spent Bias:**
The TimeSpentOnWebsite variable might not always correlate with purchasing intent. Users may spend extended periods on the website without buying anything, skewing the analysis of engagement versus conversion. Identifying which pages were visited would help refine the analysis.

**Static Snapshot of Data:**
If this dataset represents a static snapshot in time, it may not account for changes in customer behavior due to external factors such as seasonality, promotions, or market trends. A more dynamic, time-series dataset would be more useful for understanding trends over time.

**No Information on Returning Customers:**
The dataset does not differentiate between first-time and returning customers. This limits the ability to assess the effectiveness of loyalty programs or customer retention strategies.

**No** Transaction-Specific Data:
While the NumberOfPurchases column provides some information, the dataset lacks specific transaction data (e.g., order date, total amount spent per transaction), limiting the ability to analyze buying patterns more thoroughly.

# Analysis 

      data <- read.csv("customer_purchase_data.csv")
### Convert data types
      data$Age <- as.integer(data$Age)
      data$Gender <- factor(data$Gender, levels = c(0, 1), labels = c("Female", "Male"))
      data$AnnualIncome <- as.numeric(data$AnnualIncome)
      data$NumberOfPurchases <- as.integer(data$NumberOfPurchases)
      data$ProductCategory <- factor(data$ProductCategory, levels = c(0, 1, 2, 3, 4), 
                                labels = c("Electronics", "Clothing", "Home Goods", "Beauty", "Sports"))
      data$TimeSpentOnWebsite <- as.numeric(data$TimeSpentOnWebsite)
      data$LoyaltyProgram <- factor(data$LoyaltyProgram, levels = c(0, 1), labels = 
      c("No", "Yes"))
      data$DiscountsAvailed <- as.integer(data$DiscountsAvailed)
      data$PurchaseStatus <- as.numeric(data$PurchaseStatus)  # Used as numeric for 
      correlation,

### Distribution of Time Spent on Website: 
      ggplot(data, aes(x=TimeSpentOnWebsite)) + geom_histogram(binwidth=5, fill="skyblue", color="black") + theme_minimal()

### Time Spent on Website vs Purchase Probability:
      ggplot(data, aes(x=TimeSpentOnWebsite, y=PurchaseStatus)) + geom_point() + 
      geom_smooth(method="glm", method.args=list(family="binomial"), se=FALSE)

### Impact of Discounts on Purchase Status:
      ggplot(data, aes(x=DiscountsAvailed, y=PurchaseStatus)) + geom_point() + 
      geom_smooth(method="glm", method.args=list(family="binomial"), se=FALSE)

### Predictive Modeling
A logistic regression model was used to predict the PurchaseStatus based on TimeSpentOnWebsite, DiscountsAvailed, and their interaction.
       
       model <- glm(PurchaseStatus ~ TimeSpentOnWebsite + DiscountsAvailed+ 
       TimeSpent_DiscountInteraction, data=trainData, family=binomial)
       summary(model)

- The model was trained on 80% of the data and tested on the remaining 20%.
- The confusion matrix was calculated to evaluate model performance:



      confusion_matrix <- table(Predicted = predicted_classes, Actual = 
      testData$PurchaseStatus)
      accuracy <- sum(diag(confusion_matrix)) / sum(confusion_matrix)
      precision <- confusion_matrix[2,2] / sum(confusion_matrix[2,])
      recall <- confusion_matrix[2,2] / sum(confusion_matrix[,2])

**Key Metrics**
- Accuracy: 72%
- Precision: 78%
- Recall: 64%


### ROC Curve and AUC:
The AUC-ROC curve was plotted to visualize the performance of the logistic regression model. The AUC (Area Under Curve) was calculated to be approximately 0.81, indicating a strong predictive performance.

      pred <- prediction(predictions, testData$PurchaseStatus)
      perf <- performance(pred, measure = "tpr", x.measure = "fpr")
      plot(perf, main="ROC Curve")
      auc <- performance(pred, measure = "auc")
      auc_value <- auc@y.values[[1]]
      print(auc_value)

    ### Key Insights from R Analysis

**High-Engagement Users:**

- TimeSpentOnWebsite was a significant predictor of purchase behavior. Customers who spent more time on the website had a higher likelihood of making a purchase. This finding supports the idea that engagement correlates strongly with conversion rates.

**Discount Influence:**

- DiscountsAvailed played an important role in increasing the likelihood of purchases, particularly when combined with longer website engagement. Customers who spent more time on the website and took advantage of discounts were more likely to make a purchase.

**Age Group Behavior:**

- The analysis revealed that customers aged 25-34 spent more time on the website, and this age group had the highest likelihood of making a purchase. Marketing efforts should be tailored to target this demographic more effectively.

**Logistic Regression Performance:**

- The logistic regression model performed well with an AUC-ROC of 0.81, indicating a strong predictive ability. The model can reliably predict whether a customer is likely to make a purchase based on time spent on the website, discounts availed, and their interaction.

### Key Insights from Power BI Visualizations

**Loyalty** Program:**

- Contrary to expectations, customers who were not part of the loyalty program made more purchases than those enrolled. This suggests that factors outside of loyalty rewards, such as discounts or product-specific preferences, might be driving purchasing behavior.

**Age and Category Preferences:**

- 18-year-old males had the highest number of purchases in the Beauty category, while 19-year-old females led in the Home Goods category. These demographic and category-specific preferences could guide personalized marketing strategies for younger customers.

**Older Users' Dominance:**

- Users aged 50 to 68 dominated most metrics, including the number of purchases, time spent on the website, and category engagement. This highlights the importance of targeting this demographic with tailored offers and website experiences.

**Gender-Based Variances:**

- There were slight gender-based differences in time spent on certain categories, particularly the Beauty category, where females tended to spend slightly more time on the website. Understanding these small variances can help refine category-specific marketing strategies for different genders.


The combined insights from the R analysis and Power BI visualizations reveal key behaviors that can help optimize customer engagement and conversion strategies. The strong role of time spent on the website, discounts, and age-group-specific behavior were consistent across both analyses. Additionally, insights from Power BI point to unique trends, such as the surprising behavior of non-loyalty members and demographic-specific category preferences. These insights should guide targeted marketing campaigns and business decisions to improve customer conversions.  

# Visualizations

### Power BI

https://app.powerbi.com/view?r=eyJrIjoiOWNlNzU4ZWItZmZkNy00NmVkLTlmZjMtYjQ5Njc3YzI2MGI3IiwidCI6ImE0MzM3ZTM0LTk0MjktNDQxNS05YjljLTJjNTQ3NmQzYWY1ZSIsImMiOjF9&pageName=c8930dc39c87056b6ad9
