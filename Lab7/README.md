# Анализ данных сетевого трафика при помощи библиотеки Arrow


## Цель

1.  Изучить возможности технологии [Apache
    Arrow](https://arrow.apache.org/) для обработки и анализ больших
    данных
2.  Получить навыки применения Arrow совместно с языком программирования
    R
3.  Получить навыки анализа `метаинфомации о сетевом трафике`
4.  Получить навыки применения облачных технологий хранения, подготовки
    и анализа данных: [Yandex Object
    Storage](https://yandex.cloud/ru/docs/storage/), [Rstudio
    Server](https://posit.co/products/open-source/rstudio-server/).

## ️Исходные данные

1.  R 4.4.1
2.  RStudio 2024.04.2+764
3.  Yandex Cloud

## ️Общий план выполнения

Используя язык программирования `R`, библиотеку `arrow` и облачную
`IDE Rstudio Server`, развернутую в `Yandex Cloud`, выполнить задания и
составить отчет.

## Содержание ЛР

### Получение данных

1.  **Установка пакета**

``` r
library(arrow)
```

    Warning: пакет 'arrow' был собран под R версии 4.4.2


    Присоединяю пакет: 'arrow'

    Следующий объект скрыт от 'package:utils':

        timestamp

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

1.  **Загрузка данных**

``` r
data <- read_parquet("./data/tm_data.pqt")
```

1.  **Проверка получения данных**

``` r
glimpse(data)
```

    Rows: 105,747,730
    Columns: 5
    $ timestamp <dbl> 1.578326e+12, 1.578326e+12, 1.578326e+12, 1.578326e+12, 1.57…
    $ src       <chr> "13.43.52.51", "16.79.101.100", "18.43.118.103", "15.71.108.…
    $ dst       <chr> "18.70.112.62", "12.48.65.39", "14.51.30.86", "14.50.119.33"…
    $ port      <int> 40, 92, 27, 57, 115, 92, 65, 123, 79, 72, 123, 123, 22, 118,…
    $ bytes     <int> 57354, 11895, 898, 7496, 20979, 8620, 46033, 1500, 979, 1036…

### Задание 1: Найдите утечку данных из Вашей сети

Важнейшие документы с результатами нашей исследовательской деятельности
в области создания вакцин скачиваются в виде больших заархивированных
дампов. Один из хостов в нашей сети используется для пересылки этой
информации – он пересылает гораздо больше информации на внешние ресурсы
в Интернете, чем остальные компьютеры нашей сети. Определите его
IP-адрес

``` r
data %>% 
  filter(grepl("^1[2|3|4]\\..+", src) & !grepl("^1[2|3|4]\\..+", dst)) %>%
  group_by(src) %>%
  summarise(traffic = sum(bytes)) %>%
  arrange(desc(traffic)) %>%
  head(1) %>%
  select(src) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">src</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">13.37.84.125</td>
</tr>
</tbody>
</table>

### Задание 2: Найдите утечку данных 2

Другой атакующий установил автоматическую задачу в системном
планировщике cron для экспорта содержимого внутренней wiki системы. Эта
система генерирует большое количество трафика в нерабочие часы, больше
чем остальные хосты. Определите IP этой системы. Известно, что ее IP
адрес отличается от нарушителя из предыдущей задачи.

``` r
data %>%
  filter(grepl("^1[2|3|4]\\..+", src) & !grepl("^1[2|3|4]\\..+", dst)) %>%
  mutate(timestamp = as.POSIXct(timestamp / 1000, origin = "1970-01-01", tz = "UTC")) %>%
  mutate(hour = format(timestamp, "%H")) %>%
  group_by(hour) %>%
  summarise(sd = sd(bytes)) %>%
  arrange(desc(sd)) %>%
  head() %>%
  select(hour) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">hour</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">07</td>
</tr>
<tr class="even">
<td style="text-align: left;">05</td>
</tr>
<tr class="odd">
<td style="text-align: left;">12</td>
</tr>
<tr class="even">
<td style="text-align: left;">15</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00</td>
</tr>
<tr class="even">
<td style="text-align: left;">10</td>
</tr>
</tbody>
</table>

``` r
data %>%
  filter(grepl("^1[2|3|4]\\..+", src) & !grepl("^1[2|3|4]\\..+", dst)) %>%
  mutate(timestamp = as.POSIXct(timestamp / 1000, origin = "1970-01-01", tz = "UTC")) %>%
  mutate(hour = format(timestamp, "%H")) %>%
  filter(hour == "07") %>%
  group_by(src) %>%
  summarise(traffic = sum(bytes)) %>%
  arrange(desc(traffic)) %>%
  head(1) %>%
  select(src) %>%
  knitr::kable() 
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">src</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">12.55.77.96</td>
</tr>
</tbody>
</table>

### Задание 3: Найдите утечку данных 3

Еще один нарушитель собирает содержимое электронной почты и отправляет в
Интернет используя порт, который обычно используется для другого типа
трафика. Атакующий пересылает большое количество информации используя
этот порт, которое нехарактерно для других хостов, использующих этот
номер порта. Определите IP этой системы. Известно, что ее IP адрес
отличается от нарушителей из предыдущих задач.

``` r
data %>%
  filter(grepl("^1[2|3|4]\\..+", src) & !grepl("^1[2|3|4]\\..+", dst)) %>%
  group_by(port) %>%
  summarise(sd = sd(bytes),
            min = min(bytes),
            max = max(bytes),
            med = median(bytes),
            men = mean(bytes),
            x = men - med) %>%
  arrange(desc(sd)) %>%
  knitr::kable()
```

<table>
<colgroup>
<col style="width: 7%" />
<col style="width: 17%" />
<col style="width: 10%" />
<col style="width: 10%" />
<col style="width: 13%" />
<col style="width: 19%" />
<col style="width: 20%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: right;">port</th>
<th style="text-align: right;">sd</th>
<th style="text-align: right;">min</th>
<th style="text-align: right;">max</th>
<th style="text-align: right;">med</th>
<th style="text-align: right;">men</th>
<th style="text-align: right;">x</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">31</td>
<td style="text-align: right;">68056.17487</td>
<td style="text-align: right;">445912</td>
<td style="text-align: right;">843091</td>
<td style="text-align: right;">674457.0</td>
<td style="text-align: right;">670532.12644</td>
<td style="text-align: right;">-3924.8735632</td>
</tr>
<tr class="even">
<td style="text-align: right;">95</td>
<td style="text-align: right;">36352.14684</td>
<td style="text-align: right;">218617</td>
<td style="text-align: right;">480587</td>
<td style="text-align: right;">358552.5</td>
<td style="text-align: right;">358248.57264</td>
<td style="text-align: right;">-303.9273625</td>
</tr>
<tr class="odd">
<td style="text-align: right;">78</td>
<td style="text-align: right;">36129.25403</td>
<td style="text-align: right;">220522</td>
<td style="text-align: right;">480609</td>
<td style="text-align: right;">358306.0</td>
<td style="text-align: right;">357866.35709</td>
<td style="text-align: right;">-439.6429078</td>
</tr>
<tr class="even">
<td style="text-align: right;">21</td>
<td style="text-align: right;">36045.36403</td>
<td style="text-align: right;">202204</td>
<td style="text-align: right;">493178</td>
<td style="text-align: right;">356865.0</td>
<td style="text-align: right;">357520.90742</td>
<td style="text-align: right;">655.9074237</td>
</tr>
<tr class="odd">
<td style="text-align: right;">36</td>
<td style="text-align: right;">35781.95006</td>
<td style="text-align: right;">235843</td>
<td style="text-align: right;">493179</td>
<td style="text-align: right;">358183.0</td>
<td style="text-align: right;">358407.11873</td>
<td style="text-align: right;">224.1187262</td>
</tr>
<tr class="even">
<td style="text-align: right;">32</td>
<td style="text-align: right;">35639.62337</td>
<td style="text-align: right;">241941</td>
<td style="text-align: right;">481352</td>
<td style="text-align: right;">357815.0</td>
<td style="text-align: right;">357550.10909</td>
<td style="text-align: right;">-264.8909058</td>
</tr>
<tr class="odd">
<td style="text-align: right;">92</td>
<td style="text-align: right;">27292.92257</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">182077</td>
<td style="text-align: right;">30743.0</td>
<td style="text-align: right;">35125.88648</td>
<td style="text-align: right;">4382.8864821</td>
</tr>
<tr class="even">
<td style="text-align: right;">52</td>
<td style="text-align: right;">27289.76175</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">184543</td>
<td style="text-align: right;">30642.0</td>
<td style="text-align: right;">35087.10449</td>
<td style="text-align: right;">4445.1044943</td>
</tr>
<tr class="odd">
<td style="text-align: right;">105</td>
<td style="text-align: right;">27289.69388</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">197766</td>
<td style="text-align: right;">30596.0</td>
<td style="text-align: right;">35084.28196</td>
<td style="text-align: right;">4488.2819648</td>
</tr>
<tr class="even">
<td style="text-align: right;">56</td>
<td style="text-align: right;">27288.59410</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">183523</td>
<td style="text-align: right;">30740.0</td>
<td style="text-align: right;">35123.72354</td>
<td style="text-align: right;">4383.7235431</td>
</tr>
<tr class="odd">
<td style="text-align: right;">39</td>
<td style="text-align: right;">27282.34838</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">198527</td>
<td style="text-align: right;">30714.0</td>
<td style="text-align: right;">35141.40219</td>
<td style="text-align: right;">4427.4021864</td>
</tr>
<tr class="even">
<td style="text-align: right;">29</td>
<td style="text-align: right;">27279.58460</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">185819</td>
<td style="text-align: right;">30627.0</td>
<td style="text-align: right;">35088.66889</td>
<td style="text-align: right;">4461.6688915</td>
</tr>
<tr class="odd">
<td style="text-align: right;">102</td>
<td style="text-align: right;">27277.06462</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">193588</td>
<td style="text-align: right;">30643.0</td>
<td style="text-align: right;">35089.74549</td>
<td style="text-align: right;">4446.7454895</td>
</tr>
<tr class="even">
<td style="text-align: right;">119</td>
<td style="text-align: right;">27273.09123</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">190151</td>
<td style="text-align: right;">30622.0</td>
<td style="text-align: right;">35097.95656</td>
<td style="text-align: right;">4475.9565632</td>
</tr>
<tr class="odd">
<td style="text-align: right;">57</td>
<td style="text-align: right;">27272.67166</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">180469</td>
<td style="text-align: right;">30648.0</td>
<td style="text-align: right;">35093.04327</td>
<td style="text-align: right;">4445.0432735</td>
</tr>
<tr class="even">
<td style="text-align: right;">44</td>
<td style="text-align: right;">27271.50392</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">179574</td>
<td style="text-align: right;">30663.0</td>
<td style="text-align: right;">35116.48624</td>
<td style="text-align: right;">4453.4862406</td>
</tr>
<tr class="odd">
<td style="text-align: right;">89</td>
<td style="text-align: right;">27271.44609</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">194106</td>
<td style="text-align: right;">30627.0</td>
<td style="text-align: right;">35085.71461</td>
<td style="text-align: right;">4458.7146086</td>
</tr>
<tr class="even">
<td style="text-align: right;">65</td>
<td style="text-align: right;">27270.20892</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">178369</td>
<td style="text-align: right;">30568.0</td>
<td style="text-align: right;">35048.42798</td>
<td style="text-align: right;">4480.4279818</td>
</tr>
<tr class="odd">
<td style="text-align: right;">40</td>
<td style="text-align: right;">27269.76582</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">195144</td>
<td style="text-align: right;">30577.0</td>
<td style="text-align: right;">35073.33497</td>
<td style="text-align: right;">4496.3349749</td>
</tr>
<tr class="even">
<td style="text-align: right;">75</td>
<td style="text-align: right;">27267.70905</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">194650</td>
<td style="text-align: right;">30683.0</td>
<td style="text-align: right;">35096.76154</td>
<td style="text-align: right;">4413.7615428</td>
</tr>
<tr class="odd">
<td style="text-align: right;">81</td>
<td style="text-align: right;">27266.30465</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">192430</td>
<td style="text-align: right;">30721.0</td>
<td style="text-align: right;">35128.34483</td>
<td style="text-align: right;">4407.3448287</td>
</tr>
<tr class="even">
<td style="text-align: right;">114</td>
<td style="text-align: right;">27260.65550</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">184647</td>
<td style="text-align: right;">30600.0</td>
<td style="text-align: right;">35050.11187</td>
<td style="text-align: right;">4450.1118715</td>
</tr>
<tr class="odd">
<td style="text-align: right;">118</td>
<td style="text-align: right;">27260.22522</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">186905</td>
<td style="text-align: right;">30625.0</td>
<td style="text-align: right;">35084.54822</td>
<td style="text-align: right;">4459.5482228</td>
</tr>
<tr class="even">
<td style="text-align: right;">74</td>
<td style="text-align: right;">27257.13246</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">189818</td>
<td style="text-align: right;">30677.0</td>
<td style="text-align: right;">35095.60773</td>
<td style="text-align: right;">4418.6077296</td>
</tr>
<tr class="odd">
<td style="text-align: right;">37</td>
<td style="text-align: right;">27256.00935</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">209402</td>
<td style="text-align: right;">30666.0</td>
<td style="text-align: right;">35088.68089</td>
<td style="text-align: right;">4422.6808915</td>
</tr>
<tr class="even">
<td style="text-align: right;">115</td>
<td style="text-align: right;">27255.49374</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">175763</td>
<td style="text-align: right;">30673.0</td>
<td style="text-align: right;">35106.57147</td>
<td style="text-align: right;">4433.5714701</td>
</tr>
<tr class="odd">
<td style="text-align: right;">55</td>
<td style="text-align: right;">27243.18927</td>
<td style="text-align: right;">21</td>
<td style="text-align: right;">182142</td>
<td style="text-align: right;">30656.0</td>
<td style="text-align: right;">35078.57391</td>
<td style="text-align: right;">4422.5739093</td>
</tr>
<tr class="even">
<td style="text-align: right;">22</td>
<td style="text-align: right;">150.13779</td>
<td style="text-align: right;">385</td>
<td style="text-align: right;">1731</td>
<td style="text-align: right;">1000.0</td>
<td style="text-align: right;">999.39662</td>
<td style="text-align: right;">-0.6033800</td>
</tr>
<tr class="odd">
<td style="text-align: right;">72</td>
<td style="text-align: right;">150.10311</td>
<td style="text-align: right;">384</td>
<td style="text-align: right;">1754</td>
<td style="text-align: right;">1000.0</td>
<td style="text-align: right;">999.54772</td>
<td style="text-align: right;">-0.4522802</td>
</tr>
<tr class="even">
<td style="text-align: right;">50</td>
<td style="text-align: right;">150.08146</td>
<td style="text-align: right;">384</td>
<td style="text-align: right;">1785</td>
<td style="text-align: right;">1000.0</td>
<td style="text-align: right;">999.50720</td>
<td style="text-align: right;">-0.4927954</td>
</tr>
<tr class="odd">
<td style="text-align: right;">79</td>
<td style="text-align: right;">150.04089</td>
<td style="text-align: right;">388</td>
<td style="text-align: right;">1693</td>
<td style="text-align: right;">999.0</td>
<td style="text-align: right;">999.52448</td>
<td style="text-align: right;">0.5244760</td>
</tr>
<tr class="even">
<td style="text-align: right;">80</td>
<td style="text-align: right;">150.00337</td>
<td style="text-align: right;">385</td>
<td style="text-align: right;">1723</td>
<td style="text-align: right;">999.0</td>
<td style="text-align: right;">999.27433</td>
<td style="text-align: right;">0.2743269</td>
</tr>
<tr class="odd">
<td style="text-align: right;">82</td>
<td style="text-align: right;">149.99305</td>
<td style="text-align: right;">384</td>
<td style="text-align: right;">1749</td>
<td style="text-align: right;">999.0</td>
<td style="text-align: right;">999.51180</td>
<td style="text-align: right;">0.5117970</td>
</tr>
<tr class="even">
<td style="text-align: right;">27</td>
<td style="text-align: right;">149.97546</td>
<td style="text-align: right;">384</td>
<td style="text-align: right;">1741</td>
<td style="text-align: right;">1000.0</td>
<td style="text-align: right;">999.54074</td>
<td style="text-align: right;">-0.4592597</td>
</tr>
<tr class="odd">
<td style="text-align: right;">121</td>
<td style="text-align: right;">149.97199</td>
<td style="text-align: right;">384</td>
<td style="text-align: right;">1731</td>
<td style="text-align: right;">1000.0</td>
<td style="text-align: right;">999.54047</td>
<td style="text-align: right;">-0.4595318</td>
</tr>
<tr class="even">
<td style="text-align: right;">68</td>
<td style="text-align: right;">149.97102</td>
<td style="text-align: right;">384</td>
<td style="text-align: right;">1745</td>
<td style="text-align: right;">999.0</td>
<td style="text-align: right;">999.33069</td>
<td style="text-align: right;">0.3306864</td>
</tr>
<tr class="odd">
<td style="text-align: right;">94</td>
<td style="text-align: right;">149.94980</td>
<td style="text-align: right;">385</td>
<td style="text-align: right;">1700</td>
<td style="text-align: right;">999.0</td>
<td style="text-align: right;">999.11294</td>
<td style="text-align: right;">0.1129389</td>
</tr>
<tr class="even">
<td style="text-align: right;">61</td>
<td style="text-align: right;">149.89753</td>
<td style="text-align: right;">386</td>
<td style="text-align: right;">1710</td>
<td style="text-align: right;">999.0</td>
<td style="text-align: right;">999.32067</td>
<td style="text-align: right;">0.3206713</td>
</tr>
<tr class="odd">
<td style="text-align: right;">34</td>
<td style="text-align: right;">149.89080</td>
<td style="text-align: right;">384</td>
<td style="text-align: right;">1840</td>
<td style="text-align: right;">999.0</td>
<td style="text-align: right;">999.49199</td>
<td style="text-align: right;">0.4919898</td>
</tr>
<tr class="even">
<td style="text-align: right;">26</td>
<td style="text-align: right;">149.88514</td>
<td style="text-align: right;">388</td>
<td style="text-align: right;">1701</td>
<td style="text-align: right;">1000.0</td>
<td style="text-align: right;">999.49166</td>
<td style="text-align: right;">-0.5083391</td>
</tr>
<tr class="odd">
<td style="text-align: right;">77</td>
<td style="text-align: right;">149.87677</td>
<td style="text-align: right;">387</td>
<td style="text-align: right;">1720</td>
<td style="text-align: right;">999.0</td>
<td style="text-align: right;">999.44501</td>
<td style="text-align: right;">0.4450065</td>
</tr>
<tr class="even">
<td style="text-align: right;">23</td>
<td style="text-align: right;">149.86922</td>
<td style="text-align: right;">384</td>
<td style="text-align: right;">1737</td>
<td style="text-align: right;">1000.0</td>
<td style="text-align: right;">999.70347</td>
<td style="text-align: right;">-0.2965281</td>
</tr>
<tr class="odd">
<td style="text-align: right;">96</td>
<td style="text-align: right;">149.86518</td>
<td style="text-align: right;">385</td>
<td style="text-align: right;">1739</td>
<td style="text-align: right;">1000.0</td>
<td style="text-align: right;">999.62518</td>
<td style="text-align: right;">-0.3748209</td>
</tr>
<tr class="even">
<td style="text-align: right;">124</td>
<td style="text-align: right;">10.28874</td>
<td style="text-align: right;">42</td>
<td style="text-align: right;">255</td>
<td style="text-align: right;">42.0</td>
<td style="text-align: right;">42.56429</td>
<td style="text-align: right;">0.5642883</td>
</tr>
<tr class="odd">
<td style="text-align: right;">25</td>
<td style="text-align: right;">0.00000</td>
<td style="text-align: right;">42</td>
<td style="text-align: right;">42</td>
<td style="text-align: right;">42.0</td>
<td style="text-align: right;">42.00000</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: right;">42</td>
<td style="text-align: right;">0.00000</td>
<td style="text-align: right;">1337</td>
<td style="text-align: right;">1337</td>
<td style="text-align: right;">1337.0</td>
<td style="text-align: right;">1337.00000</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: right;">51</td>
<td style="text-align: right;">0.00000</td>
<td style="text-align: right;">1337</td>
<td style="text-align: right;">1337</td>
<td style="text-align: right;">1337.0</td>
<td style="text-align: right;">1337.00000</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: right;">90</td>
<td style="text-align: right;">0.00000</td>
<td style="text-align: right;">42</td>
<td style="text-align: right;">42</td>
<td style="text-align: right;">42.0</td>
<td style="text-align: right;">42.00000</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: right;">106</td>
<td style="text-align: right;">0.00000</td>
<td style="text-align: right;">255</td>
<td style="text-align: right;">255</td>
<td style="text-align: right;">255.0</td>
<td style="text-align: right;">255.00000</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: right;">112</td>
<td style="text-align: right;">0.00000</td>
<td style="text-align: right;">42</td>
<td style="text-align: right;">42</td>
<td style="text-align: right;">42.0</td>
<td style="text-align: right;">42.00000</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: right;">117</td>
<td style="text-align: right;">0.00000</td>
<td style="text-align: right;">1500</td>
<td style="text-align: right;">1500</td>
<td style="text-align: right;">1500.0</td>
<td style="text-align: right;">1500.00000</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: right;">123</td>
<td style="text-align: right;">0.00000</td>
<td style="text-align: right;">1500</td>
<td style="text-align: right;">1500</td>
<td style="text-align: right;">1500.0</td>
<td style="text-align: right;">1500.00000</td>
<td style="text-align: right;">0.0000000</td>
</tr>
</tbody>
</table>

``` r
data %>%
  filter(grepl("^1[2|3|4]\\..+", src) & !grepl("^1[2|3|4]\\..+", dst) & port==124) %>%
  group_by(src) %>%
  summarise(traffic = sum(bytes)) %>%
  arrange(desc(traffic)) %>%
  head(1) %>%
  select(src) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">src</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">12.30.96.87</td>
</tr>
</tbody>
</table>

## ️Оценка результата

Был проведён анализ сетевого трафика при помощи библиотеки `arrow`

## ️Вывод

В результате выполнения работы были:

-   изучены возможности технологии [Apache
    Arrow](https://arrow.apache.org/) для обработки и анализ больших
    данных
-   Получены навыки применения `arrow` совместно с языком
    программирования `R`
-   получены навыки анализа `метаинфомации о сетевом трафике`
