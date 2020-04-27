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

- In this challenge, I change the first value of `FIELD_52` and `FIELD_53` because it doesn't follow the pattern of the column. Same as `FIELD_56` and `FIELD_57` but the last value instead of the first value.

- I also drop `FIELD_7` due to its complexity and `FIELD_32` due to its low importance information.

- `FIELD_35` is encoded from ordinal feature to numerical feature.

- `FIELD_36` and `FIELD_31` have string data type, but the meaning is TRUE/FALSE, so I convert them into boolean/logical data type.

[![handmade-de.png](https://i.postimg.cc/13H6vmYs/handmade-de.png)](https://postimg.cc/rDDKmkkP)


- The algorithm that has the most efficient performance is IV/WOE (you can see more about that [here](https://www.kaggle.com/pavansanagapati/weight-of-evidence-woe-information-value-iv))

[![iv-woe-de.png](https://i.postimg.cc/9FJR58Cj/iv-woe-de.png)](https://postimg.cc/Vd0LXFJ7)
