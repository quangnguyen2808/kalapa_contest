# Kalapa Contest

Kalapa contest is a challenge about credit score on aivivn.vn

## Getting Started

You can get the Kalapa contest information [here](https://challenge.kalapa.vn/?fbclid=IwAR04BPNrO_LKLqTBwnQM_b-R45b1NnTQHSoGNnNNqRR1mkDoOMiAB5sJayw)

You can get access to the data and its description [here](https://drive.google.com/drive/folders/1uxaWbSdC5u3IEN9uSXBvh-ZKslcTlGo-)

### Prerequisites

* RStudio (recommended IDE)

* tidyverse

* dplyr

* scorecard

* ranger

### Structures

The kalapa folder contains the R project. 

You just need to focus on kalapa_script.R, this is the source code file. Open this in RStudio.

train.csv and test.csv are train and test dataset

## Installation

### Clone

* Clone this repo to your local machine using `git clone https://github.com/quangnguyen2808/kalapa_contest.git`

### Setup

Open RStudio and install packages in Console:

```
> install.packages("ranger")
> install.packages("tidyverse")
> install.packages("scorecard")
> install.packages("dplyr")
```

## Tests

What I do:

- Change the first value of `FIELD_52` and `FIELD_53`.

- Change last value of `FIELD_56` and `FIELD_57`.

- Drop `FIELD_7` and `FIELD_32`.

- Ordinal to numerical: `FIELD_35`

- String to Boolean: `FIELD_36` and `FIELD_31`.

```
fix_first_value <- function(x){
  return (replace(x, sort(unique(x))[1], sort(unique(x))[2]))
}

fix_last_value <- function(x){
  return (replace(x, sort(unique(x))[length(sort(unique(x)))], sort(unique(x))[length(sort(unique(x)))-1]))
}

ordinal_encode <- function(x){
  x[x == "Zero"] = 0
  x[x == "One"] = 1
  x[x == "Two"] = 2
  x[x == "Three"] = 3
  x[x == "Four"] = 4
  x[x == "Five"] = 5
  x[x == "None"] = -1
  return(as.numeric(x))
}

df_train = select(df_train,-c(FIELD_7, FIELD_32))
df_train$FIELD_35 <- ordinal_encode(df_train$FIELD_35)

df_train %>% 
  mutate_if(is.character, function(x) {str_to_lower(x)}) %>%
  mutate_at(c("FIELD_53", "FIELD_52"), fix_first_value) %>%
  mutate_at(c("FIELD_56","FIELD_57"), fix_last_value) %>%
  mutate(FIELD_31 = as.logical(FIELD_31)) -> df_train

df_test = select(df_test,-c(FIELD_7, FIELD_32))
df_test$FIELD_35 <- ordinal_encode(df_test$FIELD_35)

df_test %>%
  mutate_if(is.character, function(x) {str_to_lower(x)}) %>%
  mutate_at(c("FIELD_53", "FIELD_52"), fix_first_value) %>%
  mutate_at(c("FIELD_56", "FIELD_57"), fix_last_value) %>%
  mutate_at(c("FIELD_36", "FIELD_31"), function(x){as.logical(x)}) -> df_test
```
- IV/WOE all features (you can see more about that [here](https://www.kaggle.com/pavansanagapati/weight-of-evidence-woe-information-value-iv))

```
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
```
