# Основы обработки данных с помощью R и Dplyr

## Цель работы

Развить практические навыки использования функций обработки данных пакета dplyr – функции select(), filter(), mutate(), arrange(), group_by()

## Исходные данные

1.  Программное обеспечение Windows 10

2.  Rstudio Desktop

3.  Интерпретатор языка R 4.4.2

4.  Программные пакеты nycflights13 и dplyr

## План

1.  Установить программный пакет dplyr

2.  Проанализировать набор данных

3.  Ответить на вопросы

## Шаги

1.  Для начала прохождения курса необходимо установить программный пакет nycflights13

```{r}
install.packages('nycflights13')
```
2. Далее подключаем библеотеки nycflights13 и dplyr.

```{r}
library(nycflights13)
library(dplyr)
```

3. Выполнение заданий.

a\) Сколько встроенных в пакет nycflights13 датафреймов?
```{r}
length(data(package="nycflights13")$results[, "Item"])
```

```{r}
[1] 5
```

b\) Сколько строк в каждом датафрейме?

```{r}
list(nrow(airlines), nrow(airports), nrow(flights), nrow(planes), nrow(weather))
```

```{r}
[1] 5

[[1]]
[1] 16

[[2]]
[1] 1458

[[3]]
[1] 336776

[[4]]
[1] 3322

[[5]]
[1] 26115

```
c\) Сколько столбцов в каждом датафрейме?

```{r}
list(ncol(airlines), ncol(airports), ncol(flights), ncol(planes), ncol(weather))
```

```{r}
[[1]]
[1] 2

[[2]]
[1] 8

[[3]]
[1] 19

[[4]]
[1] 9

[[5]]
[1] 15
```

d\) Как просмотреть примерный вид датафрейма?

```{r}
flights %>% glimpse()
```

```{r}
Rows: 336,776
Columns: 19
$ year           <int> 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013,…
$ month          <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
$ day            <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
$ dep_time       <int> 517, 533, 542, 544, 554, 554, 555, 557, 557, 558, 558, 558, 558, …
$ sched_dep_time <int> 515, 529, 540, 545, 600, 558, 600, 600, 600, 600, 600, 600, 600, …
$ dep_delay      <dbl> 2, 4, 2, -1, -6, -4, -5, -3, -3, -2, -2, -2, -2, -2, -1, 0, -1, 0…
$ arr_time       <int> 830, 850, 923, 1004, 812, 740, 913, 709, 838, 753, 849, 853, 924,…
$ sched_arr_time <int> 819, 830, 850, 1022, 837, 728, 854, 723, 846, 745, 851, 856, 917,…
$ arr_delay      <dbl> 11, 20, 33, -18, -25, 12, 19, -14, -8, 8, -2, -3, 7, -14, 31, -4,…
$ carrier        <chr> "UA", "UA", "AA", "B6", "DL", "UA", "B6", "EV", "B6", "AA", "B6",…
$ flight         <int> 1545, 1714, 1141, 725, 461, 1696, 507, 5708, 79, 301, 49, 71, 194…
$ tailnum        <chr> "N14228", "N24211", "N619AA", "N804JB", "N668DN", "N39463", "N516…
$ origin         <chr> "EWR", "LGA", "JFK", "JFK", "LGA", "EWR", "EWR", "LGA", "JFK", "L…
$ dest           <chr> "IAH", "IAH", "MIA", "BQN", "ATL", "ORD", "FLL", "IAD", "MCO", "O…
$ air_time       <dbl> 227, 227, 160, 183, 116, 150, 158, 53, 140, 138, 149, 158, 345, 3…
$ distance       <dbl> 1400, 1416, 1089, 1576, 762, 719, 1065, 229, 944, 733, 1028, 1005…
$ hour           <dbl> 5, 5, 5, 5, 6, 5, 6, 6, 6, 6, 6, 6, 6, 6, 6, 5, 6, 6, 6, 6, 6, 6,…
$ minute         <dbl> 15, 29, 40, 45, 0, 58, 0, 0, 0, 0, 0, 0, 0, 0, 0, 59, 0, 0, 0, 0,…
$ time_hour      <dttm> 2013-01-01 05:00:00, 2013-01-01 05:00:00, 2013-01-01 05:00:00
```
e\) Сколько компаний-перевозчиков (carrier) учитывают эти наборы данных (представлено в наборах данных)?

```{r}
flights %>% filter(!is.na(carrier)) %>% distinct(carrier) %>% nrow()
```

```{r}
[1] 16
```
f\) Сколько рейсов принял аэропорт John F Kennedy Intl в мае?

```{r}
flights %>% filter(origin == "JFK", month == 5) %>% nrow()
```

```{r}
[1] 9397
```

g\) Какой самый северный аэропорт?

```{r}
airports %>% arrange(desc(lat)) %>% slice(1)
```

```{r}
# A tibble: 1 × 8
  faa   name                      lat   lon   alt    tz dst   tzone
  <chr> <chr>                   <dbl> <dbl> <dbl> <dbl> <chr> <chr>
1 EEN   Dillant Hopkins Airport  72.3  42.9   149    -5 A     <NA>
```

h\) Какой аэропорт самый высокогорный (находится выше всех над уровнем моря)?

```{r}
airports %>% arrange(desc(alt)) %>% slice(1)
```

```{r}
# A tibble: 1 × 8
  faa   name        lat   lon   alt    tz dst   tzone         
  <chr> <chr>     <dbl> <dbl> <dbl> <dbl> <chr> <chr>         
1 TEX   Telluride  38.0 -108.  9078    -7 A     America/Denver
```

i\) Какие бортовые номера у самых старых самолетов?

```{r}
planes %>% arrange(desc(year)) %>% select(tailnum, year)
```

```{r}
# A tibble: 10 × 9
   tailnum  year type              manufacturer model engines seats speed engine
   <chr>   <int> <chr>             <chr>        <chr>   <int> <int> <int> <chr> 
 1 N381AA   1956 Fixed wing multi… DOUGLAS      DC-7…       4   102   232 Recip…
 2 N201AA   1959 Fixed wing singl… CESSNA       150         1     2    90 Recip…
 3 N567AA   1959 Fixed wing singl… DEHAVILLAND  OTTE…       1    16    95 Recip…
 4 N378AA   1963 Fixed wing singl… CESSNA       172E        1     4   105 Recip…
 5 N575AA   1963 Fixed wing singl… CESSNA       210-…       1     6    NA Recip…
 6 N14629   1965 Fixed wing multi… BOEING       737-…       2   149    NA Turbo…
 7 N615AA   1967 Fixed wing multi… BEECH        65-A…       2     9   202 Turbo…
 8 N425AA   1968 Fixed wing singl… PIPER        PA-2…       1     4   107 Recip…
 9 N383AA   1972 Fixed wing multi… BEECH        E-90        2    10    NA Turbo…
10 N364AA   1973 Fixed wing multi… CESSNA       310Q        2     6   167 Recip…
```

g\) Какая средняя температура воздуха была в сентябре в аэропорту John F Kennedy Intl (в градусах Цельсия).

```{r}
weather %>% filter(origin == "JFK", month == 9) %>% summarise(avg_temp = mean((temp - 32) * 5 / 9, na.rm = TRUE))
```

```{r}
# A tibble: 1 × 1
     mt
  <dbl>
1  19.4
```

k\) Самолеты какой авиакомпании совершили больше всего вылетов в июне?

```{r}
flights %>% filter(month == 6) %>% count(carrier, sort = TRUE)
```

```{r}
# A tibble: 1 × 2
  carrier     n
  <chr>   <int>
1 UA       4975
```

l\) Самолеты какой авиакомпании задерживались чаще других в 2013 году?

```{r}
flights %>% filter(dep_delay > 0 & year == 2013) %>% count(carrier, sort = TRUE)
```

```{r}
# A tibble: 1 × 2
  carrier     n
  <chr>   <int>
1 UA      27261
```

## Оценка результата

В результате работы была использована библиотека nycflights13 и drlyr и были выполнены задания с использованием набора данных nycflights13

## Вывод

Были изучены проанализированны данные набора nycflights13. Были выполнены поставленные практикой задачи.