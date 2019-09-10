---
title: "Taming Wild Data"
author: "Data Services @ HSL"
date: "9/6/2019"
output: 
  html_document:
    keep_md: true
---

# Motivation

Data analysis involves a large amount of [janitorwork](http://www.nytimes.com/2014/08/18/technology/for-big-data-scientists-hurdle-to-insights-is-janitor-work.html) -- munging and cleaning data to facilitate downstream data analysis. In fact, data scientists say that around [80%](https://www.infoworld.com/article/3228245/data-science/the-80-20-data-science-dilemma.html) of their time is taken up by data cleaning tasks compared to just 20% for the actual analyses.

Our goal will be to produce "tidy data" that we can then use to derive some insights. [Tidy data](https://vita.had.co.nz/papers/tidy-data.pdf) is defined as:
1. Each variable forms a column.
2. Each observation forms a row.
3. Each type of observational unit forms a table.

Transforming the data into tidy data will take different steps depending on the nature of the untidyness. Hadley Wickham, who works for RStudio, says "tidy datasets are all alike but every messy dataset is messy in its own way."

This lesson demonstrates techniques for data cleaning and manipulation. We will make use of the **tidyr** package to clean the data. The **dplyr** package will help us effectively manipulate and conditionally compute summary statistics over subsets of data, while the **stringr** package will help us interact with string data. 

> Note: This lesson assumes a basic familiarity with R and should not be your first exposure to R and RStudio

# Overall goal

Let's say that you are a tuberculosis researcher and you would like to know the effect of a country's gdp on its tuberculosis incidence. To do this, the goal today will be to merge a dataset with tuberculosis variables to a dataset with gdp. Along the way we will have to clean the datasets to make sure they match up properly.

# Set-up

1. Navigate to the [HSL website](data.hsl.virginia.edu) and download today's files under Workshops --> Workshop Materials --> Taming Wild Data

2. Move those files to a folder somewhere you can find

3. In RStudio, go to File --> New Project. Choose "Existing Directory" and Browse to the folder containing your workshop materials. This creates an instance of RStudio running in the project folder so that your scripts can find your data files.

4. We need to load the readr, dplyr, tidyr, and stringr packages. All of these packages are contained in the tidyverse megapackage. Let's load those packages now - hopefully you have already installed them. If you get an error loading the package, try `install.packages("tidyverse")`.


```r
#install.packages("tidyverse")

# Load packages
library(tidyverse)
```

```
## ── Attaching packages ──────────────────────────────────────── tidyverse 1.2.1 ──
```

```
## ✔ ggplot2 3.2.0     ✔ purrr   0.3.2
## ✔ tibble  2.1.3     ✔ dplyr   0.8.3
## ✔ tidyr   0.8.3     ✔ stringr 1.4.0
## ✔ readr   1.3.1     ✔ forcats 0.4.0
```

```
## ── Conflicts ─────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```

# Read in original datasets


```r
tb <- read_csv("tb.csv")
```

```
## Parsed with column specification:
## cols(
##   country = col_character(),
##   who_region = col_character(),
##   year = col_double(),
##   pop = col_double(),
##   incidence_100k = col_double(),
##   incidence_number = col_double(),
##   hiv_percent = col_double(),
##   hiv_incidence_100k = col_double(),
##   hiv_number = col_double(),
##   mort_nohiv_100k = col_double(),
##   mort_nohiv_number = col_double(),
##   mort_hiv_100k = col_double(),
##   mort_hiv_number = col_double(),
##   mort_100k = col_double(),
##   mort_number = col_double(),
##   case_fatality_ratio = col_double(),
##   new_incidence_100k = col_double(),
##   case_detection_percent = col_double()
## )
```

```r
gm1 <- read_csv("gm_messy1.csv")
```

```
## Parsed with column specification:
## cols(
##   country = col_character(),
##   continent = col_character(),
##   year = col_double(),
##   le_gdp = col_character()
## )
```

```r
gm2 <- read_csv("gm_messy2.csv")
```

```
## Parsed with column specification:
## cols(
##   country = col_character(),
##   continent = col_character(),
##   year.1952 = col_double(),
##   year.1957 = col_double(),
##   year.1962 = col_double(),
##   year.1967 = col_double(),
##   year.1972 = col_double(),
##   year.1977 = col_double(),
##   year.1982 = col_double(),
##   year.1987 = col_double(),
##   year.1992 = col_double(),
##   year.1997 = col_double(),
##   year.2002 = col_double(),
##   year.2007 = col_double()
## )
```

Now let's take a look at each one of these files

```r
tb
```

```
## # A tibble: 3,850 x 18
##    country who_region  year    pop incidence_100k incidence_number
##    <chr>   <chr>      <dbl>  <dbl>          <dbl>            <dbl>
##  1 Afghan… EMR         2000 2.01e7            190            38000
##  2 Afghan… EMR         2001 2.10e7            189            40000
##  3 Afghan… EMR         2002 2.20e7            189            42000
##  4 Afghan… EMR         2003 2.31e7            189            44000
##  5 Afghan… EMR         2004 2.41e7            189            46000
##  6 Afghan… EMR         2005 2.51e7            189            47000
##  7 Afghan… EMR         2006 2.59e7            189            49000
##  8 Afghan… EMR         2007 2.66e7            189            50000
##  9 Afghan… EMR         2008 2.73e7            189            52000
## 10 Afghan… EMR         2009 2.80e7            189            53000
## # … with 3,840 more rows, and 12 more variables: hiv_percent <dbl>,
## #   hiv_incidence_100k <dbl>, hiv_number <dbl>, mort_nohiv_100k <dbl>,
## #   mort_nohiv_number <dbl>, mort_hiv_100k <dbl>, mort_hiv_number <dbl>,
## #   mort_100k <dbl>, mort_number <dbl>, case_fatality_ratio <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>
```

```r
gm1
```

```
## # A tibble: 1,704 x 4
##    country     continent  year le_gdp             
##    <chr>       <chr>     <dbl> <chr>              
##  1 Afghanistan Asia       1952 28.801::779.4453145
##  2 Afghanistan Asia       1957 30.332::820.8530296
##  3 Afghanistan Asia       1962 31.997::853.10071  
##  4 Afghanistan Asia       1967 34.02::836.1971382 
##  5 Afghanistan Asia       1972 36.088::739.9811058
##  6 Afghanistan Asia       1977 38.438::786.11336  
##  7 Afghanistan Asia       1982 39.854::978.0114388
##  8 Afghanistan Asia       1987 40.822::852.3959448
##  9 Afghanistan Asia       1992 41.674::649.3413952
## 10 Afghanistan Asia       1997 41.763::635.341351 
## # … with 1,694 more rows
```

```r
gm2
```

```
## # A tibble: 142 x 14
##    country continent year.1952 year.1957 year.1962 year.1967 year.1972
##    <chr>   <chr>         <dbl>     <dbl>     <dbl>     <dbl>     <dbl>
##  1 Afghan… Asia        8425333   9240934  10267083  11537966  13079460
##  2 Albania Europe      1282697   1476505   1728137   1984060   2263554
##  3 Algeria Africa      9279525  10270856  11000948  12760499  14760787
##  4 Angola  Africa      4232095   4561361   4826015   5247469   5894858
##  5 Argent… Americas   17876956  19610538  21283783  22934225  24779799
##  6 Austra… Oceania     8691212   9712569  10794968  11872264  13177000
##  7 Austria Europe      6927772   6965860   7129864   7376998   7544201
##  8 Bahrain Asia         120447    138655    171863    202182    230800
##  9 Bangla… Asia       46886859  51365468  56839289  62821884  70759295
## 10 Belgium Europe      8730405   8989111   9218400   9556500   9709100
## # … with 132 more rows, and 7 more variables: year.1977 <dbl>,
## #   year.1982 <dbl>, year.1987 <dbl>, year.1992 <dbl>, year.1997 <dbl>,
## #   year.2002 <dbl>, year.2007 <dbl>
```

```r
# Let's spend a little time understanding what each dataset really looks like
glimpse(tb)
```

```
## Observations: 3,850
## Variables: 18
## $ country                <chr> "Afghanistan", "Afghanistan", "Afghanista…
## $ who_region             <chr> "EMR", "EMR", "EMR", "EMR", "EMR", "EMR",…
## $ year                   <dbl> 2000, 2001, 2002, 2003, 2004, 2005, 2006,…
## $ pop                    <dbl> 20093756, 20966463, 21979923, 23064851, 2…
## $ incidence_100k         <dbl> 190, 189, 189, 189, 189, 189, 189, 189, 1…
## $ incidence_number       <dbl> 38000, 40000, 42000, 44000, 46000, 47000,…
## $ hiv_percent            <dbl> 0.36, 0.30, 0.26, 0.23, 0.22, 0.22, 0.22,…
## $ hiv_incidence_100k     <dbl> 0.68, 0.57, 0.49, 0.44, 0.41, 0.42, 0.42,…
## $ hiv_number             <dbl> 140, 120, 110, 100, 100, 100, 110, 120, 1…
## $ mort_nohiv_100k        <dbl> 67.00, 62.00, 56.00, 57.00, 51.00, 46.00,…
## $ mort_nohiv_number      <dbl> 14000, 13000, 12000, 13000, 12000, 12000,…
## $ mort_hiv_100k          <dbl> 0.15, 0.17, 0.27, 0.25, 0.21, 0.19, 0.18,…
## $ mort_hiv_number        <dbl> 31, 35, 60, 57, 50, 48, 46, 45, 48, 55, 5…
## $ mort_100k              <dbl> 67.00, 62.00, 56.00, 57.00, 51.00, 46.00,…
## $ mort_number            <dbl> 14000, 13000, 12000, 13000, 12000, 12000,…
## $ case_fatality_ratio    <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ new_incidence_100k     <dbl> 35, 48, 63, 60, 76, 87, 98, 108, 104, 93,…
## $ case_detection_percent <dbl> 19, 26, 33, 32, 40, 46, 52, 57, 55, 49, 5…
```

```r
glimpse(gm1)
```

```
## Observations: 1,704
## Variables: 4
## $ country   <chr> "Afghanistan", "Afghanistan", "Afghanistan", "Afghanis…
## $ continent <chr> "Asia", "Asia", "Asia", "Asia", "Asia", "Asia", "Asia"…
## $ year      <dbl> 1952, 1957, 1962, 1967, 1972, 1977, 1982, 1987, 1992, …
## $ le_gdp    <chr> "28.801::779.4453145", "30.332::820.8530296", "31.997:…
```

```r
glimpse(gm2)
```

```
## Observations: 142
## Variables: 14
## $ country   <chr> "Afghanistan", "Albania", "Algeria", "Angola", "Argent…
## $ continent <chr> "Asia", "Europe", "Africa", "Africa", "Americas", "Oce…
## $ year.1952 <dbl> 8425333, 1282697, 9279525, 4232095, 17876956, 8691212,…
## $ year.1957 <dbl> 9240934, 1476505, 10270856, 4561361, 19610538, 9712569…
## $ year.1962 <dbl> 10267083, 1728137, 11000948, 4826015, 21283783, 107949…
## $ year.1967 <dbl> 11537966, 1984060, 12760499, 5247469, 22934225, 118722…
## $ year.1972 <dbl> 13079460, 2263554, 14760787, 5894858, 24779799, 131770…
## $ year.1977 <dbl> 14880372, 2509048, 17152804, 6162675, 26983828, 140741…
## $ year.1982 <dbl> 12881816, 2780097, 20033753, 7016384, 29341374, 151842…
## $ year.1987 <dbl> 13867957, 3075321, 23254956, 7874230, 31620918, 162572…
## $ year.1992 <dbl> 16317921, 3326498, 26298373, 8735988, 33958947, 174819…
## $ year.1997 <dbl> 22227415, 3428038, 29072015, 9875024, 36203463, 185652…
## $ year.2002 <dbl> 25268405, 3508512, 31287142, 10866106, 38331121, 19546…
## $ year.2007 <dbl> 31889923, 3600523, 33333216, 12420476, 40301927, 20434…
```

# The pipe: **%>%**

Today we are going to be building up pipelines of functions where each step changes the dataset in some way. We would like to connect functions together in a logical way rather than creating ugly nested code.

The dplyr package imports functionality from the [magrittr](https://github.com/smbache/magrittr) package that lets you _pipe_ the output of one function to the input of another, so you can avoid nesting functions. It looks like this: **`%>%`**. Quick keyboard shortcut is `Command+Shift+M` (mac) or `Control+Shift+M` (pc). You don't have to load the magrittr package to use it since dplyr imports its functionality when you load the dplyr package. 


```r
# one function without the pipe
tail(tb, 20)
```

```
## # A tibble: 20 x 18
##    country who_region  year    pop incidence_100k incidence_number
##    <chr>   <chr>      <dbl>  <dbl>          <dbl>            <dbl>
##  1 Zambia  AFR         2016 1.66e7            376            62000
##  2 Zambia  AFR         2017 1.71e7            361            62000
##  3 Zimbab… AFR         2000 1.22e7            605            74000
##  4 Zimbab… AFR         2001 1.24e7            617            76000
##  5 Zimbab… AFR         2002 1.25e7            617            77000
##  6 Zimbab… AFR         2003 1.26e7            617            78000
##  7 Zimbab… AFR         2004 1.28e7            607            78000
##  8 Zimbab… AFR         2005 1.29e7            588            76000
##  9 Zimbab… AFR         2006 1.31e7            561            74000
## 10 Zimbab… AFR         2007 1.33e7            527            70000
## 11 Zimbab… AFR         2008 1.36e7            487            66000
## 12 Zimbab… AFR         2009 1.38e7            450            62000
## 13 Zimbab… AFR         2010 1.41e7            416            59000
## 14 Zimbab… AFR         2011 1.44e7            384            55000
## 15 Zimbab… AFR         2012 1.47e7            355            52000
## 16 Zimbab… AFR         2013 1.51e7            304            46000
## 17 Zimbab… AFR         2014 1.54e7            278            43000
## 18 Zimbab… AFR         2015 1.58e7            242            38000
## 19 Zimbab… AFR         2016 1.62e7            233            38000
## 20 Zimbab… AFR         2017 1.65e7            221            37000
## # … with 12 more variables: hiv_percent <dbl>, hiv_incidence_100k <dbl>,
## #   hiv_number <dbl>, mort_nohiv_100k <dbl>, mort_nohiv_number <dbl>,
## #   mort_hiv_100k <dbl>, mort_hiv_number <dbl>, mort_100k <dbl>,
## #   mort_number <dbl>, case_fatality_ratio <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>
```

```r
# one function with the pipe
tb %>% 
  tail(20)
```

```
## # A tibble: 20 x 18
##    country who_region  year    pop incidence_100k incidence_number
##    <chr>   <chr>      <dbl>  <dbl>          <dbl>            <dbl>
##  1 Zambia  AFR         2016 1.66e7            376            62000
##  2 Zambia  AFR         2017 1.71e7            361            62000
##  3 Zimbab… AFR         2000 1.22e7            605            74000
##  4 Zimbab… AFR         2001 1.24e7            617            76000
##  5 Zimbab… AFR         2002 1.25e7            617            77000
##  6 Zimbab… AFR         2003 1.26e7            617            78000
##  7 Zimbab… AFR         2004 1.28e7            607            78000
##  8 Zimbab… AFR         2005 1.29e7            588            76000
##  9 Zimbab… AFR         2006 1.31e7            561            74000
## 10 Zimbab… AFR         2007 1.33e7            527            70000
## 11 Zimbab… AFR         2008 1.36e7            487            66000
## 12 Zimbab… AFR         2009 1.38e7            450            62000
## 13 Zimbab… AFR         2010 1.41e7            416            59000
## 14 Zimbab… AFR         2011 1.44e7            384            55000
## 15 Zimbab… AFR         2012 1.47e7            355            52000
## 16 Zimbab… AFR         2013 1.51e7            304            46000
## 17 Zimbab… AFR         2014 1.54e7            278            43000
## 18 Zimbab… AFR         2015 1.58e7            242            38000
## 19 Zimbab… AFR         2016 1.62e7            233            38000
## 20 Zimbab… AFR         2017 1.65e7            221            37000
## # … with 12 more variables: hiv_percent <dbl>, hiv_incidence_100k <dbl>,
## #   hiv_number <dbl>, mort_nohiv_100k <dbl>, mort_nohiv_number <dbl>,
## #   mort_hiv_100k <dbl>, mort_hiv_number <dbl>, mort_100k <dbl>,
## #   mort_number <dbl>, case_fatality_ratio <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>
```

```r
# two functions without the pipe
unique(tail(tb, 20)$country)
```

```
## [1] "Zambia"   "Zimbabwe"
```

```r
# two functions with the pipe
tb %>% 
  tail(20) %>%
  distinct(country)
```

```
## # A tibble: 2 x 1
##   country 
##   <chr>   
## 1 Zambia  
## 2 Zimbabwe
```

We can use this pipe operator to chain together operations rather than nesting functions together. Let's learn a few functions for data manipulation.

# The dplyr package

The [dplyr package](https://dplyr.tidyverse.org/) is an R package that makes data manipulation fast and easy.

The dplyr package gives you a handful of useful functions for managing data. On their own they don't do anything that base R can't do, but the power lies in piping them together. The first three functions we will learn are arrange, filter, and select

1. `arrange()` -- sorts the dataframe by the specified column(s) 
2. `filter()` -- subsets dataframe for **rows** that meet logical criteria
3. `select()` -- subsets dataframe for specified **columns**

They all take a data frame or tibble as their input for the first argument, and they all return a data frame or tibble as output.

## arrange()

The `arrange()` function does what it sounds like. It takes a data frame or tbl and arranges (or sorts) by column(s) of interest. The first argument is the data, and subsequent arguments are columns to sort on. Use the `desc()` function to arrange by descending.

Let's use arrange to see tb in order of incidence_100k


```r
tb %>%
  arrange(incidence_100k)
```

```
## # A tibble: 3,850 x 18
##    country who_region  year    pop incidence_100k incidence_number
##    <chr>   <chr>      <dbl>  <dbl>          <dbl>            <dbl>
##  1 Barbad… AMR         2011 280601              0                0
##  2 Barbad… AMR         2015 284217              0                0
##  3 Barbad… AMR         2017 285719              0                0
##  4 Bermuda AMR         2000  64028              0                0
##  5 Bermuda AMR         2001  64323              0                0
##  6 Bermuda AMR         2002  64610              0                0
##  7 Bermuda AMR         2013  62771              0                0
##  8 Bermuda AMR         2014  62382              0                0
##  9 Bermuda AMR         2015  62003              0                0
## 10 Bonair… AMR         2010  20940              0                0
## # … with 3,840 more rows, and 12 more variables: hiv_percent <dbl>,
## #   hiv_incidence_100k <dbl>, hiv_number <dbl>, mort_nohiv_100k <dbl>,
## #   mort_nohiv_number <dbl>, mort_hiv_100k <dbl>, mort_hiv_number <dbl>,
## #   mort_100k <dbl>, mort_number <dbl>, case_fatality_ratio <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>
```

What about by descending incidence so ones with high incidence are on top?


```r
tb %>%
  arrange(desc(incidence_100k))
```

```
## # A tibble: 3,850 x 18
##    country who_region  year    pop incidence_100k incidence_number
##    <chr>   <chr>      <dbl>  <dbl>          <dbl>            <dbl>
##  1 Lesotho AFR         2005 1.95e6           1280            25000
##  2 Lesotho AFR         2006 1.97e6           1280            25000
##  3 Lesotho AFR         2004 1.93e6           1260            24000
##  4 Lesotho AFR         2007 1.98e6           1260            25000
##  5 Lesotho AFR         2003 1.92e6           1220            23000
##  6 Lesotho AFR         2008 2.00e6           1220            24000
##  7 Lesotho AFR         2009 2.02e6           1180            24000
##  8 Lesotho AFR         2002 1.90e6           1160            22000
##  9 Lesotho AFR         2010 2.04e6           1120            23000
## 10 Lesotho AFR         2001 1.89e6           1080            20000
## # … with 3,840 more rows, and 12 more variables: hiv_percent <dbl>,
## #   hiv_incidence_100k <dbl>, hiv_number <dbl>, mort_nohiv_100k <dbl>,
## #   mort_nohiv_number <dbl>, mort_hiv_100k <dbl>, mort_hiv_number <dbl>,
## #   mort_100k <dbl>, mort_number <dbl>, case_fatality_ratio <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>
```

## filter()

If you want to subset **rows** of the dataframe where some logical condition is true, use the `filter()` function. 

1. The first argument is the data frame you want to filter, e.g. `filter(mydata, ...`.
2. The second argument is a condition you must satisfy, e.g. `filter(mydata, variable1 == "levelA")`.

- `==`: Equal to
- `!=`: Not equal to
- `>`, `>=`: Greater than, greater than or equal to
- `<`, `<=`: Less than, less than or equal to

If you want to satisfy *all* of multiple conditions, you can use the "and" operator, `&`. 

The "or" operator `|` (the pipe character, usually shift-backslash) will return a subset that meet *any* of the conditions (but not multiple).

For example, let's return rows where the who_region is Europe


```r
tb %>% 
  filter(who_region == "EUR")
```

```
## # A tibble: 967 x 18
##    country who_region  year    pop incidence_100k incidence_number
##    <chr>   <chr>      <dbl>  <dbl>          <dbl>            <dbl>
##  1 Albania EUR         2000 3.12e6             22              690
##  2 Albania EUR         2001 3.12e6             20              640
##  3 Albania EUR         2002 3.12e6             22              680
##  4 Albania EUR         2003 3.11e6             21              640
##  5 Albania EUR         2004 3.10e6             20              630
##  6 Albania EUR         2005 3.08e6             19              580
##  7 Albania EUR         2006 3.05e6             18              540
##  8 Albania EUR         2007 3.02e6             17              500
##  9 Albania EUR         2008 2.99e6             16              490
## 10 Albania EUR         2009 2.96e6             17              510
## # … with 957 more rows, and 12 more variables: hiv_percent <dbl>,
## #   hiv_incidence_100k <dbl>, hiv_number <dbl>, mort_nohiv_100k <dbl>,
## #   mort_nohiv_number <dbl>, mort_hiv_100k <dbl>, mort_hiv_number <dbl>,
## #   mort_100k <dbl>, mort_number <dbl>, case_fatality_ratio <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>
```
  
Now let's see rows where TB incidence per 100k is greater than or equal to 750


```r
tb %>% 
  filter(incidence_100k >= 750)
```

```
## # A tibble: 58 x 18
##    country who_region  year    pop incidence_100k incidence_number
##    <chr>   <chr>      <dbl>  <dbl>          <dbl>            <dbl>
##  1 Botswa… AFR         2000 1.73e6            914            16000
##  2 Botswa… AFR         2001 1.75e6            888            16000
##  3 Botswa… AFR         2002 1.78e6            855            15000
##  4 Botswa… AFR         2003 1.80e6            816            15000
##  5 Botswa… AFR         2004 1.83e6            773            14000
##  6 Centra… AFR         2000 3.75e6           1070            40000
##  7 Centra… AFR         2001 3.83e6           1000            38000
##  8 Centra… AFR         2002 3.91e6            923            36000
##  9 Centra… AFR         2003 3.98e6            842            34000
## 10 Centra… AFR         2004 4.06e6            762            31000
## # … with 48 more rows, and 12 more variables: hiv_percent <dbl>,
## #   hiv_incidence_100k <dbl>, hiv_number <dbl>, mort_nohiv_100k <dbl>,
## #   mort_nohiv_number <dbl>, mort_hiv_100k <dbl>, mort_hiv_number <dbl>,
## #   mort_100k <dbl>, mort_number <dbl>, case_fatality_ratio <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>
```

## select()

Whereas the `filter()` function allows you to return only certain _rows_ matching a condition, the `select()` function returns only certain _columns_. The first argument is the data, and subsequent arguments are the columns you want.

See only the country and year columns


```r
tb %>%
  select(country, year)
```

```
## # A tibble: 3,850 x 2
##    country      year
##    <chr>       <dbl>
##  1 Afghanistan  2000
##  2 Afghanistan  2001
##  3 Afghanistan  2002
##  4 Afghanistan  2003
##  5 Afghanistan  2004
##  6 Afghanistan  2005
##  7 Afghanistan  2006
##  8 Afghanistan  2007
##  9 Afghanistan  2008
## 10 Afghanistan  2009
## # … with 3,840 more rows
```

See all except the mort_100k column


```r
tb %>%
  select(-mort_100k)
```

```
## # A tibble: 3,850 x 17
##    country who_region  year    pop incidence_100k incidence_number
##    <chr>   <chr>      <dbl>  <dbl>          <dbl>            <dbl>
##  1 Afghan… EMR         2000 2.01e7            190            38000
##  2 Afghan… EMR         2001 2.10e7            189            40000
##  3 Afghan… EMR         2002 2.20e7            189            42000
##  4 Afghan… EMR         2003 2.31e7            189            44000
##  5 Afghan… EMR         2004 2.41e7            189            46000
##  6 Afghan… EMR         2005 2.51e7            189            47000
##  7 Afghan… EMR         2006 2.59e7            189            49000
##  8 Afghan… EMR         2007 2.66e7            189            50000
##  9 Afghan… EMR         2008 2.73e7            189            52000
## 10 Afghan… EMR         2009 2.80e7            189            53000
## # … with 3,840 more rows, and 11 more variables: hiv_percent <dbl>,
## #   hiv_incidence_100k <dbl>, hiv_number <dbl>, mort_nohiv_100k <dbl>,
## #   mort_nohiv_number <dbl>, mort_hiv_100k <dbl>, mort_hiv_number <dbl>,
## #   mort_number <dbl>, case_fatality_ratio <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>
```

Remove all columns containing "mort"


```r
tb %>%
  select(-contains("mort"))
```

```
## # A tibble: 3,850 x 12
##    country who_region  year    pop incidence_100k incidence_number
##    <chr>   <chr>      <dbl>  <dbl>          <dbl>            <dbl>
##  1 Afghan… EMR         2000 2.01e7            190            38000
##  2 Afghan… EMR         2001 2.10e7            189            40000
##  3 Afghan… EMR         2002 2.20e7            189            42000
##  4 Afghan… EMR         2003 2.31e7            189            44000
##  5 Afghan… EMR         2004 2.41e7            189            46000
##  6 Afghan… EMR         2005 2.51e7            189            47000
##  7 Afghan… EMR         2006 2.59e7            189            49000
##  8 Afghan… EMR         2007 2.66e7            189            50000
##  9 Afghan… EMR         2008 2.73e7            189            52000
## 10 Afghan… EMR         2009 2.80e7            189            53000
## # … with 3,840 more rows, and 6 more variables: hiv_percent <dbl>,
## #   hiv_incidence_100k <dbl>, hiv_number <dbl>, case_fatality_ratio <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>
```

# dplyr in action

Let's filter for where the incidence per 100k is greater than or equal to 750 then just select the country, year, and incidence columns


```r
tb %>% 
  filter(incidence_100k >= 750) %>% 
  select(country, year, incidence_100k)
```

```
## # A tibble: 58 x 3
##    country                   year incidence_100k
##    <chr>                    <dbl>          <dbl>
##  1 Botswana                  2000            914
##  2 Botswana                  2001            888
##  3 Botswana                  2002            855
##  4 Botswana                  2003            816
##  5 Botswana                  2004            773
##  6 Central African Republic  2000           1070
##  7 Central African Republic  2001           1000
##  8 Central African Republic  2002            923
##  9 Central African Republic  2003            842
## 10 Central African Republic  2004            762
## # … with 48 more rows
```

```r
# see all the output
tb %>% 
  filter(incidence_100k >= 750) %>% 
  select(country, year, incidence_100k) %>%
  print(n = Inf)
```

```
## # A tibble: 58 x 3
##    country                   year incidence_100k
##    <chr>                    <dbl>          <dbl>
##  1 Botswana                  2000            914
##  2 Botswana                  2001            888
##  3 Botswana                  2002            855
##  4 Botswana                  2003            816
##  5 Botswana                  2004            773
##  6 Central African Republic  2000           1070
##  7 Central African Republic  2001           1000
##  8 Central African Republic  2002            923
##  9 Central African Republic  2003            842
## 10 Central African Republic  2004            762
## 11 Eswatini                  2002            780
## 12 Eswatini                  2003            891
## 13 Eswatini                  2004            921
## 14 Eswatini                  2005            984
## 15 Eswatini                  2006           1010
## 16 Eswatini                  2007            976
## 17 Eswatini                  2008            937
## 18 Eswatini                  2009           1060
## 19 Eswatini                  2010           1050
## 20 Eswatini                  2011            851
## 21 Lesotho                   2000            992
## 22 Lesotho                   2001           1080
## 23 Lesotho                   2002           1160
## 24 Lesotho                   2003           1220
## 25 Lesotho                   2004           1260
## 26 Lesotho                   2005           1280
## 27 Lesotho                   2006           1280
## 28 Lesotho                   2007           1260
## 29 Lesotho                   2008           1220
## 30 Lesotho                   2009           1180
## 31 Lesotho                   2010           1120
## 32 Lesotho                   2011           1050
## 33 Lesotho                   2012            985
## 34 Lesotho                   2013            916
## 35 Lesotho                   2014            852
## 36 Lesotho                   2015            788
## 37 Namibia                   2001            845
## 38 Namibia                   2002            846
## 39 Namibia                   2003            912
## 40 Namibia                   2004            935
## 41 Namibia                   2005            918
## 42 Namibia                   2006            892
## 43 Namibia                   2007            833
## 44 Namibia                   2008            798
## 45 South Africa              2003            820
## 46 South Africa              2004            883
## 47 South Africa              2005            932
## 48 South Africa              2006            963
## 49 South Africa              2007            977
## 50 South Africa              2008            977
## 51 South Africa              2009            967
## 52 South Africa              2010            948
## 53 South Africa              2011            922
## 54 South Africa              2012            892
## 55 South Africa              2013            849
## 56 South Africa              2014            820
## 57 South Africa              2015            759
## 58 Zambia                    2000            759
```

Let's put the output above in order by descending incidence


```r
tb %>% 
  filter(incidence_100k > 750) %>% 
  select(country, year, incidence_100k) %>%
  arrange(desc(incidence_100k)) %>%
  print(n = Inf)
```

```
## # A tibble: 58 x 3
##    country                   year incidence_100k
##    <chr>                    <dbl>          <dbl>
##  1 Lesotho                   2005           1280
##  2 Lesotho                   2006           1280
##  3 Lesotho                   2004           1260
##  4 Lesotho                   2007           1260
##  5 Lesotho                   2003           1220
##  6 Lesotho                   2008           1220
##  7 Lesotho                   2009           1180
##  8 Lesotho                   2002           1160
##  9 Lesotho                   2010           1120
## 10 Lesotho                   2001           1080
## 11 Central African Republic  2000           1070
## 12 Eswatini                  2009           1060
## 13 Eswatini                  2010           1050
## 14 Lesotho                   2011           1050
## 15 Eswatini                  2006           1010
## 16 Central African Republic  2001           1000
## 17 Lesotho                   2000            992
## 18 Lesotho                   2012            985
## 19 Eswatini                  2005            984
## 20 South Africa              2007            977
## 21 South Africa              2008            977
## 22 Eswatini                  2007            976
## 23 South Africa              2009            967
## 24 South Africa              2006            963
## 25 South Africa              2010            948
## 26 Eswatini                  2008            937
## 27 Namibia                   2004            935
## 28 South Africa              2005            932
## 29 Central African Republic  2002            923
## 30 South Africa              2011            922
## 31 Eswatini                  2004            921
## 32 Namibia                   2005            918
## 33 Lesotho                   2013            916
## 34 Botswana                  2000            914
## 35 Namibia                   2003            912
## 36 Namibia                   2006            892
## 37 South Africa              2012            892
## 38 Eswatini                  2003            891
## 39 Botswana                  2001            888
## 40 South Africa              2004            883
## 41 Botswana                  2002            855
## 42 Lesotho                   2014            852
## 43 Eswatini                  2011            851
## 44 South Africa              2013            849
## 45 Namibia                   2002            846
## 46 Namibia                   2001            845
## 47 Central African Republic  2003            842
## 48 Namibia                   2007            833
## 49 South Africa              2003            820
## 50 South Africa              2014            820
## 51 Botswana                  2003            816
## 52 Namibia                   2008            798
## 53 Lesotho                   2015            788
## 54 Eswatini                  2002            780
## 55 Botswana                  2004            773
## 56 Central African Republic  2004            762
## 57 South Africa              2015            759
## 58 Zambia                    2000            759
```

Using just these few functions we have already learned, we can gain quite a bit of insight into a dataset!

# EXERCISE 1

1. In 2007, which 10 countries had the highest incidence_100k? (*Hint: filter() then arrange(), then head()*)


```r
tb %>% 
  filter(year == 2007) %>%
  arrange(desc(incidence_100k)) %>%
  head(10)
```

```
## # A tibble: 10 x 18
##    country who_region  year    pop incidence_100k incidence_number
##    <chr>   <chr>      <dbl>  <dbl>          <dbl>            <dbl>
##  1 Lesotho AFR         2007 1.98e6           1260            25000
##  2 South … AFR         2007 4.99e7            977           487000
##  3 Eswati… AFR         2007 1.14e6            976            11000
##  4 Namibia AFR         2007 2.08e6            833            17000
##  5 Botswa… AFR         2007 1.91e6            641            12000
##  6 Kenya   AFR         2007 3.81e7            618           235000
##  7 Gabon   AFR         2007 1.49e6            572             8500
##  8 Centra… AFR         2007 4.28e6            561            24000
##  9 Zambia  AFR         2007 1.27e7            554            71000
## 10 Mozamb… AFR         2007 2.22e7            531           118000
## # … with 12 more variables: hiv_percent <dbl>, hiv_incidence_100k <dbl>,
## #   hiv_number <dbl>, mort_nohiv_100k <dbl>, mort_nohiv_number <dbl>,
## #   mort_hiv_100k <dbl>, mort_hiv_number <dbl>, mort_100k <dbl>,
## #   mort_number <dbl>, case_fatality_ratio <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>
```

2. Begin with your answer to #1 and `select()` only the country and the incidence_100k columns.


```r
tb %>% 
  filter(year == 2007) %>%
  arrange(desc(incidence_100k)) %>%
  head(10) %>%
  select(country, incidence_100k)
```

```
## # A tibble: 10 x 2
##    country                  incidence_100k
##    <chr>                             <dbl>
##  1 Lesotho                            1260
##  2 South Africa                        977
##  3 Eswatini                            976
##  4 Namibia                             833
##  5 Botswana                            641
##  6 Kenya                               618
##  7 Gabon                               572
##  8 Central African Republic            561
##  9 Zambia                              554
## 10 Mozambique                          531
```

3. Within the South East Asia who_region, which rows have incidence_100K > 500?


```r
tb %>% 
  filter(who_region == "SEA" & incidence_100k > 300)
```

```
## # A tibble: 70 x 18
##    country who_region  year    pop incidence_100k incidence_number
##    <chr>   <chr>      <dbl>  <dbl>          <dbl>            <dbl>
##  1 Democr… SEA         2000 2.29e7            513           118000
##  2 Democr… SEA         2001 2.31e7            513           119000
##  3 Democr… SEA         2002 2.33e7            513           120000
##  4 Democr… SEA         2003 2.35e7            513           121000
##  5 Democr… SEA         2004 2.37e7            513           122000
##  6 Democr… SEA         2005 2.39e7            513           123000
##  7 Democr… SEA         2006 2.41e7            513           123000
##  8 Democr… SEA         2007 2.42e7            513           124000
##  9 Democr… SEA         2008 2.43e7            513           125000
## 10 Democr… SEA         2009 2.45e7            513           125000
## # … with 60 more rows, and 12 more variables: hiv_percent <dbl>,
## #   hiv_incidence_100k <dbl>, hiv_number <dbl>, mort_nohiv_100k <dbl>,
## #   mort_nohiv_number <dbl>, mort_hiv_100k <dbl>, mort_hiv_number <dbl>,
## #   mort_100k <dbl>, mort_number <dbl>, case_fatality_ratio <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>
```

4. Start with your output in #3 and find out which distinct countries are meet these criteria (SEA region and incidence > 300)


```r
tb %>% 
  filter(who_region == "SEA" & incidence_100k > 300) %>%
  distinct(country)
```

```
## # A tibble: 4 x 1
##   country                              
##   <chr>                                
## 1 Democratic People's Republic of Korea
## 2 Indonesia                            
## 3 Myanmar                              
## 4 Timor-Leste
```

# The tidyr package

## separate()


```r
glimpse(gm1)
```

```
## Observations: 1,704
## Variables: 4
## $ country   <chr> "Afghanistan", "Afghanistan", "Afghanistan", "Afghanis…
## $ continent <chr> "Asia", "Asia", "Asia", "Asia", "Asia", "Asia", "Asia"…
## $ year      <dbl> 1952, 1957, 1962, 1967, 1972, 1977, 1982, 1987, 1992, …
## $ le_gdp    <chr> "28.801::779.4453145", "30.332::820.8530296", "31.997:…
```

In looking at the output of the gm1 dataset, we have some work to do to tidy the data for analysis. 

The main problem I see is that there are multiple pieces of data encoded into the column called `le_gdp`. Luckily, the 2 pieces of data are each separated neatly by `::`. Let's use the separate function from the tidyr package to create 2 new columns from the original `le_gdp` column.


```r
# take a look at the separate function
?separate

gm1 %>% 
  separate(le_gdp, into = c("lifeExp", "gdpPerCap"), sep = "::")
```

```
## # A tibble: 1,704 x 5
##    country     continent  year lifeExp gdpPerCap  
##    <chr>       <chr>     <dbl> <chr>   <chr>      
##  1 Afghanistan Asia       1952 28.801  779.4453145
##  2 Afghanistan Asia       1957 30.332  820.8530296
##  3 Afghanistan Asia       1962 31.997  853.10071  
##  4 Afghanistan Asia       1967 34.02   836.1971382
##  5 Afghanistan Asia       1972 36.088  739.9811058
##  6 Afghanistan Asia       1977 38.438  786.11336  
##  7 Afghanistan Asia       1982 39.854  978.0114388
##  8 Afghanistan Asia       1987 40.822  852.3959448
##  9 Afghanistan Asia       1992 41.674  649.3413952
## 10 Afghanistan Asia       1997 41.763  635.341351 
## # … with 1,694 more rows
```

If the separator was not as neat as this, you can input any [regular expression](https://en.wikipedia.org/wiki/Regular_expression) into the separator argument. We will do a bit more today with regular expressions, but also see Further Resources at the bottom of the script for more about regular expressions.

Notice that our original variable `le_gdp` no longer appears. This change is good, but so far it is only in the console. The originial dataframe is still unchanged. 


```r
gm1
```

```
## # A tibble: 1,704 x 4
##    country     continent  year le_gdp             
##    <chr>       <chr>     <dbl> <chr>              
##  1 Afghanistan Asia       1952 28.801::779.4453145
##  2 Afghanistan Asia       1957 30.332::820.8530296
##  3 Afghanistan Asia       1962 31.997::853.10071  
##  4 Afghanistan Asia       1967 34.02::836.1971382 
##  5 Afghanistan Asia       1972 36.088::739.9811058
##  6 Afghanistan Asia       1977 38.438::786.11336  
##  7 Afghanistan Asia       1982 39.854::978.0114388
##  8 Afghanistan Asia       1987 40.822::852.3959448
##  9 Afghanistan Asia       1992 41.674::649.3413952
## 10 Afghanistan Asia       1997 41.763::635.341351 
## # … with 1,694 more rows
```

Let's keep the change created by the `separate()` function by saving our pipeline back into gm.


```r
gm1 <- gm1 %>% 
  separate(le_gdp, into = c("lifeExp", "gdpPerCap"), sep = "::")
```

This is often the workflow we would recommend. Try a change in the console. Make sure the change is what you intended. Then save the change, either in to the original dataset or into a new object.

## gather()

The next problem we will tackle is reshaping a dataframe. 

Notice in our gm2 data dataframe that we have several variables that seem to be encoding the same information across different years. 


```r
glimpse(gm2)
```

```
## Observations: 142
## Variables: 14
## $ country   <chr> "Afghanistan", "Albania", "Algeria", "Angola", "Argent…
## $ continent <chr> "Asia", "Europe", "Africa", "Africa", "Americas", "Oce…
## $ year.1952 <dbl> 8425333, 1282697, 9279525, 4232095, 17876956, 8691212,…
## $ year.1957 <dbl> 9240934, 1476505, 10270856, 4561361, 19610538, 9712569…
## $ year.1962 <dbl> 10267083, 1728137, 11000948, 4826015, 21283783, 107949…
## $ year.1967 <dbl> 11537966, 1984060, 12760499, 5247469, 22934225, 118722…
## $ year.1972 <dbl> 13079460, 2263554, 14760787, 5894858, 24779799, 131770…
## $ year.1977 <dbl> 14880372, 2509048, 17152804, 6162675, 26983828, 140741…
## $ year.1982 <dbl> 12881816, 2780097, 20033753, 7016384, 29341374, 151842…
## $ year.1987 <dbl> 13867957, 3075321, 23254956, 7874230, 31620918, 162572…
## $ year.1992 <dbl> 16317921, 3326498, 26298373, 8735988, 33958947, 174819…
## $ year.1997 <dbl> 22227415, 3428038, 29072015, 9875024, 36203463, 185652…
## $ year.2002 <dbl> 25268405, 3508512, 31287142, 10866106, 38331121, 19546…
## $ year.2007 <dbl> 31889923, 3600523, 33333216, 12420476, 40301927, 20434…
```

```r
head(gm2)
```

```
## # A tibble: 6 x 14
##   country continent year.1952 year.1957 year.1962 year.1967 year.1972
##   <chr>   <chr>         <dbl>     <dbl>     <dbl>     <dbl>     <dbl>
## 1 Afghan… Asia        8425333   9240934  10267083  11537966  13079460
## 2 Albania Europe      1282697   1476505   1728137   1984060   2263554
## 3 Algeria Africa      9279525  10270856  11000948  12760499  14760787
## 4 Angola  Africa      4232095   4561361   4826015   5247469   5894858
## 5 Argent… Americas   17876956  19610538  21283783  22934225  24779799
## 6 Austra… Oceania     8691212   9712569  10794968  11872264  13177000
## # … with 7 more variables: year.1977 <dbl>, year.1982 <dbl>,
## #   year.1987 <dbl>, year.1992 <dbl>, year.1997 <dbl>, year.2002 <dbl>,
## #   year.2007 <dbl>
```

Columns year.1952 through year.2007 tell us the population for that country in each of the years. We will use `gather()` to change the dataframe from wide to long, making each of the years a different row. This will also help a lot when merging our `gm2` dataset together with the `tb` dataset which has a row for each country-year.

After the dataframe, `gather()` needs 3 other arguments. The first two are very important, but take some getting used to. 

- `key` is the new column you want to create that has the old dataframe column headers
- `value` corresponds to the row entries from old dataframe that you want in a new column
- The third argument is the vector of columns that we want to gather


```r
gm2 <- gm2 %>% 
  gather(key = year, value = pop, year.1952:year.2007)
```

That change is good -- let's save it as gm2

Let's clean up the resulting year column so that they are just the years.

## mutate()

`mutate()` will create a new variable or change an existing variable in place. In this case, we would like to modify the existing year column so that it just has the year rather than the "year." ahead of it.

Just like the other dplyr functions, `mutate()` takes a dataframe as the first argument and then the name of the new variable followed by a function to create the new variable.

To clean up strings, the stringr package has several excellent functions. Today, we'll use `str_replace()` to remove the "year." before the numeric year.

`str_replace()` takes the regular expression to find followed by the replacement. I think of it like Find and Replace in Microsoft Word


```r
gm2 %>%
  mutate(year = str_replace(year, "year.", ""))
```

```
## # A tibble: 1,704 x 4
##    country     continent year       pop
##    <chr>       <chr>     <chr>    <dbl>
##  1 Afghanistan Asia      1952   8425333
##  2 Albania     Europe    1952   1282697
##  3 Algeria     Africa    1952   9279525
##  4 Angola      Africa    1952   4232095
##  5 Argentina   Americas  1952  17876956
##  6 Australia   Oceania   1952   8691212
##  7 Austria     Europe    1952   6927772
##  8 Bahrain     Asia      1952    120447
##  9 Bangladesh  Asia      1952  46886859
## 10 Belgium     Europe    1952   8730405
## # … with 1,694 more rows
```

There is still a slight problem with the year column. Can you spot it?

Great, let's fix it using `mutate()`


```r
gm2 %>%
  mutate(year = str_replace(year, "year.", "")) %>% 
  mutate(year = as.numeric(year))
```

```
## # A tibble: 1,704 x 4
##    country     continent  year      pop
##    <chr>       <chr>     <dbl>    <dbl>
##  1 Afghanistan Asia       1952  8425333
##  2 Albania     Europe     1952  1282697
##  3 Algeria     Africa     1952  9279525
##  4 Angola      Africa     1952  4232095
##  5 Argentina   Americas   1952 17876956
##  6 Australia   Oceania    1952  8691212
##  7 Austria     Europe     1952  6927772
##  8 Bahrain     Asia       1952   120447
##  9 Bangladesh  Asia       1952 46886859
## 10 Belgium     Europe     1952  8730405
## # … with 1,694 more rows
```

```r
# perfect! Let's save that change
gm2 <- gm2 %>%
  mutate(year = str_replace(year, "year.", "")) %>% 
  mutate(year = as.numeric(year))
```

If we wanted to reshape a dataframe from long to wide, we would use `spread()`. Typically, this is more rare than `gather()`


```r
spread(gm2, year, pop)
```

```
## # A tibble: 142 x 14
##    country continent `1952` `1957` `1962` `1967` `1972` `1977` `1982`
##    <chr>   <chr>      <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
##  1 Afghan… Asia      8.43e6 9.24e6 1.03e7 1.15e7 1.31e7 1.49e7 1.29e7
##  2 Albania Europe    1.28e6 1.48e6 1.73e6 1.98e6 2.26e6 2.51e6 2.78e6
##  3 Algeria Africa    9.28e6 1.03e7 1.10e7 1.28e7 1.48e7 1.72e7 2.00e7
##  4 Angola  Africa    4.23e6 4.56e6 4.83e6 5.25e6 5.89e6 6.16e6 7.02e6
##  5 Argent… Americas  1.79e7 1.96e7 2.13e7 2.29e7 2.48e7 2.70e7 2.93e7
##  6 Austra… Oceania   8.69e6 9.71e6 1.08e7 1.19e7 1.32e7 1.41e7 1.52e7
##  7 Austria Europe    6.93e6 6.97e6 7.13e6 7.38e6 7.54e6 7.57e6 7.57e6
##  8 Bahrain Asia      1.20e5 1.39e5 1.72e5 2.02e5 2.31e5 2.97e5 3.78e5
##  9 Bangla… Asia      4.69e7 5.14e7 5.68e7 6.28e7 7.08e7 8.04e7 9.31e7
## 10 Belgium Europe    8.73e6 8.99e6 9.22e6 9.56e6 9.71e6 9.82e6 9.86e6
## # … with 132 more rows, and 5 more variables: `1987` <dbl>, `1992` <dbl>,
## #   `1997` <dbl>, `2002` <dbl>, `2007` <dbl>
```

# Clean up tb using select()

Before we join some datasets together, let's get rid of columns on the tb dataframe we will no longer use.

Let's remove anything starting with "mort", ending with "ratio", or containing "hiv"


```r
glimpse(tb)
```

```
## Observations: 3,850
## Variables: 18
## $ country                <chr> "Afghanistan", "Afghanistan", "Afghanista…
## $ who_region             <chr> "EMR", "EMR", "EMR", "EMR", "EMR", "EMR",…
## $ year                   <dbl> 2000, 2001, 2002, 2003, 2004, 2005, 2006,…
## $ pop                    <dbl> 20093756, 20966463, 21979923, 23064851, 2…
## $ incidence_100k         <dbl> 190, 189, 189, 189, 189, 189, 189, 189, 1…
## $ incidence_number       <dbl> 38000, 40000, 42000, 44000, 46000, 47000,…
## $ hiv_percent            <dbl> 0.36, 0.30, 0.26, 0.23, 0.22, 0.22, 0.22,…
## $ hiv_incidence_100k     <dbl> 0.68, 0.57, 0.49, 0.44, 0.41, 0.42, 0.42,…
## $ hiv_number             <dbl> 140, 120, 110, 100, 100, 100, 110, 120, 1…
## $ mort_nohiv_100k        <dbl> 67.00, 62.00, 56.00, 57.00, 51.00, 46.00,…
## $ mort_nohiv_number      <dbl> 14000, 13000, 12000, 13000, 12000, 12000,…
## $ mort_hiv_100k          <dbl> 0.15, 0.17, 0.27, 0.25, 0.21, 0.19, 0.18,…
## $ mort_hiv_number        <dbl> 31, 35, 60, 57, 50, 48, 46, 45, 48, 55, 5…
## $ mort_100k              <dbl> 67.00, 62.00, 56.00, 57.00, 51.00, 46.00,…
## $ mort_number            <dbl> 14000, 13000, 12000, 13000, 12000, 12000,…
## $ case_fatality_ratio    <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
## $ new_incidence_100k     <dbl> 35, 48, 63, 60, 76, 87, 98, 108, 104, 93,…
## $ case_detection_percent <dbl> 19, 26, 33, 32, 40, 46, 52, 57, 55, 49, 5…
```

The `starts_with()`, `ends_with()`, and `contains()` helper functions are a nice way to take care of similarly named variables


```r
tb %>% 
  select(-starts_with("mort"), -ends_with("ratio"), -contains("hiv"))
```

```
## # A tibble: 3,850 x 8
##    country who_region  year    pop incidence_100k incidence_number
##    <chr>   <chr>      <dbl>  <dbl>          <dbl>            <dbl>
##  1 Afghan… EMR         2000 2.01e7            190            38000
##  2 Afghan… EMR         2001 2.10e7            189            40000
##  3 Afghan… EMR         2002 2.20e7            189            42000
##  4 Afghan… EMR         2003 2.31e7            189            44000
##  5 Afghan… EMR         2004 2.41e7            189            46000
##  6 Afghan… EMR         2005 2.51e7            189            47000
##  7 Afghan… EMR         2006 2.59e7            189            49000
##  8 Afghan… EMR         2007 2.66e7            189            50000
##  9 Afghan… EMR         2008 2.73e7            189            52000
## 10 Afghan… EMR         2009 2.80e7            189            53000
## # … with 3,840 more rows, and 2 more variables: new_incidence_100k <dbl>,
## #   case_detection_percent <dbl>
```

```r
# save that change
tb <- tb %>% 
  select(-starts_with("mort"), -ends_with("ratio"), -contains("hiv"))
```

tb should now have 8 variables

# EXERCISE 2

1. On your own, remove continent from the gm2 dataset and save the change


```r
gm2 <- gm2 %>%
  select(-continent)
```



gm2 should now have 3 variables

2. Remove pop from the tb dataset and save that change


```r
tb <- tb %>% select(-pop)
```

tb should now have 7 variables

# Break

To make sure we all have the same datasets, in case something went wrong up above, let's upload the above datasets to be safe.


```r
gm1 <- read_csv("gm1_clean.csv")
```

```
## Parsed with column specification:
## cols(
##   country = col_character(),
##   continent = col_character(),
##   year = col_double(),
##   lifeExp = col_double(),
##   gdpPerCap = col_double()
## )
```

```r
gm2  <- read_csv("gm2_clean.csv")
```

```
## Parsed with column specification:
## cols(
##   country = col_character(),
##   year = col_double(),
##   pop = col_double()
## )
```

```r
tb  <- read_csv("tb_clean.csv")
```

```
## Parsed with column specification:
## cols(
##   country = col_character(),
##   who_region = col_character(),
##   year = col_double(),
##   incidence_100k = col_double(),
##   incidence_number = col_double(),
##   new_incidence_100k = col_double(),
##   case_detection_percent = col_double()
## )
```

# Joins

There are several types of joins (merges) that we can use. The one you should use depends upon your specific needs.

First let's look closely at which columns link the two datasets.


```r
head(tb)
```

```
## # A tibble: 6 x 7
##   country who_region  year incidence_100k incidence_number new_incidence_1…
##   <chr>   <chr>      <dbl>          <dbl>            <dbl>            <dbl>
## 1 Afghan… EMR         2000            190            38000               35
## 2 Afghan… EMR         2001            189            40000               48
## 3 Afghan… EMR         2002            189            42000               63
## 4 Afghan… EMR         2003            189            44000               60
## 5 Afghan… EMR         2004            189            46000               76
## 6 Afghan… EMR         2005            189            47000               87
## # … with 1 more variable: case_detection_percent <dbl>
```

```r
head(gm1)
```

```
## # A tibble: 6 x 5
##   country     continent  year lifeExp gdpPerCap
##   <chr>       <chr>     <dbl>   <dbl>     <dbl>
## 1 Afghanistan Asia       1952    28.8      779.
## 2 Afghanistan Asia       1957    30.3      821.
## 3 Afghanistan Asia       1962    32.0      853.
## 4 Afghanistan Asia       1967    34.0      836.
## 5 Afghanistan Asia       1972    36.1      740.
## 6 Afghanistan Asia       1977    38.4      786.
```

Each row in tb AND gm is a country-year. These two variables uniquely identify each of the observations in both datasets. Thus, these are the variables we should join on.

Do all the country names match?

Let's see with an example. Let's find USA in each. The function we will use is `str_detect()` from the stringr package. We'll use `str_detect()` inside `filter()` to only return rows with that regular expression pattern.


```r
# in tb
tb %>%
  filter(str_detect(country, "United")) %>%
  distinct(country) 
```

```
## # A tibble: 4 x 1
##   country                                             
##   <chr>                                               
## 1 United Arab Emirates                                
## 2 United Kingdom of Great Britain and Northern Ireland
## 3 United Republic of Tanzania                         
## 4 United States of America
```

```r
# called United States of America

# in gm
gm1 %>%
  filter(str_detect(country, "United"))  %>%
  distinct(country)
```

```
## # A tibble: 2 x 1
##   country       
##   <chr>         
## 1 United Kingdom
## 2 United States
```

```r
# called United States
```

## anti_join()

How many more countries do not match? To find out, the function we'll use is anti_join which returns rows in the left dataframe that are not present in the right


```r
#countries in tb not in gm
anti_join(tb, gm1, by = "country") %>% 
  distinct(country) %>%
  print(n = Inf)
```

```
## # A tibble: 97 x 1
##    country                                             
##    <chr>                                               
##  1 American Samoa                                      
##  2 Andorra                                             
##  3 Anguilla                                            
##  4 Antigua and Barbuda                                 
##  5 Armenia                                             
##  6 Aruba                                               
##  7 Azerbaijan                                          
##  8 Bahamas                                             
##  9 Barbados                                            
## 10 Belarus                                             
## 11 Belize                                              
## 12 Bermuda                                             
## 13 Bhutan                                              
## 14 Bolivia (Plurinational State of)                    
## 15 Bonaire, Saint Eustatius and Saba                   
## 16 British Virgin Islands                              
## 17 Brunei Darussalam                                   
## 18 Cabo Verde                                          
## 19 Cayman Islands                                      
## 20 China, Hong Kong SAR                                
## 21 China, Macao SAR                                    
## 22 Congo                                               
## 23 Cook Islands                                        
## 24 Côte d'Ivoire                                       
## 25 Curaçao                                             
## 26 Cyprus                                              
## 27 Czechia                                             
## 28 Democratic People's Republic of Korea               
## 29 Democratic Republic of the Congo                    
## 30 Dominica                                            
## 31 Estonia                                             
## 32 Eswatini                                            
## 33 Fiji                                                
## 34 French Polynesia                                    
## 35 Georgia                                             
## 36 Greenland                                           
## 37 Grenada                                             
## 38 Guam                                                
## 39 Guyana                                              
## 40 Iran (Islamic Republic of)                          
## 41 Kazakhstan                                          
## 42 Kiribati                                            
## 43 Kyrgyzstan                                          
## 44 Lao People's Democratic Republic                    
## 45 Latvia                                              
## 46 Lithuania                                           
## 47 Luxembourg                                          
## 48 Maldives                                            
## 49 Malta                                               
## 50 Marshall Islands                                    
## 51 Micronesia (Federated States of)                    
## 52 Monaco                                              
## 53 Montserrat                                          
## 54 Nauru                                               
## 55 Netherlands Antilles                                
## 56 New Caledonia                                       
## 57 Niue                                                
## 58 North Macedonia                                     
## 59 Northern Mariana Islands                            
## 60 Palau                                               
## 61 Papua New Guinea                                    
## 62 Qatar                                               
## 63 Republic of Korea                                   
## 64 Republic of Moldova                                 
## 65 Russian Federation                                  
## 66 Saint Kitts and Nevis                               
## 67 Saint Lucia                                         
## 68 Saint Vincent and the Grenadines                    
## 69 Samoa                                               
## 70 San Marino                                          
## 71 Serbia & Montenegro                                 
## 72 Seychelles                                          
## 73 Sint Maarten (Dutch part)                           
## 74 Slovakia                                            
## 75 Solomon Islands                                     
## 76 South Sudan                                         
## 77 Suriname                                            
## 78 Syrian Arab Republic                                
## 79 Tajikistan                                          
## 80 Timor-Leste                                         
## 81 Tokelau                                             
## 82 Tonga                                               
## 83 Turkmenistan                                        
## 84 Turks and Caicos Islands                            
## 85 Tuvalu                                              
## 86 Ukraine                                             
## 87 United Arab Emirates                                
## 88 United Kingdom of Great Britain and Northern Ireland
## 89 United Republic of Tanzania                         
## 90 United States of America                            
## 91 Uzbekistan                                          
## 92 Vanuatu                                             
## 93 Venezuela (Bolivarian Republic of)                  
## 94 Viet Nam                                            
## 95 Wallis and Futuna Islands                           
## 96 West Bank and Gaza Strip                            
## 97 Yemen
```

```r
# countries in gm not in tb
anti_join(gm1, tb, by = "country") %>% 
  distinct(country) %>%
  print(n = Inf)
```

```
## # A tibble: 21 x 1
##    country           
##    <chr>             
##  1 Bolivia           
##  2 Congo, Dem. Rep.  
##  3 Congo, Rep.       
##  4 Cote d'Ivoire     
##  5 Czech Republic    
##  6 Hong Kong, China  
##  7 Iran              
##  8 Korea, Dem. Rep.  
##  9 Korea, Rep.       
## 10 Reunion           
## 11 Slovak Republic   
## 12 Swaziland         
## 13 Syria             
## 14 Taiwan            
## 15 Tanzania          
## 16 United Kingdom    
## 17 United States     
## 18 Venezuela         
## 19 Vietnam           
## 20 West Bank and Gaza
## 21 Yemen, Rep.
```

Let's change country names in tb to match those in the gm datasets. Luckily, I have made a table for you with the original country names from tb followed by the countries' names in the gm dataset


```r
name_lookup <- read_csv("new_country_names.csv")
```

```
## Parsed with column specification:
## cols(
##   old_country = col_character(),
##   new_country = col_character()
## )
```

```r
name_lookup %>% print(n = 30)
```

```
## # A tibble: 218 x 2
##    old_country                       new_country                      
##    <chr>                             <chr>                            
##  1 Afghanistan                       Afghanistan                      
##  2 Albania                           Albania                          
##  3 Algeria                           Algeria                          
##  4 American Samoa                    American Samoa                   
##  5 Andorra                           Andorra                          
##  6 Angola                            Angola                           
##  7 Anguilla                          Anguilla                         
##  8 Antigua and Barbuda               Antigua and Barbuda              
##  9 Argentina                         Argentina                        
## 10 Armenia                           Armenia                          
## 11 Aruba                             Aruba                            
## 12 Australia                         Australia                        
## 13 Austria                           Austria                          
## 14 Azerbaijan                        Azerbaijan                       
## 15 Bahamas                           Bahamas                          
## 16 Bahrain                           Bahrain                          
## 17 Bangladesh                        Bangladesh                       
## 18 Barbados                          Barbados                         
## 19 Belarus                           Belarus                          
## 20 Belgium                           Belgium                          
## 21 Belize                            Belize                           
## 22 Benin                             Benin                            
## 23 Bermuda                           Bermuda                          
## 24 Bhutan                            Bhutan                           
## 25 Bolivia (Plurinational State of)  Bolivia                          
## 26 Bonaire, Saint Eustatius and Saba Bonaire, Saint Eustatius and Saba
## 27 Bosnia and Herzegovina            Bosnia and Herzegovina           
## 28 Botswana                          Botswana                         
## 29 Brazil                            Brazil                           
## 30 British Virgin Islands            British Virgin Islands           
## # … with 188 more rows
```

## left_join()

Now we want to merge tb with name_lookup to attach our preferred country names to the tb dataset before we merge on gm1 and gm2.

- `left_join()`: return all rows from the left, and all columns from left and right. Rows in left with no match in right will have NA values in the new columns. If there are multiple matches between left and right, all combinations of the matches are returned.

Let's start by looking at the help menu for `left_join()`


```r
?left_join
```

Pay special attention to the argument "by = " to learn how to join datasets where the names of the columns to join are different.


```r
left_join(tb, name_lookup, by = c("country" = "old_country")) %>% View()
```

Save that change.


```r
tb <- left_join(tb, name_lookup, by = c("country" = "old_country"))
```

Now take away the old country column and replace it with the new_country (but call new_country country)


```r
tb %>%
  select(-country) %>%
  select(country = new_country, everything()) %>% View()

# save the change
tb <- tb %>%
  select(-country) %>%
  select(country = new_country, everything())
```

Great! Now let's worry about the years. Years that match between tb and gm are 2002 and 2007. 

# EXERCISE 3

Let's just keep those rows belonging to 2002 or 2007. Write this code on your own and call the resulting dataset tb2000


```r
tb2000 <- filter(tb, year == 2002 | year == 2007)
```

## Other types of joins

- `right_join()`: return all rows from the right, and all columns from left and right. Rows in right with no match in left will have NA values in the new columns. If there are multiple matches between left and right, all combinations of the matches are returned.

- `inner_join()`: return all rows from left where there are matching values in right, and all columns from left and right. If there are multiple matches between left and right, all combination of the matches are returned.

- `full_join()`: return all rows and all columns from both left and right. Where there are not matching values, returns NA for the one missing.

### Left join

Let's try a left join first, starting with tb2000 on the left and adding gm1 to it on the right

Think about the join that we are about to do. How many columns do we expect? How many rows?


```r
left_join(tb2000, gm1, by = c("country", "year")) %>% View()
```

What if we had joined tb to gm1


```r
left_join(tb, gm1, by = c("country", "year")) %>% View()
```

### Right join

A right join will keep all the rows in the right dataset. If we join tb to gm1, what are the dimensions we expect now?


```r
right_join(tb, gm1, by = c("country", "year"))
```

```
## # A tibble: 1,704 x 10
##    country who_region  year incidence_100k incidence_number
##    <chr>   <chr>      <dbl>          <dbl>            <dbl>
##  1 Afghan… <NA>        1952             NA               NA
##  2 Afghan… <NA>        1957             NA               NA
##  3 Afghan… <NA>        1962             NA               NA
##  4 Afghan… <NA>        1967             NA               NA
##  5 Afghan… <NA>        1972             NA               NA
##  6 Afghan… <NA>        1977             NA               NA
##  7 Afghan… <NA>        1982             NA               NA
##  8 Afghan… <NA>        1987             NA               NA
##  9 Afghan… <NA>        1992             NA               NA
## 10 Afghan… <NA>        1997             NA               NA
## # … with 1,694 more rows, and 5 more variables: new_incidence_100k <dbl>,
## #   case_detection_percent <dbl>, continent <chr>, lifeExp <dbl>,
## #   gdpPerCap <dbl>
```

### Inner join

An inner join keeps only rows that were in both datasets. How many columns do we expect? How many rows?


```r
inner_join(tb, gm1, by = c("country", "year"))
```

```
## # A tibble: 278 x 10
##    country who_region  year incidence_100k incidence_number
##    <chr>   <chr>      <dbl>          <dbl>            <dbl>
##  1 Afghan… EMR         2002            189            42000
##  2 Afghan… EMR         2007            189            50000
##  3 Albania EUR         2002             22              680
##  4 Albania EUR         2007             17              500
##  5 Algeria AFR         2002             74            24000
##  6 Algeria AFR         2007             78            27000
##  7 Angola  AFR         2002            320            56000
##  8 Angola  AFR         2007            379            80000
##  9 Argent… AMR         2002             35            13000
## 10 Argent… AMR         2007             29            12000
## # … with 268 more rows, and 5 more variables: new_incidence_100k <dbl>,
## #   case_detection_percent <dbl>, continent <chr>, lifeExp <dbl>,
## #   gdpPerCap <dbl>
```

### Full join

Full joins keep all rows from both the left and the right datasets. How many columns do we expect? How many rows?


```r
3850 + 1704
```

```
## [1] 5554
```

```r
full_join(tb, gm1, by = c("country", "year"))
```

```
## # A tibble: 5,276 x 10
##    country who_region  year incidence_100k incidence_number
##    <chr>   <chr>      <dbl>          <dbl>            <dbl>
##  1 Afghan… EMR         2000            190            38000
##  2 Afghan… EMR         2001            189            40000
##  3 Afghan… EMR         2002            189            42000
##  4 Afghan… EMR         2003            189            44000
##  5 Afghan… EMR         2004            189            46000
##  6 Afghan… EMR         2005            189            47000
##  7 Afghan… EMR         2006            189            49000
##  8 Afghan… EMR         2007            189            50000
##  9 Afghan… EMR         2008            189            52000
## 10 Afghan… EMR         2009            189            53000
## # … with 5,266 more rows, and 5 more variables: new_incidence_100k <dbl>,
## #   case_detection_percent <dbl>, continent <chr>, lifeExp <dbl>,
## #   gdpPerCap <dbl>
```

Now that we understand how joins work, let's create a joined dataset that keeps just rows where we have data from tb and gm. What join should we use?


```r
tbgm <- inner_join(tb, gm1, by = c("country", "year"))

View(tbgm)
```

Now to that dataframe, let's left_join the gm2 dataset. gm2 also has country-year as its observational unit. Great!


```r
joined <- left_join(tbgm, gm2, by = c("country", "year"))
write_csv(joined, "joined_tbgm.csv")
View(joined)
```

# Change column order with select()

`select()` can also be used to change the order of columns in our dataframe


```r
joined %>% 
  select(country, year, who_region, continent, pop, everything())
```

```
## # A tibble: 278 x 11
##    country  year who_region continent    pop incidence_100k
##    <chr>   <dbl> <chr>      <chr>      <dbl>          <dbl>
##  1 Afghan…  2002 EMR        Asia      2.53e7            189
##  2 Afghan…  2007 EMR        Asia      3.19e7            189
##  3 Albania  2002 EUR        Europe    3.51e6             22
##  4 Albania  2007 EUR        Europe    3.60e6             17
##  5 Algeria  2002 AFR        Africa    3.13e7             74
##  6 Algeria  2007 AFR        Africa    3.33e7             78
##  7 Angola   2002 AFR        Africa    1.09e7            320
##  8 Angola   2007 AFR        Africa    1.24e7            379
##  9 Argent…  2002 AMR        Americas  3.83e7             35
## 10 Argent…  2007 AMR        Americas  4.03e7             29
## # … with 268 more rows, and 5 more variables: incidence_number <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>, lifeExp <dbl>,
## #   gdpPerCap <dbl>
```

```r
# save the change
joined <- joined %>% 
  select(country, year, who_region, continent, pop, everything())
```

In the code above, note the use of the helper function `everything()` to select all the columns I have not called by name.

# Derive Insights using dplyr

Now that we have variables from gm and tb in the same place, we are ready to answer our question about how gdp impacts tb incidence

In country-years with highest gdp, what is the median incidence?
In country-years with lowest gdp, what is the median incidence?

First let's see what the median incidence is for all the country-years together. To do that we'll use a new dplyr function `summarize()`

### summarize()

The `summarize()` function summarizes multiple values to a single value. `summarize()` differs from base r functions like `mean()` or `median()` in that it will return a dataframe rather than a single value.

Notice that summarize takes a data frame and returns a data frame. In this case it's a 1x1 data frame with a single row and a single column.


```r
joined %>% 
  summarize(median(incidence_100k))
```

```
## # A tibble: 1 x 1
##   `median(incidence_100k)`
##                      <dbl>
## 1                     62.5
```

The name of the column, by default is whatever the expression was used to summarize the data. This usually isn't pretty, and if we wanted to work with this resulting data frame later on, we'd want to name that returned value something easier to deal with.


```r
joined %>% 
  summarize(medInc = median(incidence_100k))
```

```
## # A tibble: 1 x 1
##   medInc
##    <dbl>
## 1   62.5
```

# EXERCISE 4

1. Use dplyr functions to find the median incidence_100k for the 20 countries with the highest gdpPerCap


```r
# 20 highest gdp
joined %>%
  arrange(desc(gdpPerCap)) %>%
  head(20) %>%
  summarize(medInc = median(incidence_100k))
```

```
## # A tibble: 1 x 1
##   medInc
##    <dbl>
## 1    7.2
```

2. Follow the same process for the 20 countries with the lowest gdp


```r
# 20 lowest gdp
joined %>%
  arrange(gdpPerCap) %>%
  head(20) %>%
  summarize(medInc = median(incidence_100k))
```

```
## # A tibble: 1 x 1
##   medInc
##    <dbl>
## 1    319
```

3. Does gdp seem to influence tb incidence_100k? Share your conclusions with a neighbor.

------------------------------------

Before we end, let's learn one more dplyr trick.

### group_by()

By itself `group_by()` does not do much. All this does is takes an existing data frame and converts it into a grouped data frame.


```r
joined %>% 
  group_by(who_region)
```

```
## # A tibble: 278 x 11
## # Groups:   who_region [6]
##    country  year who_region continent    pop incidence_100k
##    <chr>   <dbl> <chr>      <chr>      <dbl>          <dbl>
##  1 Afghan…  2002 EMR        Asia      2.53e7            189
##  2 Afghan…  2007 EMR        Asia      3.19e7            189
##  3 Albania  2002 EUR        Europe    3.51e6             22
##  4 Albania  2007 EUR        Europe    3.60e6             17
##  5 Algeria  2002 AFR        Africa    3.13e7             74
##  6 Algeria  2007 AFR        Africa    3.33e7             78
##  7 Angola   2002 AFR        Africa    1.09e7            320
##  8 Angola   2007 AFR        Africa    1.24e7            379
##  9 Argent…  2002 AMR        Americas  3.83e7             35
## 10 Argent…  2007 AMR        Americas  4.03e7             29
## # … with 268 more rows, and 5 more variables: incidence_number <dbl>,
## #   new_incidence_100k <dbl>, case_detection_percent <dbl>, lifeExp <dbl>,
## #   gdpPerCap <dbl>
```

The real power comes in where `group_by()` and `summarize()` are used together. First, write the `group_by()` statement. Then pipe the result to a call to `summarize()`.

Let's use this workflow to get the median incidence_100k for each who_region


```r
joined %>% 
  group_by(who_region) %>% 
  summarize(medInc = median(incidence_100k))
```

```
## # A tibble: 6 x 2
##   who_region medInc
##   <chr>       <dbl>
## 1 AFR         254  
## 2 AMR          31  
## 3 EMR          32  
## 4 EUR          12.5
## 5 SEA         256  
## 6 WPR          87.5
```

While we're at it, let's calculate the median gdpPerCap too


```r
joined %>% 
  group_by(who_region) %>% 
  summarize(medInc = median(incidence_100k),
            medgdp = median(gdpPerCap))
```

```
## # A tibble: 6 x 3
##   who_region medInc medgdp
##   <chr>       <dbl>  <dbl>
## 1 AFR         254    1110.
## 2 AMR          31    7382.
## 3 EMR          32    4493.
## 4 EUR          12.5 26653.
## 5 SEA         256    1697.
## 6 WPR          87.5 15843.
```

Put the output from the above in order by increasing median gdp


```r
joined %>% 
  group_by(who_region) %>% 
  summarize(medInc = median(incidence_100k),
            medgdp = median(gdpPerCap)) %>%
  arrange(medgdp)
```

```
## # A tibble: 6 x 3
##   who_region medInc medgdp
##   <chr>       <dbl>  <dbl>
## 1 AFR         254    1110.
## 2 SEA         256    1697.
## 3 EMR          32    4493.
## 4 AMR          31    7382.
## 5 WPR          87.5 15843.
## 6 EUR          12.5 26653.
```

## EXERCISE 5

1. Calculate a correlation coefficient between gdp and incidence for each continent in 2007. *Hint*: `filter()`, `group_by()`, `summarize()` using the function cor()


```r
joined %>%
  filter(year == 2007) %>%
  group_by(continent) %>%
  summarize(cor = cor(gdpPerCap, incidence_100k))
```

```
## # A tibble: 5 x 2
##   continent     cor
##   <chr>       <dbl>
## 1 Africa     0.0862
## 2 Americas  -0.431 
## 3 Asia      -0.485 
## 4 Europe    -0.614 
## 5 Oceania   -1
```

2. In #1, there are interesting correlation coefficients for Africa and for Oceania. Let's investigate a little more for Africa. Within Africa and 2007, sort the dataset by gdp and then look through the incidence_100k. Use select() to limit your view to the variables of interest *Hint:* `filter()`, `arrange()`, `select()`


```r
joined %>% 
  filter(continent == "Africa" & year == 2007) %>% 
  arrange(gdpPerCap) %>%
  select(country, gdpPerCap, incidence_100k)
```

```
## # A tibble: 51 x 3
##    country                  gdpPerCap incidence_100k
##    <chr>                        <dbl>          <dbl>
##  1 Congo, Dem. Rep.              278.            327
##  2 Liberia                       415.            277
##  3 Burundi                       430.            169
##  4 Zimbabwe                      470.            527
##  5 Guinea-Bissau                 579.            367
##  6 Niger                         620.            129
##  7 Eritrea                       641.            145
##  8 Ethiopia                      691.            310
##  9 Central African Republic      706.            561
## 10 Gambia                        753.            186
## # … with 41 more rows
```

```r
joined %>%
  filter(country == "Algeria" & year == 2007) %>%
  summarize(cor = cor(gdpPerCap, incidence_100k))
```

```
## # A tibble: 1 x 1
##     cor
##   <dbl>
## 1    NA
```

3. Investigate the r = -1 correlation we saw from Oceania *Hint:* `filter()`. r = -1 is very rare. Why did we get this correlation coefficient?


```
## # A tibble: 2 x 11
##   country  year who_region continent    pop incidence_100k incidence_number
##   <chr>   <dbl> <chr>      <chr>      <dbl>          <dbl>            <dbl>
## 1 Austra…  2007 WPR        Oceania   2.04e7            6.1             1300
## 2 New Ze…  2007 WPR        Oceania   4.12e6            7.4              320
## # … with 4 more variables: new_incidence_100k <dbl>,
## #   case_detection_percent <dbl>, lifeExp <dbl>, gdpPerCap <dbl>
```

4. Do countries with low lifeExp also have high rates of tb? Look for this trend by who_region only in 2007 _Hint:_ 3 pipes: `filter`, `group_by`, `summarize`.


```
## # A tibble: 6 x 2
##   who_region    cor
##   <chr>       <dbl>
## 1 AFR        -0.551
## 2 AMR        -0.792
## 3 EMR        -0.701
## 4 EUR        -0.644
## 5 SEA        -0.285
## 6 WPR        -0.836
```

# Further Resources

1. The [**_R for Data Science_ book**](http://r4ds.had.co.nz) is a fabulous resource for learning to do data science in R.

2. There are cheatsheets available on the [RStudio website](https://www.rstudio.com/resources/cheatsheets/) for **tidyr**, **dplyr**, and **stringr**, among others. They are excellent quick reference guides for what we learned today.

3. Resources for Regular Expressions:
For a nice cheatsheet for writing regular expressions in R, see a [Regex cheatsheet](http://www.cbs.dtu.dk/courses/27610/regular-expressions-cheat-sheet-v2.pdf). Jenny Bryan has created a nice website tutorial for learning to use [Regular Expressions in R](http://stat545.com/block022_regular-expression.html).
