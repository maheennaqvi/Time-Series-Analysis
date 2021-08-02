# Time Series with R
library(tidyverse)
library(lubridate)
library(tidyquant)

bike_orderlines_tbl <- read_rds("00_data/bike_sales/data_wrangled/bike_orderlines.rds")

glimpse(bike_orderlines_tbl)

order_date_tbl <- bike_orderlines_tbl %>%
    select(order_date)

order_date_tbl %>%
    pull(order_date) %>%
    class()

order_date_tbl %>%
    mutate(order_date_chr = as.character(order_date)) %>%
    mutate(order_date_chr2 = order_date_chr %>% str_c(" 00:00:00")) %>%
    
    mutate(order_date_date = order_date_chr %>% ymd()) %>%
    mutate(order_date_dttm = order_date_chr2 %>% ymd_hms())

"06/01/18" %>% mdy() %>% class()

"06/01/18 12:30:15" %>% mdy_hms() %>% class()

"January 1, 1985" %>% mdy()

"2011-01-01" %>% ymd() %>% year()

"2011-01-01" %>% ymd() %>% month(label = TRUE, abbr = FALSE) 

"2011-01-01" %>% ymd() %>% wday(label = TRUE, abbr = FALSE)

"2011-01-01" %>% ymd() %>% day()

now()

today()


today() + days(12)

today() + ddays(12)

today() + years(4)

today() + dyears(4)

i <- interval(today(), today() + ddays(12)) 

i / ddays(1) 
i / dminutes(1)

order_date_tbl %>%
    mutate(today = today()) %>%
    mutate(diff_days = interval(order_date, today) / ddays(1))

bike_sales_y_tbl <- bike_orderlines_tbl %>%
    select(order_date, total_price) %>%
    
    mutate(order_date = ymd(order_date)) %>%
    mutate(year = year(order_date)) %>%
    group_by(year) %>%
    summarize(sales = sum(total_price)) %>%
    ungroup()

bike_sales_y_tbl

bike_sales_m_tbl <- bike_orderlines_tbl %>%
    select(order_date, total_price) %>%
    mutate(order_date = ymd(order_date)) %>%
    mutate(
        year  = year(order_date),
        month = month(order_date, label = TRUE, abbr = TRUE)
    ) %>%
    group_by(year, month) %>%
    summarize(sales = sum(total_price)) %>%
    ungroup()

bike_sales_m_tbl

bike_orderlines_tbl %>%
    select(order_date, total_price) %>%
    mutate(order_date = ymd(order_date)) %>%
    mutate(year_month = floor_date(order_date, unit = "month")) %>%
    group_by(year_month) %>%
    summarize(sales = sum(total_price))
bike_sales_y_tbl %>%
    mutate(sales_lag_1 = lag(sales, n = 1)) %>%
    mutate(sales_lag_1 = case_when(
        is.na(sales_lag_1) ~ sales,
        TRUE ~ sales_lag_1
    )) %>%
    mutate(diff_1 = sales - sales_lag_1) %>%
    mutate(pct_diff_1 = diff_1 / sales_lag_1) %>%
    mutate(pct_diff_1_chr = scales::percent(pct_diff_1))

calculate_pct_diff <- function(data) {
    
    data %>% 
        mutate(sales_lag_1 = lag(sales, n = 1)) %>%
        mutate(sales_lag_1 = case_when(
            is.na(sales_lag_1) ~ sales,
            TRUE ~ sales_lag_1
        )) %>%
        mutate(diff_1 = sales - sales_lag_1) %>%
        mutate(pct_diff_1 = diff_1 / sales_lag_1) %>%
        mutate(pct_diff_1_chr = scales::percent(pct_diff_1))
    
}

bike_sales_m_tbl %>%
    calculate_pct_diff()

bike_sales_y_tbl %>%
    mutate(sales_2011 = first(sales)) %>%
    mutate(diff_2011  = sales - sales_2011) %>%
    mutate(pct_diff_2011 = diff_2011 / sales_2011) %>%
    mutate(pct_diff_2011_chr = scales::percent(pct_diff_2011))

bike_sales_m_tbl %>%
    
    group_by(year) %>%
    
    mutate(sales_jan = first(sales)) %>% 
    mutate(
        diff_jan         = sales - sales_jan,
        pct_diff_jan     = diff_jan / sales_jan,
        pct_diff_jan_chr = scales::percent(pct_diff_jan)
    )

bike_sales_y_tbl %>%
    mutate(cumulative_sales = cumsum(sales)) %>%
    mutate(cumulative_sales_pct = cumulative_sales / sum(sales)) %>%
    mutate(cumulative_sales_pct_chr = cumulative_sales_pct %>% scales::percent())

bike_sales_m_tbl %>%
    
    group_by(year) %>%
    
    mutate(cumulative_sales = cumsum(sales)) %>%
    mutate(cumulative_sales_pct = cumulative_sales / sum(sales)) %>%
    mutate(cumulative_sales_pct_chr = scales::percent(cumulative_sales_pct))

bike_sales_m_tbl %>%
    mutate(roll_mean_3 = rollmean(sales, k = 3, na.pad = TRUE, align = "right", fill = NA)) %>%
    
    mutate(roll_mean_6 = rollmean(sales, k = 6, na.pad = TRUE, align = "right", fill = NA))

bike_orderlines_tbl %>%
    
    mutate(order_date= ymd(order_date)) %>%
    filter(order_date %>% between(left = ymd("2012-01-01"), right = ymd("2013-12-31"))) 

bike_orderlines_tbl %>%
    
    mutate(order_date= ymd(order_date)) %>%
    filter(year(order_date) %in% c(2012,2013))
