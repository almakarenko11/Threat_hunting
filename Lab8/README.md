# Практика в DuckDB


# Анализ данных сетевого трафика с использованием аналитической in-memory СУБД DuckDB

## Цель работы

1.  Изучить возможности СУБД DuckDB для обработки и анализ больших
    данных
2.  Получить навыки применения DuckDB совместно с языком
    программирования R
3.  Получить навыки анализа метаинфомации о сетевом трафике
4.  Получить навыки применения облачных технологий хранения, подготовки
    и анализа данных: Yandex Object Storage, Rstudio Server.

## Исходные данные

1.  Программное обеспечение Windows 10
2.  Данные tm_data.pqt
3.  Библиотека DuckDB
4.  Библиотека dplyr
5.  Rstudio Server

## План

1.  Импортировать данные с помощью функции download.file
2.  Занести данные в таблицу DuckDB
3.  Выполнить задания

## Шаги

1.  Скачиваем данные с поомщью функции download.file

``` r
#options(timeout = 1000000)
#download.file("https://storage.yandexcloud.net/arrow-datasets/tm_data.pqt",destfile = "tm_data.pqt")
```

2.  Загружаем данные пакета в таблицу tbl

``` r
library(duckdb)
```

    Warning: package 'duckdb' was built under R version 4.4.2

    Loading required package: DBI

    Warning: package 'DBI' was built under R version 4.4.2

``` r
library(dplyr)
```

    Warning: package 'dplyr' was built under R version 4.4.2


    Attaching package: 'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

``` r
con <- dbConnect(duckdb())
dbExecute(con,"CREATE TABLE tbl as SELECT * FROM read_parquet('tm_data.pqt')")
```

    [1] 105747730

2.  Приступаем к выполнению заданий

<!-- -->

1.  Найдите утечку данных из вашей сети

``` r
dbGetQuery(con,"SELECT src FROM tbl
WHERE (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%') 
AND NOT (dst LIKE '12.%' AND dst LIKE '13.%' AND dst LIKE '14.%')
GROUP BY src
order by sum(bytes) desc
limit 1") %>% knitr::kable()
```

| src          |
|:-------------|
| 13.37.84.125 |

2.  Найдите утечку данных 2

``` r
dbGetQuery(con,"SELECT 
    time,
    COUNT(*) AS trafictime
FROM (
    SELECT 
        timestamp,
        src,
        dst,
        bytes,
        (
            (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%')
            AND (dst NOT LIKE '12.%' AND dst NOT LIKE '13.%' AND dst NOT LIKE '14.%')
        ) AS trafic,
        EXTRACT(HOUR FROM epoch_ms(CAST(timestamp AS BIGINT))) AS time
    FROM tbl
) sub
WHERE trafic = TRUE AND time BETWEEN 0 AND 24
GROUP BY time
ORDER BY trafictime DESC;") %>% knitr::kable()
```

| time | trafictime |
|-----:|-----------:|
|   16 |    4490576 |
|   22 |    4489703 |
|   18 |    4489386 |
|   23 |    4488093 |
|   19 |    4487345 |
|   21 |    4487109 |
|   17 |    4483578 |
|   20 |    4482712 |
|   13 |     169617 |
|    7 |     169241 |
|    0 |     169068 |
|    3 |     169050 |
|   14 |     169028 |
|    6 |     169015 |
|   12 |     168892 |
|   10 |     168750 |
|    2 |     168711 |
|   11 |     168684 |
|    1 |     168539 |
|    4 |     168422 |
|   15 |     168355 |
|    5 |     168283 |
|    9 |     168283 |
|    8 |     168205 |

``` r
dbGetQuery(con,"
SELECT src
FROM (
    SELECT src, SUM(bytes) AS total_bytes
    FROM (
        SELECT *,
            EXTRACT(HOUR FROM epoch_ms(CAST(timestamp AS BIGINT))) AS time
        FROM tbl
    ) sub
    WHERE src <> '13.37.84.125'
        AND (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%')
        AND (dst NOT LIKE '12.%' AND dst NOT LIKE '13.%' AND dst NOT LIKE '14.%')
        AND time BETWEEN 1 AND 15
    GROUP BY src
) grp
ORDER BY total_bytes DESC
LIMIT 1;") %>% knitr::kable()
```

| src         |
|:------------|
| 12.55.77.96 |

3.  Найдите утечку данных 3

``` r
dbExecute(con,"CREATE TEMPORARY TABLE task31 AS
SELECT src, bytes, port
FROM tbl
WHERE src <> '13.37.84.125'
    AND src <> '12.55.77.96'
    AND (src LIKE '12.%' OR src LIKE '13.%' OR src LIKE '14.%')
    AND (dst NOT LIKE '12.%' AND dst NOT LIKE '13.%' AND dst NOT LIKE '14.%');")
```

    [1] 38498353

``` r
dbGetQuery(con,"SELECT port, AVG(bytes) AS mean_bytes, MAX(bytes) AS max_bytes, SUM(bytes) AS sum_bytes, MAX(bytes) - AVG(bytes) AS Raz
FROM task31
GROUP BY port
HAVING MAX(bytes) - AVG(bytes) != 0
ORDER BY Raz DESC
LIMIT 1;") %>% knitr::kable()
```

| port | mean_bytes | max_bytes |   sum_bytes |    Raz |
|-----:|-----------:|----------:|------------:|-------:|
|   37 |   35089.99 |    209402 | 32136394510 | 174312 |

``` r
dbGetQuery(con,"SELECT src
FROM (
    SELECT src, AVG(bytes) AS mean_bytes
    FROM task31
    WHERE port = 37
    GROUP BY src
) AS task32
ORDER BY mean_bytes DESC
LIMIT 1;") %>% knitr::kable()
```

| src          |
|:-------------|
| 14.31.107.42 |

## Оценка результата

Был скачан и проанализирован пакет данных tm_data, были выполнены три
задания.

## Вывод

Мы познакомились с DuckDB и применили его для анализа сетевого трафика.
