# Telecom Customer Churn Analysis in R
# NOTE: written to mirror the Python analysis I ran; I could not execute R in my environment, so run once and tell me about any errors.
# install.packages(c("tidyverse","caret","pROC","rpart","randomForest","gbm"))
library(tidyverse); library(caret); library(pROC); library(rpart); library(randomForest); library(gbm)

# 1. Load & clean -------------------------------------------------------------
df <- read.csv("Telcom-Customer-Churn.csv", stringsAsFactors = FALSE)
df$TotalCharges[is.na(df$TotalCharges)] <- 0            # 11 new customers (tenure = 0)
ids <- df$customerID                                    # keep IDs aside; they must not enter the models
df <- df %>% select(-customerID) %>%
  mutate(SeniorCitizen = factor(SeniorCitizen), across(where(is.character), factor))
df$Churn <- relevel(df$Churn, ref = "No")
cat("Churn rate:", mean(df$Churn == "Yes"), "\n")

# 2. Exploratory analysis ---------------------------------------------------------
rate_plot <- function(v) df %>% group_by(.data[[v]]) %>%
  summarise(churn = mean(Churn == "Yes")) %>%
  ggplot(aes(.data[[v]], churn)) + geom_col(fill = "#e76f51") +
  scale_y_continuous(labels = scales::percent) + labs(title = paste("Churn rate by", v), x = NULL)
for (v in c("Contract", "InternetService", "PaymentMethod", "TechSupport", "SeniorCitizen")) print(rate_plot(v))
ggplot(df, aes(tenure, fill = Churn)) + geom_histogram(bins = 24, alpha = .7, position = "identity")
ggplot(df, aes(MonthlyCharges, fill = Churn)) + geom_histogram(bins = 24, alpha = .7, position = "identity")

# 3. Train / test split ---------------------------------------------------------
set.seed(42)
idx   <- createDataPartition(df$Churn, p = 0.8, list = FALSE)
train <- df[idx, ]; test <- df[-idx, ]
levels(train$Churn) <- levels(test$Churn) <- c("No", "Yes")

# 4. Models ------------------------------------------------------------------------
glm_fit <- glm(Churn ~ ., data = train, family = binomial)
print(summary(glm_fit)); print(round(exp(coef(glm_fit)), 2))   # odds ratios
tree_fit <- rpart(Churn ~ ., data = train, method = "class", control = rpart.control(maxdepth = 5, minbucket = 30, cp = 0.001))
rf_fit   <- randomForest(Churn ~ ., data = train, ntree = 400, nodesize = 5, importance = TRUE)
train_gbm <- train %>% mutate(y = as.numeric(Churn == "Yes")) %>% select(-Churn)  # gbm needs a real 0/1 column
gbm_fit  <- gbm(y ~ ., data = train_gbm, distribution = "bernoulli",
                n.trees = 150, interaction.depth = 3, shrinkage = 0.05, cv.folds = 5, verbose = FALSE)

p <- list(
  "Logistic Regression" = predict(glm_fit,  test, type = "response"),
  "Decision Tree"       = predict(tree_fit, test, type = "prob")[, "Yes"],
  "Random Forest"       = predict(rf_fit,   test, type = "prob")[, "Yes"],
  "Gradient Boosting"   = predict(gbm_fit,  test, n.trees = 150, type = "response"))

# 5. Evaluation ----------------------------------------------------------------------
truth <- test$Churn
eval_tbl <- map_dfr(names(p), function(n) {
  cls <- factor(ifelse(p[[n]] >= 0.35, "Yes", "No"), levels = c("No", "Yes"))
  cm  <- confusionMatrix(cls, truth, positive = "Yes")
  tibble(Model = n, AUC = as.numeric(auc(roc(truth, p[[n]], quiet = TRUE))),
         Accuracy = cm$overall["Accuracy"], Precision = cm$byClass["Precision"], Recall = cm$byClass["Recall"])
})
print(eval_tbl)
plot(roc(truth, p[["Gradient Boosting"]]), col = "red", main = "ROC curves")
for (i in 1:3) plot(roc(truth, p[[i]]), add = TRUE, col = i + 1)
varImpPlot(rf_fit); print(summary(gbm_fit, plotit = FALSE))

# 6. Score every customer and build retention tiers --------------------------------
df$churn_prob <- predict(gbm_fit, df, n.trees = 150, type = "response")
df$risk_tier  <- cut(df$churn_prob, c(-Inf, .3, .6, Inf), labels = c("Low", "Medium", "High"))
print(df %>% group_by(risk_tier) %>% summarise(n = n(), actual_churn = mean(Churn == "Yes"), avg_monthly = mean(MonthlyCharges)))
df$customerID <- ids                                    # re-attach IDs so scores map to customers
write.csv(df %>% select(customerID, everything()), "customer_risk_scores_R.csv", row.names = FALSE)