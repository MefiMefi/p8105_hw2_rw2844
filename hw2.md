p8105\_hw2\_rw2844
================
Renjie Wei
2020/9/24

  - [Problem 1](#problem-1)
  - [Problem 2](#problem-2)
  - [Problem 3](#problem-3)

# Problem 1

Read and create `mr_trash` dataset.

``` r
trash_path = "./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx"
mr_trash = 
  read_excel(
  path = trash_path,
  sheet = "Mr. Trash Wheel",
  range = cell_cols("A:N")) %>%
  janitor::clean_names() %>% 
  drop_na(dumpster) %>% 
  mutate(
    sports_balls = round(sports_balls),
    sports_balls = as.integer(sports_balls)
  )
```

Read and clean the precipitation data.

``` r
precip_18 =
  read_excel(
    "./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx",
        sheet = "2018 Precipitation",
        skip = 1
    ) %>% 
    janitor::clean_names() %>% 
    drop_na(month) %>% 
    mutate(year = 2018) %>% 
    relocate(year) 

precip_17 = 
    read_excel(
        "./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx",
        sheet = "2017 Precipitation",
        skip = 1
    ) %>% 
    janitor::clean_names() %>% 
    drop_na(month) %>% 
    mutate(year = 2017) %>% 
    relocate(year)
```

Combine two datasets.

``` r
month_df = 
    tibble(
        month = 1:12,
        month_name = month.name
    )
precip_df =
    bind_rows(precip_18, precip_17)
precip_df = 
    left_join(precip_df, month_df, by = "month")
head(precip_df)
```

    ## # A tibble: 6 x 4
    ##    year month total month_name
    ##   <dbl> <dbl> <dbl> <chr>     
    ## 1  2018     1  0.94 January   
    ## 2  2018     2  4.8  February  
    ## 3  2018     3  2.69 March     
    ## 4  2018     4  4.69 April     
    ## 5  2018     5  9.27 May       
    ## 6  2018     6  4.77 June

This dataset contains information from the Mr. Trashwheel trash
collector in Baltimore, Maryland. As trash enters the inner harbor, the
trashwheel collects that trash, and stores it in a dumpster. The dataset
contains information on year, month, and trash collected, include some
specific kinds of trash. There are a total of 344 rows in our final
dataset. Additional data sheets include month precipitation data.

  - In this dataset:
    
      - The median number of sports balls found in a dumpster in 2017
        was 8
    
      - The total precipitation in 2018 was 70.33 inches.

# Problem 2

Read and clean the NYC Transit data.

``` r
nyc_trans_df = 
  read_csv(
    "./data/NYC_Transit_Subway_Entrance_And_Exit_Data.csv",
  ) %>%
  janitor::clean_names() %>% 
  select(
    .,line, station_name, station_latitude, station_longitude, starts_with("route"), entry, vending, entrance_type, ada
    ) %>% 
  mutate(
    entry = recode(entry, "YES" = TRUE, "NO" = FALSE),
    vending = recode(vending, "YES" = TRUE, "NO" = FALSE),
      )
  
#check whether the variables are right or not
str(nyc_trans_df)
```

    ## tibble [1,868 x 19] (S3: tbl_df/tbl/data.frame)
    ##  $ line             : chr [1:1868] "4 Avenue" "4 Avenue" "4 Avenue" "4 Avenue" ...
    ##  $ station_name     : chr [1:1868] "25th St" "25th St" "36th St" "36th St" ...
    ##  $ station_latitude : num [1:1868] 40.7 40.7 40.7 40.7 40.7 ...
    ##  $ station_longitude: num [1:1868] -74 -74 -74 -74 -74 ...
    ##  $ route1           : chr [1:1868] "R" "R" "N" "N" ...
    ##  $ route2           : chr [1:1868] NA NA "R" "R" ...
    ##  $ route3           : chr [1:1868] NA NA NA NA ...
    ##  $ route4           : chr [1:1868] NA NA NA NA ...
    ##  $ route5           : chr [1:1868] NA NA NA NA ...
    ##  $ route6           : chr [1:1868] NA NA NA NA ...
    ##  $ route7           : chr [1:1868] NA NA NA NA ...
    ##  $ route8           : num [1:1868] NA NA NA NA NA NA NA NA NA NA ...
    ##  $ route9           : num [1:1868] NA NA NA NA NA NA NA NA NA NA ...
    ##  $ route10          : num [1:1868] NA NA NA NA NA NA NA NA NA NA ...
    ##  $ route11          : num [1:1868] NA NA NA NA NA NA NA NA NA NA ...
    ##  $ entry            : logi [1:1868] TRUE TRUE TRUE TRUE TRUE TRUE ...
    ##  $ vending          : logi [1:1868] TRUE TRUE TRUE TRUE TRUE TRUE ...
    ##  $ entrance_type    : chr [1:1868] "Stair" "Stair" "Stair" "Stair" ...
    ##  $ ada              : logi [1:1868] FALSE FALSE FALSE FALSE FALSE FALSE ...

In the chunk above, we loaded the NYC Transit dataset, cleaned the names
of columns, selected the desired variables and changed two character
variables to logical variables —- `entry` and `vending`. There are 1868
rows and 19 columns in the `nyc_trans_df`. The dataset contained the
following variables: line, station\_name, station\_latitude,
station\_longitude, route1, route2, route3, route4, route5, route6,
route7, route8, route9, route10, route11, entry, vending,
entrance\_type, ada. These data are not tidy.

  - **Answers:**
      - There are 465 stations in NYC.
      - And 84 of them are ADA compliant.
      - The proportion of station entrances / exits without vending
        allow entrance is 0.3770492.

Reformat the data:

``` r
# reformat
dist_route_df = station_distinct %>%
  # convert
  mutate_at(vars(route8:route11), as.character) %>% 
  pivot_longer(
    route1:route11,
    names_to = "route",
    values_to = "train")

head(dist_route_df)
```

    ## # A tibble: 6 x 10
    ##   line  station_name station_latitude station_longitu~ entry vending
    ##   <chr> <chr>                   <dbl>            <dbl> <lgl> <lgl>  
    ## 1 4 Av~ 25th St                  40.7            -74.0 TRUE  TRUE   
    ## 2 4 Av~ 25th St                  40.7            -74.0 TRUE  TRUE   
    ## 3 4 Av~ 25th St                  40.7            -74.0 TRUE  TRUE   
    ## 4 4 Av~ 25th St                  40.7            -74.0 TRUE  TRUE   
    ## 5 4 Av~ 25th St                  40.7            -74.0 TRUE  TRUE   
    ## 6 4 Av~ 25th St                  40.7            -74.0 TRUE  TRUE   
    ## # ... with 4 more variables: entrance_type <chr>, ada <lgl>, route <chr>,
    ## #   train <chr>

  - The number of distinct stations serve the A train is 60.
  - The number of distinct stations serve the A train and are ADA
    compliant is 17.

# Problem 3

Read and clean the pols-month dataset:

``` r
pols_df =
  read_csv("./data/fivethirtyeight_datasets/fivethirtyeight_datasets/pols-month.csv") %>%
  janitor::clean_names() %>%
  #separate mon
  separate(
    mon, 
    c("year", "month", "day"), 
    convert = TRUE
    ) %>%
  mutate(month = month.name[month]) %>%
  mutate(
    president = recode(prez_gop, '1' = "gop", '0' = "dem")
    ) %>%
  # remove and organize variables
  select(-prez_dem, -prez_gop, -day) %>%
  relocate(year, month, president)

head(pols_df)
```

    ## # A tibble: 6 x 9
    ##    year month    president gov_gop sen_gop rep_gop gov_dem sen_dem rep_dem
    ##   <int> <chr>    <chr>       <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
    ## 1  1947 January  dem            23      51     253      23      45     198
    ## 2  1947 February dem            23      51     253      23      45     198
    ## 3  1947 March    dem            23      51     253      23      45     198
    ## 4  1947 April    dem            23      51     253      23      45     198
    ## 5  1947 May      dem            23      51     253      23      45     198
    ## 6  1947 June     dem            23      51     253      23      45     198

Read and clean the snp dataset:

``` r
snp_df =
  read_csv("./data/fivethirtyeight_datasets/fivethirtyeight_datasets/snp.csv") %>%
  janitor::clean_names() %>%
  separate(
    date, 
    c("month", "day", "year"), 
    convert = TRUE
    ) %>%
  mutate(month = month.name[month]) %>%
  select(-day) %>%
  relocate(year, month)

head(snp_df)
```

    ## # A tibble: 6 x 3
    ##    year month    close
    ##   <int> <chr>    <dbl>
    ## 1  2015 July     2080.
    ## 2  2015 June     2063.
    ## 3  2015 May      2107.
    ## 4  2015 April    2086.
    ## 5  2015 March    2068.
    ## 6  2015 February 2104.

Read and clean the unemployment dataset:

``` r
unemp_df =
  read_csv("./data/fivethirtyeight_datasets/fivethirtyeight_datasets/unemployment.csv") %>%
  #wide -> long
  pivot_longer(
    Jan:Dec,
    names_to = "month",
    values_to = "unemployment_rate"
  ) %>%
  mutate(month = match(month, month.abb)) %>%
  mutate(month = month.name[month]) %>%
  janitor::clean_names() %>%
  mutate(year = as.integer(year))

head(unemp_df)
```

    ## # A tibble: 6 x 3
    ##    year month    unemployment_rate
    ##   <int> <chr>                <dbl>
    ## 1  1948 January                3.4
    ## 2  1948 February               3.8
    ## 3  1948 March                  4  
    ## 4  1948 April                  3.9
    ## 5  1948 May                    3.5
    ## 6  1948 June                   3.6

Merge them\!

``` r
pols_snp = left_join(
  pols_df, snp_df, by = c("year", "month")
  )
merge_df = 
  left_join(
    pols_snp, unemp_df, by = c("year", "month")
    ) %>%
  relocate(year, month, president, close, unemployment_rate)


head(merge_df)
```

    ## # A tibble: 6 x 11
    ##    year month president close unemployment_ra~ gov_gop sen_gop rep_gop gov_dem
    ##   <int> <chr> <chr>     <dbl>            <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
    ## 1  1947 Janu~ dem          NA               NA      23      51     253      23
    ## 2  1947 Febr~ dem          NA               NA      23      51     253      23
    ## 3  1947 March dem          NA               NA      23      51     253      23
    ## 4  1947 April dem          NA               NA      23      51     253      23
    ## 5  1947 May   dem          NA               NA      23      51     253      23
    ## 6  1947 June  dem          NA               NA      23      51     253      23
    ## # ... with 2 more variables: sen_dem <dbl>, rep_dem <dbl>

  - In the `pols_df`, there are822rows and 9 columns. The year ranges is
    1947 to 2015. The variables are: year, month, president, gov\_gop,
    sen\_gop, rep\_gop, gov\_dem, sen\_dem, rep\_dem.
      - `mon`: date of the count
      - `prez_gop`: indicator of whether the president was republican on
        the associated date (1 = yes, 0 = no)
      - `gov_gop`: the number of republican governors on the associated
        date
      - `sen_gop`: the number of republican senators on the associated
        date
      - `rep_gop`: the number of republican representatives on the
        associated date
      - `prez_dem`: indicator of whether the president was democratic on
        the associated date (1 = yes, 0 = no)
      - `gov_dem`: the number of democratic governors on the associated
        date
      - `sen_dem`: the number of democratic senators on the associated
        date
      - `rep_dem`: the number of democratic representatives on the
        associated date
  - For `snp_df`,there are787rows and 3 columns. The year ranges is 1950
    to 2015. The variables are: year, month, close.
      - `close`:the closing values of the S\&P stock index on the
        associated date
  - For `unemp_df`,there are816rows and 3 columns. The year ranges is
    1948 to 2015. The variables are: year, month, unemployment\_rate.
      - `Jan:Dec`:percentage of unemployment in January-December of the
        associated year
  - For the `merge_df`, there are822rows and 11 columns. The year ranges
    is 1947 to 2015.
      - This merged dataset tells us the number of national politicians,
        the closing value, and the unemployment rate.
