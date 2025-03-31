---
title: "Homework 1, Applied Methods in Statistics, Spring 2025"
output: 
  html_document:
    keep_md: true
date: "Due on 2025/3/19"
---

# Problem 1

This problem uses data to answer specific question about global health and economics. The data contradicts commonly held preconceived notions. For example, for each of the six pairs of countries below, which country do you think had the highest child mortality in 2015? 

1. Sri Lanka or Turkey
2. Poland or South Korea
3. Malaysia or Russia
4. Pakistan or Vietnam
5. Thailand or South Africa

Most people get them wrong. Why is this? In part it is due to our preconceived notion that the world is divided into two groups: the
_Western world_ versus the _third world_, characterized by "long life,small family" and "short life, large family" respectively. In this problem we will use data visualization to gain insights on this topic.  

## Problem 1.1

The first step in our analysis is to download and organize the data. 

### Problem 1.1.1

We will use the following datasets:

1.  Childhood mortality: "child_mortality.csv"
2.  Life expectancy: "life_expectancy.csv"
3.  Fertility: "fertility.csv"
4.  Population: "population.csv"
5.  Total GDP: "gdp.csv"

Download these csv files from e3 under "Homework 1". Create five `tibble` table objects, one for each of the tables provided in the above files. 


``` r
library(tidyverse)

t1 <- read_csv("child_mortality.csv")
t2 <- read_csv("life_expectancy.csv")
t3 <- read_csv("fertility.csv")
t4 <- read_csv("population.csv")
t5 <- read_csv("gdp.csv")
```

### Problem 1.1.2

Write a function called `my_func` that takes a table as an argument and returns the column name. For each of the five tables, what is the name of the column containing the country names? 


``` r
my_func <- function(x){
  names(x)[1]
}

answer <- c( my_func(t1), my_func(t2), my_func(t3), my_func(t4), my_func(t5) )

answer
```

```
## [1] "Under five mortality"    "Life expectancy"        
## [3] "Total fertility rate"    "Total population"       
## [5] "GDP (constant 2000 US$)"
```

### Problem 1.1.3 

In the previous problem we noted that these datasets are inconsistent in naming their country column. Fix this by assigning a common name `country` to this column in the various tables.


``` r
the_name <- "country"
names(t1)[1] <- the_name
names(t2)[1] <- the_name
names(t3)[1] <- the_name
names(t4)[1] <- the_name
names(t5)[1] <- the_name
```

### Problem 1.1.4 

Notice that in these tables, years are represented by columns. We want to create a tidy dataset in which each row is a unit or observation and our 5 values of interest, including the year for that unit, are in the columns. The unit here is a country/year pair.

We call this the _long_ format. Use the `pivot_longer` function from the `tidyr` package to create a new table for childhood mortality using the long format. Call the new columns `year` and `child_mortality`.


``` r
pivot_longer(data=t1, cols=!country, names_to="year", values_to="child_mortality")
```

```
## # A tibble: 59,184 × 3
##    country  year  child_mortality
##    <chr>    <chr>           <dbl>
##  1 Abkhazia 1800               NA
##  2 Abkhazia 1801               NA
##  3 Abkhazia 1802               NA
##  4 Abkhazia 1803               NA
##  5 Abkhazia 1804               NA
##  6 Abkhazia 1805               NA
##  7 Abkhazia 1806               NA
##  8 Abkhazia 1807               NA
##  9 Abkhazia 1808               NA
## 10 Abkhazia 1809               NA
## # ℹ 59,174 more rows
```

Now redefine the remaining tables in this way.


``` r
t1 <- pivot_longer(data=t1, cols=!country, names_to="year", values_to="child_mortality")
t2 <- pivot_longer(data=t2, cols=!country, names_to="year", values_to="life_expectancy")
t3 <- pivot_longer(data=t3, cols=!country, names_to="year", values_to="fertility")
t4 <- pivot_longer(data=t4, cols=!country, names_to="year", values_to="population")
t5 <- pivot_longer(data=t5, cols=!country, names_to="year", values_to="gdp")
```

### Problem 1.1.5

Now we want to join all these files together. Make one consolidated table containing all the columns.


``` r
dat <- t1 %>% full_join(t2) %>% full_join(t3) %>% full_join(t4) %>% full_join(t5)
```

### Problem 1.1.6

We have created a file "continent-info.csv" (from e3 under "Homework 1") that maps countries to continents. Add a column to the consolidated table containing the continent for each country. 


``` r
map <- read_csv("continent-info.csv")
dat <- left_join(dat, map, by="country")

summary(dat)
```

```
##    country              year           child_mortality life_expectancy
##  Length:59836       Length:59836       Min.   :  1.9   Min.   : 1.00  
##  Class :character   Class :character   1st Qu.:132.2   1st Qu.:30.80  
##  Mode  :character   Mode  :character   Median :357.4   Median :34.95  
##                                        Mean   :288.2   Mean   :42.65  
##                                        3rd Qu.:420.0   3rd Qu.:55.17  
##                                        Max.   :756.3   Max.   :84.80  
##                                        NA's   :19080   NA's   :15989  
##    fertility       population             gdp             continent        
##  Min.   :0.840   Min.   :5.000e+01   Min.   :1.030e+07   Length:59836      
##  1st Qu.:4.620   1st Qu.:1.420e+05   1st Qu.:1.527e+09   Class :character  
##  Median :5.900   Median :2.304e+06   Median :6.774e+09   Mode  :character  
##  Mean   :5.397   Mean   :1.808e+07   Mean   :1.406e+11                     
##  3rd Qu.:6.580   3rd Qu.:9.553e+06   3rd Qu.:4.899e+10                     
##  Max.   :9.220   Max.   :1.376e+09   Max.   :1.170e+13                     
##  NA's   :16424   NA's   :39721       NA's   :51863
```

``` r
# Write the tibble table object dat to a csv file
write_csv(dat, file="dat.csv")
```

Now, use the tibble table object `dat` for the following analyses.

## Problem 1.2

(10 points) Report the child mortalilty rate in 2015 for these 5 pairs:

1. Sri Lanka or Turkey
2. Poland or South Korea
3. Malaysia or Russia
4. Pakistan or Vietnam
5. Thailand or South Africa


``` r
library(dplyr)

data_all <- read_csv("dat.csv", show_col_types = FALSE)
data_2015 <- data_all %>% filter(year == "2015")

country_pairs <- list(
  c("Sri Lanka", "Turkey"),
  c("Poland", "South Korea"),
  c("Malaysia", "Russia"),
  c("Pakistan", "Vietnam"),
  c("Thailand", "South Africa")
)

results <- data.frame(
  country1 = character(),
  child_mortality1 = numeric(),
  country2 = character(),
  child_mortality2 = numeric(),
  difference = numeric(),
  stringsAsFactors = FALSE
)

for (pair in country_pairs) {
  temp <- data_2015 %>% 
    filter(country %in% pair) %>% 
    select(country, child_mortality)
  
  if(nrow(temp) == 2) {
    diff_val <- abs(temp$child_mortality[1] - temp$child_mortality[2])
    results <- rbind(results, data.frame(
      country1 = temp$country[1],
      child_mortality1 = temp$child_mortality[1],
      country2 = temp$country[2],
      child_mortality2 = temp$child_mortality[2],
      difference = diff_val,
      stringsAsFactors = FALSE
    ))
  } else {
    warning("配對中未找到兩個國家：", paste(pair, collapse = ", "))
  }
}

print(results)
```

```
##       country1 child_mortality1 country2 child_mortality2 difference
## 1    Sri Lanka              8.7   Turkey             13.5        4.8
## 2  South Korea              3.5   Poland              5.2        1.7
## 3     Malaysia              8.2   Russia              9.6        1.4
## 4     Pakistan             81.1  Vietnam             21.7       59.4
## 5 South Africa             42.1 Thailand             12.3       29.8
```

## Problem 1.3

To examine if in fact there was a long-life-in-a-small-family and short-life-in-a-large-family dichotomy,  we will visualize the average number of children per family (fertility) and the life expectancy for each country.

### Problem 1.3.1 

(10 points) Use `ggplot2` to create a plot of life expectancy versus fertiltiy for 1962 for Africa, Asia, Europe, and the Americas. Use color to denote continent and point size to denote population size:


``` r
# Your code goes here.
library(tidyverse)
library(ggplot2)
library(gapminder)

data_temp <- data_all %>% filter(year == "1962") 
data_1962 <- data_temp %>% filter(continent %in% c("Africa", "Asia", "Europe", "Americas"))
data_1962 <- data_1962 %>% drop_na(life_expectancy, fertility, population)
scatter_plot <- ggplot(data_1962, aes(x = life_expectancy, y = fertility, colour = continent, size = population)) + geom_point() + labs(x = "Life Expectancy", y = "Fertility", colour = "Continent", size="Population", title = "1962: Life Expectancy vs. Fertility")
scatter_plot
```

![](hwk01_new_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

(5 points) Do you see a dichotomy? Explain. 

```
- Europe(紫色): 低生育率+高預期壽命 => 有較多發展成熟或已開發國家
- Africa(紅色): 高生育率+低預期壽命 => 有較多發展中國家
- long-life-in-a-small-family 和 short-life-in-a-large-family 的對比顯示出各地區在經濟、教育、醫療等方面的落差，形成二分現象。
```

### Problem 1.3.2

(10 points) Explore how the figure in Problem 1.3.1 changes across time. Show us figures for years 1960, 1980, 2000, and 2015 to demonstrate how this figure changes through time.


``` r
# Your code goes here.
library(tidyverse)
library(ggplot2)

years_to_plot <- c("1960", "1980", "2000", "2015")
data_time <- data_all %>% 
  filter(year %in% years_to_plot, 
         continent %in% c("Africa", "Asia", "Europe", "Americas"))
data_time <- data_time %>% mutate(year = factor(year, levels = years_to_plot))
data_time <- data_time %>% drop_na(life_expectancy, fertility, population)
time_plot <- ggplot(data_time, aes(x = life_expectancy, y = fertility, colour = continent, size = population)) + geom_point(alpha = 0.7) + facet_wrap(~ year) + labs(x = "Life Expectancy", 
       y = "Fertility", 
       colour = "Continent", 
       size = "Population", 
       title = "Life Expectancy vs. Fertility (1960, 1980, 2000, 2015)") + theme_minimal()

time_plot
```

![](hwk01_new_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

(5 points) Would you say that the same dichotomy exists in 2015? Explain:

```
在 2015 年的圖中，儘管多數國家相對於 1960 年有了更高的壽命與更低的生育率，但仍可觀察到兩極分化的趨勢：
- 右下：已開發或發展程度較高的國家，壽命長、生育率低；
- 左上：經濟與醫療水平仍較落後的國家，壽命偏短、生育率較高。
相較於 1960 年，這種二分dichotomy已經不那麼極端，原先處於高生育率、低壽命的國家逐漸往中間區域移動，2015 年依然能看到兩端的存在，但整體更集中於中高壽命、較低生育率的區域。
```

## Problem 1.4 

Having time as a third dimension made it somewhat difficult to see specific country trends. Let's now focus on specific countries.

### Problem 1.4.1

(10 points) Let's compare France and its former colony Tunisia. Make a plot of fertility versus year with color denoting the country. Do the same for life expecancy. How would you compare Tunisia's improvement compared to France's from 1955 to 2015? Hint: use `geom_line`
 

``` r
library(tidyverse)
library(ggplot2)
library(gapminder)

data_fertility <- data_all %>%filter(country %in% c("Tunisia", "France")) %>% filter(as.numeric(year) >= 1955, as.numeric(year) <= 2015)

line_plot <- ggplot(data_fertility, aes(x = as.numeric(year), y = fertility, colour = country)) + geom_line() + labs(x = "Year", y = "Fertility", colour = "Country", title = "Fertility Trends: Tunisia vs. France (1955 - 2015)")

line_plot
```

![](hwk01_new_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

```
# 生育率（Fertility）

- 突尼西亞：1950–60 年代起點較高（約 6～7 之間），之後經過幾十年顯著的快速下降，到 2015 年時已逼近 2 左右。
- 法國：1955 年時生育率相對低很多（約 2～3），在之後的 60 年間只經歷了小幅波動或略微下降，維持在大約 2 左右。
- 整體差異：突尼西亞的生育率變化幅度更大，從高水準迅速降到與法國相近的水準，可推測該國在教育普及、婦女就業率提升、都市化與家庭計劃等方面的快速發展。
```


``` r
library(tidyverse)
library(ggplot2)
library(gapminder)

data_life <- data_all %>% filter(country %in% c("Tunisia", "France")) %>% filter(as.numeric(year) >= 1955, as.numeric(year) <= 2015)

line_plot <- ggplot(data_life, aes(x = as.numeric(year), y = life_expectancy, colour = country)) + geom_line() + labs(x = "Year", y = "life expecancy", colour = "Country", title = "life expecancy Trends: Tunisia vs. France (1955 - 2015)")

line_plot
```

![](hwk01_new_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

```
# 預期壽命（Life Expectancy）

- 突尼西亞：1955 年起點相對較低（大約 45～50 歲），但在隨後幾十年裡，曲線呈現非常明顯的上升，到 2015 年時已經超過 70 歲。
- 法國：1955 年時就有大約 65 歲以上的平均壽命，此後雖持續上升，但幅度較為平緩，最終在 2015 年時達到 80 歲上下。
- 整體差異：突尼西亞在這段期間內的「壽命延長」幅度更大，可推測在醫療衛生、基礎設施、生活條件等方面的大幅改善；法國則從相對高的基礎上，持續穩定成長。
```

## Problem 1.5

We are now going to examine "GDP per capita per day", which is defined as `gdp / population / 365`.

### Problem 1.5.1

(10 points) Create a smooth density estimate of the distribution of GDP per capita per day across countries in 1970. Include Africa, Asia, Europe, and the Americas in the computation. When doing this we want to weigh countries with larger populations more. We can do this using the "weight" argument in `geom_density`. 


``` r
# Your code goes here.
library(tidyverse)
library(ggplot2)

data_gdp <- data_all %>% 
  filter(year == "1970", 
         continent %in% c("Africa", "Asia", "Europe", "Americas")) %>% 
  drop_na(gdp, population) %>% 
  mutate(
    gdp = as.numeric(unlist(gdp)),
    population = as.numeric(unlist(population)),
    gdp_capita_day = gdp / population / 365
  )

ggplot(data_gdp, aes(x = gdp_capita_day, weight = population)) +
  geom_density(alpha = 0.7, fill = "skyblue") +
  scale_x_continuous(trans = "log2") +
  labs(
    title = "Weighted Density of GDP per Capita per Day in 1970",
    x = "GDP per Capita per Day (log scale)",
    y = "Density"
  ) +
  theme_minimal()
```

![](hwk01_new_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

### Problem 1.5.2

(10 points) Now do the same but show each of the four groups (Africa, Asia, Europe, and the Americas) separately.


``` r
# Your code goes here.
library(tidyverse)
library(ggplot2)

data_gdp_group <- data_all %>% 
  filter(year == "1970", 
         continent %in% c("Africa", "Asia", "Europe", "Americas")) %>% 
  drop_na(gdp, population) %>% 
  mutate(
    gdp = as.numeric(unlist(gdp)),
    population = as.numeric(unlist(population)),
    gdp_capita_day = gdp / population / 365
  )

ggplot(data_gdp_group, aes(x = gdp_capita_day, weight = population)) +
  geom_density(alpha = 0.7, fill = "skyblue") +
  scale_x_continuous(trans = "log2") +
  facet_wrap(~ continent) +
  labs(
    title = "Weighted Density of GDP per Capita per Day in 1970\nby Continent",
    x = "GDP per Capita per Day (log scale)",
    y = "Density"
  ) +
  theme_minimal()
```

![](hwk01_new_files/figure-html/unnamed-chunk-14-1.png)<!-- -->

### Problem 1.5.3

(10 points) Visualize these densities for several years. Show a couple of of them. Summarize how the distribution has changed through the years.


``` r
# Put your code here.
library(tidyverse)
library(ggplot2)

years_to_plot <- c("1960", "1970", "1980", "2000", "2015")

data_gdp_year <- data_all %>% 
  filter(year %in% years_to_plot, 
         continent %in% c("Africa", "Asia", "Europe", "Americas")) %>% 
  drop_na(gdp, population) %>% 
  mutate(
    gdp = as.numeric(unlist(gdp)),
    population = as.numeric(unlist(population)),
    gdp_capita_day = gdp / population / 365
  )

ggplot(data_gdp_year, aes(x = gdp_capita_day, weight = population)) +
  geom_density(alpha = 0.7, fill = "skyblue") +
  facet_wrap(~ year, scales = "free_x") +
  scale_x_continuous(trans = "log2") +
  labs(
    title = "Weighted Density of GDP per Capita per Day Across Years",
    x = "GDP per Capita per Day (log scale)",
    y = "Density"
  ) +
  theme_minimal()
```

![](hwk01_new_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

```
# 整體分佈右移
早期（如 1960 或 1970 年代）時，主要人口集中在較低的人均 GDP 水準（左側），也就是說全球多數人口當時仍處於相對貧窮的狀態。
隨著時間推進，分佈的高峰（主峰）逐漸往右移，代表越來越多國家（且人口較多的國家）的人均 GDP 增加，整體生活水準有所提升。

# 雙峰（Bimodal）現象的演變
早期分佈往往呈現明顯的「雙峰」或「多峰」：一個峰值落在非常低的 GDP 水準（大量人口處於貧窮），另一個較小或次要的峰值在更高的 GDP 區間（少數已開發國家）。
隨著許多發展中國家（尤其是人口大國，如中國、印度等）經濟成長，左峰向右靠攏，形成更集中或更平滑的分佈。雖然仍可能存在一些較富裕國家構成的「右峰」，但與左峰的距離縮小或峰值差異減弱。
```

# Problem 2

For these exercises, we will be using the dataset "femaleControlsPopulation.csv" (from e3 under "Homework 1"): 


``` r
library(tidyverse)

x <- unlist(read_csv("femaleControlsPopulation.csv"))
```

Here `x` represents the weights for the entire population.

## Problem 2.1

(5 points) What is the average of these weights?


``` r
# Your code goes here.
library(tidyverse)
x <- unlist(read_csv("femaleControlsPopulation.csv"))
avg <- mean(x)
print(avg)
```

```
## [1] 23.89338
```

## Problem 2.2

(5 points) After setting the seed at 1 (use `set.seed(1)`), take a random sample of size 5. What is the absolute value (use `abs`) of the difference between the average of the sample and the average of all the values?


``` r
set.seed(1)
# Your code goes here.
sample <- sample(x, size = 5)
sample_mean <- mean(sample)
overall_mean <- mean(x)
abs_diff <- abs(sample_mean - overall_mean)
print(abs_diff)
```

```
## [1] 0.3293778
```

## Problem 2.3

(5 points) After setting the seed at 5, take a random sample of size 5. What is the absolute value of the difference between the average of the sample and the average of all the values?


``` r
set.seed(5)

# Your code goes here.
sample <- sample(x, size = 5)
sample_mean <- mean(sample)
overall_mean <- mean(x)
abs_diff <- abs(sample_mean - overall_mean)
print(abs_diff)
```

```
## [1] 0.3813778
```

## Problem 2.4

(5 points) Why are the answers from Problems 2.2 and 2.3 different?

a.  Because we made a coding mistake.
b.  Because the average of the `x` is random.
c.  Because the average of the samples is a random variable.
d.  All of the above.

>> Answer: c.

## Problem 2.5

(5 points) Set the seed at 1, then using a for-loop take a random sample of 5 mice 1,000 times. Save these averages. What percent of these 1,000 averages are more than 1 ounce away from the average of `x` (i.e., `> ( mean(x)+1 )`)?


``` r
set.seed(1)

# Your code goes here.

# Your code goes here.

n=1000
sample_means <- numeric(n)

for(i in 1:n){
  sample_means[i]=mean(sample(x,size=5))
}
overall_mean= mean(x)
percent_exceed <- mean(abs(sample_means - overall_mean) > 1) * 100
print(percent_exceed)
```

```
## [1] 50.3
```

## Problem 2.6

(5 points) We are now going to increase the number of times we redo the sample from 1,000 to 10,000. Set the seed at 1, then using a for-loop take a random sample of 5 mice 10,000 times. Save these averages. What percent of these 10,000 averages are more than 1 ounce away from the average of `x`?


``` r
set.seed(1)

# Your code goes here.

n=10000
sample_means <- numeric(n)

for(i in 1:n){
  sample_means[i]=mean(sample(x,size=5))
}
overall_mean= mean(x)
percent_exceed <- mean(abs(sample_means - overall_mean) > 1) * 100
print(percent_exceed)
```

```
## [1] 50.84
```

Note that the answers to Problems 2.5 and 2.6 barely changed. This is expected. The way we think about the random value distributions is as the distribution of the list of values obtained if we repeated the experiment an infinite number of times. On a computer, we can’t perform an infinite number of iterations so instead, for our examples, we consider 1,000 to be large enough, thus 10,000 is as well. 

## Problem 2.7

(5 points) Now if instead we change the sample size, then we change the random variable and thus its distribution. Set the seed at 1, then using a for-loop take a random sample of 50 mice 1,000 times. Save these averages. What percent of these 1,000 averages are more than 1 ounce away from the average of `x`?


``` r
set.seed(1)

# Your code goes here.

n=1000
sample_means <- numeric(n)

for(i in 1:n){
  sample_means[i]=mean(sample(x,size=50))
}
overall_mean= mean(x)
ans <- mean(abs(sample_means - overall_mean) > 1) * 100
cat(ans, "%\n")
```

```
## 1.4 %
```

## Problem 2.8

(5 points) Use a histogram to “look” at the distribution of averages we get with a sample size of 5 and a sample size of 50. How would you say they differ?

a.  They are actually the same.
b.  They both look roughly normal, but with a sample size of 50 the spread is smaller.
c.  They both look roughly normal, but with a sample size of 50 the spread is larger.
d.  The second distribution does not look normal at all.


``` r
set.seed(1)
# Your code goes here.
n_iter <- 1000

sample_means_5 <- replicate(n_iter, mean(sample(x, 5)))
sample_means_50 <- replicate(n_iter, mean(sample(x, 50)))

df <- data.frame(
  sample_mean = c(sample_means_5, sample_means_50),
  sample_size = factor(rep(c("5", "50"), each = n_iter))
)

ggplot(df, aes(x = sample_mean)) +
  geom_histogram(bins = 30, fill = "skyblue", color = "black") +
  facet_wrap(~ sample_size) +
  labs(x = "Sample Mean", y = "Frequency", title = "Distribution of Sample Means\n(Sample size: 5 vs. 50)")
```

![](hwk01_new_files/figure-html/unnamed-chunk-23-1.png)<!-- -->

>> Answer: b.

## Problem 2.9

(5 points) For the last set of averages, the ones obtained from a sample size of 50, what percent are between 23 and 25?


``` r
# Your code goes here.
n_iter <- 1000
sample_means_50 <- numeric(n_iter)

for (i in 1:n_iter) {
  sample_means_50[i] <- mean(sample(x, size = 50))
}

percent_between <- mean(sample_means_50 >= 23 & sample_means_50 <= 25) * 100
print(percent_between)
```

```
## [1] 98.1
```

## Problem 2.10

(5 points) Now ask the same question of a normal distribution with mean 23.9 and standard deviation 0.48: $\Pr(23 \leq N(\mu=23.9,\sigma=0.48) \leq 25)=?$. 


``` r
# Your code goes here.
prob <- pnorm(25, mean = 23.9, sd = 0.48) - pnorm(23, mean = 23.9, sd = 0.48)
print(prob)
```

```
## [1] 0.9586412
```

## Problem 2.11

(5 points) The answer to Problems 2.9 and 2.10 were very similar. This is because we can approximate the distribution of the sample average with a normal distribution. Use an q-q plot to verify this.


``` r
# Your code goes here.
qqnorm(sample_means_50, main = "Q-Q Plot of Sample Means (n = 50)")
qqline(sample_means_50, col = "red")
```

![](hwk01_new_files/figure-html/unnamed-chunk-26-1.png)<!-- -->


