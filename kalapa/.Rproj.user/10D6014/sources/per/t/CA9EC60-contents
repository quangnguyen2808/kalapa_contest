# Clear work space:
rm(list = ls())

# Load data:
library(tidyverse)
library(dplyr)

df_train <- read_csv("train.csv")
df_test <- read_csv("test.csv")

f5352 <- function(x){
  return (replace(x, sort(unique(x))[1], sort(unique(x))[2]))
}

f5657 <- function(x){
  return (replace(x, sort(unique(x))[length(sort(unique(x)))], sort(unique(x))[length(sort(unique(x)))-1]))
}

f35 <- function(x){
  x[x == "Zezo" | x == "Zero"] = 0
  x[x == "I" | x == "One" | x == "A"] = 1
  x[x == "II" | x == "Two" | x == "B"] = 2
  x[x == "III" | x == "Three" | x == "C"] = 3
  x[x == "IV" | x == "Four" | x == "D"] = 4
  x[x == "V" | x == "Five"] = 5
  x[x == "None"] = -1
  return(as.numeric(x))
}

df_train %>% 
  mutate_if(is.character, function(x) {str_to_lower(x)}) %>%
  mutate_at(c("FIELD_53", "FIELD_52"), f5352) %>%
  mutate_at(c("FIELD_56","FIELD_57"), f5657) %>%
  mutate(FIELD_31 = as.logical(FIELD_31)) -> df_train

df_test %>%
  mutate_if(is.character, function(x) {str_to_lower(x)}) %>%
  mutate_at(c("FIELD_53", "FIELD_52"), f5352) %>%
  mutate_at(c("FIELD_56", "FIELD_57"), f5657) %>%
  mutate_at(c("FIELD_36", "FIELD_31"), function(x){as.logical(x)}) -> df_test

df_train = select(df_train,-c(FIELD_7, FIELD_32))
df_test = select(df_test,-c(FIELD_7, FIELD_32))
df_train$FIELD_35 <- f35(df_train$FIELD_35)
df_test$FIELD_35 <- f35(df_test$FIELD_35)

# Conduct binning variables: 
library(scorecard)

# Generates optimal binning for all variables/features: 
bins_var <- woebin(df_train %>% select(-id), y = "label", no_cores = 8, positive = "label|1", check_cate_num = FALSE)

# IV for variables/features: 

do.call("rbind", bins_var) %>% 
  as.data.frame() %>% 
  filter(!duplicated(variable)) %>% 
  rename(iv_var = total_iv) %>% 
  arrange(iv_var) %>% 
  mutate(variable = factor(variable, levels = variable)) -> iv_values

# Features have IV >= 0: 

iv_values %>% 
  filter(iv_var >= 0) %>% 
  pull(variable) %>% 
  as.character() -> var_IV_10


# Conduct data transformation based on IV/WoE and filter features with IV > 0.1: 

train_woe <- woebin_ply(df_train %>% select(-id), bins_var) %>% 
  as.data.frame() %>% 
  select(c("label", paste0(var_IV_10, "_", "woe")))


# Data transformation for actual test data: 

test_woe <- woebin_ply(df_test %>% select(-id), bins_var) %>% 
  as.data.frame() %>% 
  select(paste0(var_IV_10, "_", "woe"))

# A function imputes NA observations: 

replace_na_categorical <- function(x, y) {
  
  y %>% 
    table() %>% 
    as.data.frame() %>% 
    arrange(-Freq) -> my_df
  
  n_obs <- sum(my_df$Freq)
  
  pop <- my_df$. %>% as.character() %>% as.numeric()
  
  set.seed(29)
  
  x[is.na(x)] <- sample(pop, sum(is.na(x)), replace = TRUE, prob = my_df$Freq)
  
  return(x)
}

# Use the function: 

test_woe %>% 
  mutate(FIELD_13_woe = replace_na_categorical(FIELD_13_woe, train_woe$FIELD_13_woe)) %>% 
  mutate(maCv_woe = replace_na_categorical(maCv_woe, train_woe$maCv_woe)) %>% 
  mutate(district_woe = replace_na_categorical(district_woe, train_woe$district_woe)) %>% 
  mutate(FIELD_41_woe = replace_na_categorical(FIELD_41_woe, train_woe$FIELD_41_woe)) %>% 
  mutate(FIELD_10_woe = replace_na_categorical(FIELD_10_woe, train_woe$FIELD_10_woe)) %>% 
  mutate(FIELD_39_woe = replace_na_categorical(FIELD_39_woe, train_woe$FIELD_39_woe)) %>% 
  mutate(FIELD_11_woe = replace_na_categorical(FIELD_11_woe, train_woe$FIELD_11_woe)) %>%
  mutate(FIELD_9_woe = replace_na_categorical(FIELD_9_woe, train_woe$FIELD_9_woe)) %>% 
  mutate(FIELD_12_woe = replace_na_categorical(FIELD_12_woe, train_woe$FIELD_12_woe)) -> test_woe_imputed

#======================================================
# Attempt 4: Default Random Forest with Scaled Data
#======================================================

# For convinience, convert binary target variable to factor: 

train_woe %>% 
  mutate(label = case_when(label == 1 ~ "Bad", TRUE ~ "Good")) %>% 
  mutate(label = as.factor(label)) -> df_forGBM 

# Train Random Forest: 
library(ranger)
RF_default <- ranger(label ~ ., data = df_forGBM, num.trees = 750, probability = TRUE)

# Use the RF Classifier for predicting PD (Probability of Default): 
pd_sub_RF <- predict(RF_default, test_woe_imputed, type = "response")

# Save results for submission: 
pd_sub_RF$predictions %>% as.data.frame() %>% pull(Bad) -> pd_sub_RF
df_sub <- data.frame(id = 30000:49999, label = pd_sub_RF)
write_csv(df_sub, "submit.csv")