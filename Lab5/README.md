# Исследование информации о состоянии беспроводных сетей


## Цель

1.  Получить знания о методах исследования радиоэлектронной обстановки
2.  Составить представление о механизмах работы Wi-Fi сетей на канальном
    и сетевом уровне модели OSI
3.  Закрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    `tidyverse` языка R

## ️Исходные данные

1.  R 4.4.1
2.  RStudio 2024.04.2+764

## ️Общий план выполнения

Используя программный пакет `dplyr` языка программирования R провести
анализ журналов и ответить на вопросы.

## Содержание ЛР

### Шаг 1: Подготовка данных

1.  Импорт пакетов

``` r
library(dplyr)
```


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
library(readr)
```

    Warning: пакет 'readr' был собран под R версии 4.4.2

``` r
library(tidyr)
```

    Warning: пакет 'tidyr' был собран под R версии 4.4.2

1.  Получение данных (без пустых столбцов)

``` r
has_data <- function(x) { sum(!is.na(x)) > 0 }

data_1 <- read_csv("./data/data_1.csv") %>% select_if(has_data)
```

    Rows: 167 Columns: 15
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (6): BSSID, Privacy, Cipher, Authentication, LAN_IP, ESSID
    dbl  (6): channel, Speed, Power, N_beacons, N_IV, ID-length
    lgl  (1): Key
    dttm (2): First_time_seen, Last_time_seen

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
data_2 <- read_csv("./data/data_2.csv") %>% select_if(has_data)
```

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 12081 Columns: 7
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (3): Station_MAC, BSSID, Probed_ESSIDs
    dbl  (2): Power, N_packets
    dttm (2): First_time_seen, Last_time_seen

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

1.  Проверка структуры

``` r
glimpse(data_1)
```

    Rows: 167
    Columns: 14
    $ BSSID           <chr> "BE:F1:71:D5:17:8B", "6E:C7:EC:16:DA:1A", "9A:75:A8:B9…
    $ First_time_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ Last_time_seen  <dttm> 2023-07-28 11:50:50, 2023-07-28 11:55:12, 2023-07-28 …
    $ channel         <dbl> 1, 1, 1, 7, 6, 6, 11, 11, 11, 1, 6, 14, 11, 11, 6, 6, …
    $ Speed           <dbl> 195, 130, 360, 360, 130, 130, 195, 130, 130, 195, 180,…
    $ Privacy         <chr> "WPA2", "WPA2", "WPA2", "WPA2", "WPA2", "OPN", "WPA2",…
    $ Cipher          <chr> "CCMP", "CCMP", "CCMP", "CCMP", "CCMP", NA, "CCMP", "C…
    $ Authentication  <chr> "PSK", "PSK", "PSK", "PSK", "PSK", NA, "PSK", "PSK", "…
    $ Power           <dbl> -30, -30, -68, -37, -57, -63, -27, -38, -38, -66, -42,…
    $ N_beacons       <dbl> 846, 750, 694, 510, 647, 251, 1647, 1251, 704, 617, 13…
    $ N_IV            <dbl> 504, 116, 26, 21, 6, 3430, 80, 11, 0, 0, 86, 0, 0, 0, …
    $ LAN_IP          <chr> "0.  0.  0.  0", "0.  0.  0.  0", "0.  0.  0.  0", "0.…
    $ `ID-length`     <dbl> 12, 4, 2, 14, 25, 13, 12, 13, 24, 12, 10, 0, 24, 24, 1…
    $ ESSID           <chr> "C322U13 3965", "Cnet", "KC", "POCO X5 Pro 5G", NA, "M…

``` r
glimpse(data_2)
```

    Rows: 12,081
    Columns: 7
    $ Station_MAC     <chr> "CA:66:3B:8F:56:DD", "96:35:2D:3D:85:E6", "5C:3A:45:9E…
    $ First_time_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ Last_time_seen  <dttm> 2023-07-28 10:59:44, 2023-07-28 09:13:03, 2023-07-28 …
    $ Power           <dbl> -33, -65, -39, -61, -53, -43, -31, -71, -74, -65, -45,…
    $ N_packets       <dbl> 858, 4, 432, 958, 1, 344, 163, 3, 115, 437, 265, 77, 7…
    $ BSSID           <chr> "BE:F1:71:D5:17:8B", "(not associated)", "BE:F1:71:D6:…
    $ Probed_ESSIDs   <chr> "C322U13 3965", "IT2 Wireless", "C322U21 0566", "C322U…

### Шаг 2: Анализ

#### Точки доступа

1.  **Определить небезопасные точки доступа (без шифрования – OPN)**

``` r
data_1 %>% 
  filter(Privacy == "OPN") %>%
  select(BSSID) %>%
  distinct(BSSID) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:51</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:31</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:32</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:51</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:30</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:30</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:48:FF:D2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:41</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:40</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E0</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:42</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:40</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:41</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:47:D2</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:BC:15:7E:D5:DC</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B1</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:42</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:31</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:47:D1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:AB:0A:00:10:10</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:BD:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:0B:B2</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:33:12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:03:7A:1A:03:56</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:03:7F:12:34:56</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:3E:1A:5D:14:45</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:00:B1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:BD:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:EF</td>
</tr>
<tr class="odd">
<td style="text-align: left;">02:67:F1:B0:6C:98</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:CF:8B:87:B4:F9</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:53:7A:99:98:56</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:47:D0</td>
</tr>
</tbody>
</table>

1.  **Определить производителя для каждого обнаруженного устройства**

Для определения производителей обнаруженных устройств воспользуемся БД
производителей из состава Wireshark и онлайн сервисами OUI Lookup

    E8:28:C1:DC:BD:52 - Eltex Enterprise Ltd.
    00:26:99:F2:7A:EF - Cisco Systems, Inc
    E8:28:C1:DC:33:12 - Eltex Enterprise Ltd.
    E8:28:C1:DE:74:31 - Eltex Enterprise Ltd.
    E8:28:C1:DD:04:41 - Eltex Enterprise Ltd.
    02:BC:15:7E:D5:DC - ? (ESSID - MT_FREE)
    00:26:99:F2:7A:E0 - Cisco Systems, Inc
    E8:28:C1:DC:B2:50 - Eltex Enterprise Ltd.
    E8:28:C1:DC:B2:40 - Eltex Enterprise Ltd.
    E8:28:C1:DC:B2:52 - Eltex Enterprise Ltd.
    E8:28:C1:DC:C6:B0 - Eltex Enterprise Ltd.
    E8:28:C1:DC:C6:B1 - Eltex Enterprise Ltd.
    E8:28:C1:DC:C6:B2 - Eltex Enterprise Ltd.
    E8:28:C1:DD:04:51 - Eltex Enterprise Ltd.
    02:CF:8B:87:B4:F9 - ? (ESSID - MT_FREE)
    E8:28:C1:DC:C8:32 - Eltex Enterprise Ltd.
    00:AB:0A:00:10:10 - ?
    E8:28:C1:DE:74:32 - Eltex Enterprise Ltd.
    E8:28:C1:DD:04:42 - Eltex Enterprise Ltd.
    00:3E:1A:5D:14:45 - ? (ESSID - MT_FREE)
    00:25:00:FF:94:73 - Apple, Inc.
    E8:28:C1:DC:FF:F2 - Eltex Enterprise Ltd.
    E8:28:C1:DE:47:D1 - Eltex Enterprise Ltd.
    E8:28:C1:DC:BD:50 - Eltex Enterprise Ltd.
    E8:28:C1:DD:04:40 - Eltex Enterprise Ltd.
    00:53:7A:99:98:56 - ? (ESSID - MT_FREE)
    02:67:F1:B0:6C:98 - ? (ESSID - MT_FREE)
    E8:28:C1:DC:0B:B2 - Eltex Enterprise Ltd.
    E8:28:C1:DD:04:52 - Eltex Enterprise Ltd.
    E8:28:C1:DE:47:D2 - Eltex Enterprise Ltd.
    E8:28:C1:DE:47:D0 - Eltex Enterprise Ltd.
    E8:28:C1:DC:B2:42 - Eltex Enterprise Ltd.
    E8:28:C1:DC:C8:30 - Eltex Enterprise Ltd.
    00:03:7A:1A:03:56 - Taiyo Yuden Co., Ltd.
    E8:28:C1:DE:74:30 - Eltex Enterprise Ltd.
    E0:D9:E3:48:FF:D2 - Eltex Enterprise Ltd.
    E8:28:C1:DC:B2:51 - Eltex Enterprise Ltd.
    E0:D9:E3:49:00:B1 - Eltex Enterprise Ltd.
    E8:28:C1:DD:04:50 - Eltex Enterprise Ltd.
    E8:28:C1:DC:B2:41 - Eltex Enterprise Ltd.
    00:03:7F:12:34:56 - Atheros Communications, Inc.
    E8:28:C1:DC:C8:31 - Eltex Enterprise Ltd.

1.  **Выявить устройства, использующие последнюю версию протокола
    шифрования WPA3, и названия точек доступа, реализованных на этих
    устройствах**

``` r
data_1 %>% 
  filter(grepl('WPA3', Privacy)) %>%
  select(BSSID, ESSID) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
<th style="text-align: left;">ESSID</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">26:20:53:0C:98:E8</td>
<td style="text-align: left;">NA</td>
</tr>
<tr class="even">
<td style="text-align: left;">A2:FE:FF:B8:9B:C9</td>
<td style="text-align: left;">Christie’s</td>
</tr>
<tr class="odd">
<td style="text-align: left;">96:FF:FC:91:EF:64</td>
<td style="text-align: left;">NA</td>
</tr>
<tr class="even">
<td style="text-align: left;">CE:48:E7:86:4E:33</td>
<td style="text-align: left;">iPhone (Анастасия)</td>
</tr>
<tr class="odd">
<td style="text-align: left;">8E:1F:94:96:DA:FD</td>
<td style="text-align: left;">iPhone (Анастасия)</td>
</tr>
<tr class="even">
<td style="text-align: left;">BE:FD:EF:18:92:44</td>
<td style="text-align: left;">Димасик</td>
</tr>
<tr class="odd">
<td style="text-align: left;">3A:DA:00:F9:0C:02</td>
<td style="text-align: left;">iPhone XS Max 🦊🐱🦊</td>
</tr>
<tr class="even">
<td style="text-align: left;">76:C5:A0:70:08:96</td>
<td style="text-align: left;">NA</td>
</tr>
</tbody>
</table>

1.  **Отсортировать точки доступа по интервалу времени, в течение
    которого они находились на связи, по убыванию**

``` r
data_1 %>% 
  mutate(duration = difftime(Last_time_seen, First_time_seen, units="mins")) %>%
  select(BSSID, duration) %>%
  arrange(desc(duration)) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
<th style="text-align: left;">duration</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: left;">163.2500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
<td style="text-align: left;">162.9333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: left;">162.5833333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
<td style="text-align: left;">162.4333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">6E:C7:EC:16:DA:1A</td>
<td style="text-align: left;">162.1500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
<td style="text-align: left;">162.1000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:51</td>
<td style="text-align: left;">162.0833333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">48:5B:39:F9:7A:48</td>
<td style="text-align: left;">162.0833333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
<td style="text-align: left;">162.0666667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">8E:55:4A:85:5B:01</td>
<td style="text-align: left;">162.0500000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: left;">161.8333333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
<td style="text-align: left;">161.7833333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">1E:93:E3:1B:3C:F4</td>
<td style="text-align: left;">160.5500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">9A:75:A8:B9:04:1E</td>
<td style="text-align: left;">160.4666667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">0C:80:63:A9:6E:EE</td>
<td style="text-align: left;">160.4666667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:81:F2</td>
<td style="text-align: left;">159.9166667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9E:A3:A9:DB:7E:01</td>
<td style="text-align: left;">159.2500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:32</td>
<td style="text-align: left;">159.2500000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">1C:7E:E5:8E:B7:DE</td>
<td style="text-align: left;">158.7333333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E1</td>
<td style="text-align: left;">158.2000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">BE:F1:71:D5:17:8B</td>
<td style="text-align: left;">157.7833333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">BE:F1:71:D6:10:D7</td>
<td style="text-align: left;">157.6833333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9E:A3:A9:D6:28:3C</td>
<td style="text-align: left;">157.5166667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:40</td>
<td style="text-align: left;">156.6666667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:41</td>
<td style="text-align: left;">156.6666667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:81:F1</td>
<td style="text-align: left;">155.8000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:23:EB:E3:81:FE</td>
<td style="text-align: left;">155.0833333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:81:FD</td>
<td style="text-align: left;">155.0833333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9E:A3:A9:BF:12:C0</td>
<td style="text-align: left;">154.5000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:40</td>
<td style="text-align: left;">153.5333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">AA:F4:3F:EE:49:0B</td>
<td style="text-align: left;">150.7500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:47:D2</td>
<td style="text-align: left;">150.6833333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:50</td>
<td style="text-align: left;">149.8166667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">14:EB:B6:6A:76:37</td>
<td style="text-align: left;">148.5833333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">56:99:98:EE:5A:4E</td>
<td style="text-align: left;">146.8500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:42</td>
<td style="text-align: left;">144.8833333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:90:A1</td>
<td style="text-align: left;">144.3500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">0A:C5:E1:DB:17:7B</td>
<td style="text-align: left;">143.4666667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:30</td>
<td style="text-align: left;">140.7500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B1</td>
<td style="text-align: left;">139.8333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:42</td>
<td style="text-align: left;">138.6333333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:41</td>
<td style="text-align: left;">138.4500000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">12:51:07:FF:29:D6</td>
<td style="text-align: left;">124.7166667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">CE:B3:FF:84:45:FC</td>
<td style="text-align: left;">121.1833333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:31</td>
<td style="text-align: left;">119.9833333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
<td style="text-align: left;">113.6500000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">4A:EC:1E:DB:BF:95</td>
<td style="text-align: left;">110.9666667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E0</td>
<td style="text-align: left;">103.6333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:51</td>
<td style="text-align: left;">94.0500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:48:FF:D2</td>
<td style="text-align: left;">93.7333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:AB:0A:00:10:10</td>
<td style="text-align: left;">89.2666667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
<td style="text-align: left;">86.5000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">10:50:72:00:11:08</td>
<td style="text-align: left;">83.2833333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">EA:D8:D1:77:C8:08</td>
<td style="text-align: left;">83.2500000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">D2:6D:52:61:51:5D</td>
<td style="text-align: left;">77.2666667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:52</td>
<td style="text-align: left;">76.9000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">7E:3A:10:A7:59:4E</td>
<td style="text-align: left;">76.8500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">BE:F1:71:D5:0E:53</td>
<td style="text-align: left;">76.3000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">A6:02:B9:73:83:18</td>
<td style="text-align: left;">76.2833333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">9A:9F:06:44:24:5B</td>
<td style="text-align: left;">76.2000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:31</td>
<td style="text-align: left;">73.8833333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">92:F5:7B:43:0B:69</td>
<td style="text-align: left;">73.2000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:3C:92</td>
<td style="text-align: left;">72.1833333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">38:1A:52:0D:84:D7</td>
<td style="text-align: left;">71.9833333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:90:5D</td>
<td style="text-align: left;">70.9166667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">A2:64:E8:97:58:EE</td>
<td style="text-align: left;">70.8666667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">A6:02:B9:73:81:47</td>
<td style="text-align: left;">70.4000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">56:C5:2B:9F:84:90</td>
<td style="text-align: left;">69.5500000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">A6:02:B9:73:2F:76</td>
<td style="text-align: left;">69.0666667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">38:1A:52:0D:97:60</td>
<td style="text-align: left;">68.1000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">0A:24:D8:D9:24:70</td>
<td style="text-align: left;">67.8500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B0</td>
<td style="text-align: left;">64.6500000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">8A:A3:03:73:52:08</td>
<td style="text-align: left;">57.5166667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">5E:C7:C0:E4:D7:D4</td>
<td style="text-align: left;">54.4166667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:54:72</td>
<td style="text-align: left;">51.2333333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">4A:86:77:04:B7:28</td>
<td style="text-align: left;">50.1333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">B6:C4:55:B5:53:24</td>
<td style="text-align: left;">49.7833333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:BD:50</td>
<td style="text-align: left;">45.7166667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">76:70:AF:A4:D2:AF</td>
<td style="text-align: left;">45.5500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">86:DF:BF:E4:2F:23</td>
<td style="text-align: left;">44.8000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:8F:EC</td>
<td style="text-align: left;">43.9166667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">EA:7B:9B:D8:56:34</td>
<td style="text-align: left;">37.3500000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:85:1D</td>
<td style="text-align: left;">34.7000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:CB:AA:62:71</td>
<td style="text-align: left;">32.8166667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">96:FF:FC:91:EF:64</td>
<td style="text-align: left;">32.1333333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:33:12</td>
<td style="text-align: left;">22.9833333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:F0:90</td>
<td style="text-align: left;">21.8666667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">3A:70:96:C6:30:2C</td>
<td style="text-align: left;">21.6666667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">36:46:53:81:12:A0</td>
<td style="text-align: left;">20.8000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">CE:C3:F7:A4:7E:B3</td>
<td style="text-align: left;">20.4000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">26:20:53:0C:98:E8</td>
<td style="text-align: left;">17.4166667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">92:12:38:E5:7E:1E</td>
<td style="text-align: left;">14.4666667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:33:10</td>
<td style="text-align: left;">14.1000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DB:F5:F0</td>
<td style="text-align: left;">14.0333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:0B:B0</td>
<td style="text-align: left;">13.8666667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DB:F5:F2</td>
<td style="text-align: left;">13.0333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">02:67:F1:B0:6C:98</td>
<td style="text-align: left;">10.8500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:30</td>
<td style="text-align: left;">8.4666667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">1E:C2:8E:D8:30:91</td>
<td style="text-align: left;">8.3000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">8E:1F:94:96:DA:FD</td>
<td style="text-align: left;">6.9166667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:49:04:50</td>
<td style="text-align: left;">6.6833333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">CE:48:E7:86:4E:33</td>
<td style="text-align: left;">4.9166667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:8F</td>
<td style="text-align: left;">4.8000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">2A:E8:A2:02:01:73</td>
<td style="text-align: left;">3.6666667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2E:FE:13:D0:96:51</td>
<td style="text-align: left;">0.9666667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">9C:A5:13:28:D5:89</td>
<td style="text-align: left;">0.7166667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">22:C9:7F:A9:BA:9C</td>
<td style="text-align: left;">0.6833333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:54:B0</td>
<td style="text-align: left;">0.6000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">D2:25:91:F6:6C:D8</td>
<td style="text-align: left;">0.2166667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">3A:DA:00:F9:0C:02</td>
<td style="text-align: left;">0.1500000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DB:FC:F2</td>
<td style="text-align: left;">0.1500000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">DC:09:4C:32:34:9B</td>
<td style="text-align: left;">0.1333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">F2:30:AB:E9:03:ED</td>
<td style="text-align: left;">0.1166667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:40</td>
<td style="text-align: left;">0.1166667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:03:7A:1A:03:56</td>
<td style="text-align: left;">0.1000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">B2:CF:C0:00:4A:60</td>
<td style="text-align: left;">0.0833333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">BE:FD:EF:18:92:44</td>
<td style="text-align: left;">0.0666667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:BC:15:7E:D5:DC</td>
<td style="text-align: left;">0.0333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:23:EB:E3:49:31</td>
<td style="text-align: left;">0.0333333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:3E:1A:5D:14:45</td>
<td style="text-align: left;">0.0333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">76:C5:A0:70:08:96</td>
<td style="text-align: left;">0.0333333 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">82:CD:7D:04:17:3B</td>
<td style="text-align: left;">0.0333333 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:49:00:B0</td>
<td style="text-align: left;">0.0166667 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:54:B2</td>
<td style="text-align: left;">0.0166667 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">C6:BC:37:7A:67:0D</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">12:48:F9:CF:58:8E</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">76:E4:ED:B0:5C:9A</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:48:FF:D0</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E2:37:BF:8F:6A:7B</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">C2:B5:D7:7F:07:A8</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">8A:4E:75:44:5A:F6</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:03:7A:1A:18:56</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:47:D1</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">A2:FE:FF:B8:9B:C9</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:09:9A:12:55:04</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:3A:B0</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:0B:B2</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:3C:80</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:23:EB:E3:44:31</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">A6:F7:05:31:E8:EE</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">BA:2A:7A:DD:38:3E</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">12:54:1A:C6:FF:71</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">76:5E:F3:F9:A5:1C</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:03:7F:12:34:56</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:03:30</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">B2:1B:0C:67:0A:BD</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:49:00:B1</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:BD:52</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:72:D0</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:41</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F1:1A:E1</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:44:32</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:CB:AA:62:72</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:48:B4:D2</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">AE:3E:7F:C8:BC:8E</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:B3:45:5A:05:93</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:00:00:00:00:00</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">6A:B0:1A:C2:DF:49</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:3C:90</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">30:B4:B8:11:C0:90</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:EF</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:CF:8B:87:B4:F9</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:03:32</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:53:7A:99:98:56</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:03:7F:10:17:56</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:0D:97:6B:93:DF</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:47:D0</td>
<td style="text-align: left;">0.0000000 mins</td>
</tr>
</tbody>
</table>

1.  **Обнаружить топ-10 самых быстрых точек доступа**

``` r
data_1 %>% 
  arrange(desc(Speed)) %>%
  select(BSSID, Speed) %>% 
  head(10) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
<th style="text-align: right;">Speed</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">26:20:53:0C:98:E8</td>
<td style="text-align: right;">866</td>
</tr>
<tr class="even">
<td style="text-align: left;">96:FF:FC:91:EF:64</td>
<td style="text-align: right;">866</td>
</tr>
<tr class="odd">
<td style="text-align: left;">CE:48:E7:86:4E:33</td>
<td style="text-align: right;">866</td>
</tr>
<tr class="even">
<td style="text-align: left;">8E:1F:94:96:DA:FD</td>
<td style="text-align: right;">866</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9A:75:A8:B9:04:1E</td>
<td style="text-align: right;">360</td>
</tr>
<tr class="even">
<td style="text-align: left;">4A:EC:1E:DB:BF:95</td>
<td style="text-align: right;">360</td>
</tr>
<tr class="odd">
<td style="text-align: left;">56:C5:2B:9F:84:90</td>
<td style="text-align: right;">360</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:41</td>
<td style="text-align: right;">360</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:40</td>
<td style="text-align: right;">360</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:42</td>
<td style="text-align: right;">360</td>
</tr>
</tbody>
</table>

1.  **Отсортировать точки доступа по частоте отправки запросов (beacons)
    в единицу времени по их убыванию**

``` r
data_1 %>% mutate(frequency = N_beacons / as.numeric(difftime(Last_time_seen, First_time_seen, units="mins"))) %>%
  select(BSSID, frequency) %>%
  arrange(desc(frequency)) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
<th style="text-align: right;">frequency</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">76:E4:ED:B0:5C:9A</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="even">
<td style="text-align: left;">C2:B5:D7:7F:07:A8</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:47:D1</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="even">
<td style="text-align: left;">A2:FE:FF:B8:9B:C9</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="odd">
<td style="text-align: left;">BA:2A:7A:DD:38:3E</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="even">
<td style="text-align: left;">76:5E:F3:F9:A5:1C</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:03:7F:12:34:56</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:00:B1</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:BD:52</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:CB:AA:62:72</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="odd">
<td style="text-align: left;">6A:B0:1A:C2:DF:49</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:CF:8B:87:B4:F9</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:47:D0</td>
<td style="text-align: right;">Inf</td>
</tr>
<tr class="even">
<td style="text-align: left;">F2:30:AB:E9:03:ED</td>
<td style="text-align: right;">51.4285714</td>
</tr>
<tr class="odd">
<td style="text-align: left;">B2:CF:C0:00:4A:60</td>
<td style="text-align: right;">48.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">3A:DA:00:F9:0C:02</td>
<td style="text-align: right;">33.3333333</td>
</tr>
<tr class="odd">
<td style="text-align: left;">02:BC:15:7E:D5:DC</td>
<td style="text-align: right;">30.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:3E:1A:5D:14:45</td>
<td style="text-align: right;">30.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">76:C5:A0:70:08:96</td>
<td style="text-align: right;">30.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">D2:25:91:F6:6C:D8</td>
<td style="text-align: right;">23.0769231</td>
</tr>
<tr class="odd">
<td style="text-align: left;">BE:F1:71:D6:10:D7</td>
<td style="text-align: right;">10.4449847</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:03:7A:1A:03:56</td>
<td style="text-align: right;">10.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:84:D7</td>
<td style="text-align: right;">9.7800417</td>
</tr>
<tr class="even">
<td style="text-align: left;">0A:C5:E1:DB:17:7B</td>
<td style="text-align: right;">8.7197955</td>
</tr>
<tr class="odd">
<td style="text-align: left;">1E:93:E3:1B:3C:F4</td>
<td style="text-align: right;">8.6577390</td>
</tr>
<tr class="even">
<td style="text-align: left;">D2:6D:52:61:51:5D</td>
<td style="text-align: right;">8.3735979</td>
</tr>
<tr class="odd">
<td style="text-align: left;">BE:F1:71:D5:0E:53</td>
<td style="text-align: right;">8.0865007</td>
</tr>
<tr class="even">
<td style="text-align: left;">4A:86:77:04:B7:28</td>
<td style="text-align: right;">7.2007979</td>
</tr>
<tr class="odd">
<td style="text-align: left;">3A:70:96:C6:30:2C</td>
<td style="text-align: right;">6.6923077</td>
</tr>
<tr class="even">
<td style="text-align: left;">76:70:AF:A4:D2:AF</td>
<td style="text-align: right;">5.5543359</td>
</tr>
<tr class="odd">
<td style="text-align: left;">BE:F1:71:D5:17:8B</td>
<td style="text-align: right;">5.3617830</td>
</tr>
<tr class="even">
<td style="text-align: left;">AA:F4:3F:EE:49:0B</td>
<td style="text-align: right;">4.8955224</td>
</tr>
<tr class="odd">
<td style="text-align: left;">6E:C7:EC:16:DA:1A</td>
<td style="text-align: right;">4.6253469</td>
</tr>
<tr class="even">
<td style="text-align: left;">4A:EC:1E:DB:BF:95</td>
<td style="text-align: right;">4.5959748</td>
</tr>
<tr class="odd">
<td style="text-align: left;">56:C5:2B:9F:84:90</td>
<td style="text-align: right;">4.5578720</td>
</tr>
<tr class="even">
<td style="text-align: left;">9A:75:A8:B9:04:1E</td>
<td style="text-align: right;">4.3248857</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9C:A5:13:28:D5:89</td>
<td style="text-align: right;">4.1860465</td>
</tr>
<tr class="even">
<td style="text-align: left;">36:46:53:81:12:A0</td>
<td style="text-align: right;">3.9423077</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:85:1D</td>
<td style="text-align: right;">3.7463977</td>
</tr>
<tr class="even">
<td style="text-align: left;">38:1A:52:0D:8F:EC</td>
<td style="text-align: right;">2.4364326</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2E:FE:13:D0:96:51</td>
<td style="text-align: right;">2.0689655</td>
</tr>
<tr class="even">
<td style="text-align: left;">CE:48:E7:86:4E:33</td>
<td style="text-align: right;">1.8305085</td>
</tr>
<tr class="odd">
<td style="text-align: left;">8E:1F:94:96:DA:FD</td>
<td style="text-align: right;">1.7349398</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:51</td>
<td style="text-align: right;">1.7213368</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
<td style="text-align: right;">1.6039482</td>
</tr>
<tr class="even">
<td style="text-align: left;">5E:C7:C0:E4:D7:D4</td>
<td style="text-align: right;">1.5620214</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
<td style="text-align: right;">1.5438237</td>
</tr>
<tr class="even">
<td style="text-align: left;">8E:55:4A:85:5B:01</td>
<td style="text-align: right;">1.5303919</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:90:5D</td>
<td style="text-align: right;">1.2690952</td>
</tr>
<tr class="even">
<td style="text-align: left;">1C:7E:E5:8E:B7:DE</td>
<td style="text-align: right;">0.8945821</td>
</tr>
<tr class="odd">
<td style="text-align: left;">38:1A:52:0D:90:A1</td>
<td style="text-align: right;">0.7758919</td>
</tr>
<tr class="even">
<td style="text-align: left;">A2:64:E8:97:58:EE</td>
<td style="text-align: right;">0.7337723</td>
</tr>
<tr class="odd">
<td style="text-align: left;">1E:C2:8E:D8:30:91</td>
<td style="text-align: right;">0.7228916</td>
</tr>
<tr class="even">
<td style="text-align: left;">48:5B:39:F9:7A:48</td>
<td style="text-align: right;">0.6724936</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
<td style="text-align: right;">0.5192129</td>
</tr>
<tr class="even">
<td style="text-align: left;">38:1A:52:0D:97:60</td>
<td style="text-align: right;">0.4111601</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E1</td>
<td style="text-align: right;">0.4108723</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:80</td>
<td style="text-align: right;">0.3769310</td>
</tr>
<tr class="odd">
<td style="text-align: left;">A6:02:B9:73:2F:76</td>
<td style="text-align: right;">0.3764479</td>
</tr>
<tr class="even">
<td style="text-align: left;">9E:A3:A9:D6:28:3C</td>
<td style="text-align: right;">0.3237753</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:23:EB:E3:81:FE</td>
<td style="text-align: right;">0.3030629</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:81:FD</td>
<td style="text-align: right;">0.2966147</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9A:9F:06:44:24:5B</td>
<td style="text-align: right;">0.2887139</td>
</tr>
<tr class="even">
<td style="text-align: left;">96:FF:FC:91:EF:64</td>
<td style="text-align: right;">0.2800830</td>
</tr>
<tr class="odd">
<td style="text-align: left;">A6:02:B9:73:81:47</td>
<td style="text-align: right;">0.2698864</td>
</tr>
<tr class="even">
<td style="text-align: left;">0C:80:63:A9:6E:EE</td>
<td style="text-align: right;">0.2617366</td>
</tr>
<tr class="odd">
<td style="text-align: left;">12:51:07:FF:29:D6</td>
<td style="text-align: right;">0.2565816</td>
</tr>
<tr class="even">
<td style="text-align: left;">9E:A3:A9:DB:7E:01</td>
<td style="text-align: right;">0.2511774</td>
</tr>
<tr class="odd">
<td style="text-align: left;">92:F5:7B:43:0B:69</td>
<td style="text-align: right;">0.2459016</td>
</tr>
<tr class="even">
<td style="text-align: left;">86:DF:BF:E4:2F:23</td>
<td style="text-align: right;">0.2455357</td>
</tr>
<tr class="odd">
<td style="text-align: left;">A6:02:B9:73:83:18</td>
<td style="text-align: right;">0.2228534</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:40</td>
<td style="text-align: right;">0.1914894</td>
</tr>
<tr class="odd">
<td style="text-align: left;">26:20:53:0C:98:E8</td>
<td style="text-align: right;">0.1722488</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:42</td>
<td style="text-align: right;">0.1659053</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:41</td>
<td style="text-align: right;">0.1595745</td>
</tr>
<tr class="even">
<td style="text-align: left;">B6:C4:55:B5:53:24</td>
<td style="text-align: right;">0.1406093</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:50</td>
<td style="text-align: right;">0.1334965</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:81:F1</td>
<td style="text-align: right;">0.1219512</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:BD:50</td>
<td style="text-align: right;">0.1093693</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:51</td>
<td style="text-align: right;">0.0956938</td>
</tr>
<tr class="odd">
<td style="text-align: left;">02:67:F1:B0:6C:98</td>
<td style="text-align: right;">0.0921659</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:32</td>
<td style="text-align: right;">0.0753532</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:31</td>
<td style="text-align: right;">0.0666759</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B0</td>
<td style="text-align: right;">0.0618716</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:CB:AA:62:71</td>
<td style="text-align: right;">0.0609446</td>
</tr>
<tr class="even">
<td style="text-align: left;">9E:A3:A9:BF:12:C0</td>
<td style="text-align: right;">0.0582524</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:30</td>
<td style="text-align: right;">0.0497336</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:81:F2</td>
<td style="text-align: right;">0.0437728</td>
</tr>
<tr class="odd">
<td style="text-align: left;">7E:3A:10:A7:59:4E</td>
<td style="text-align: right;">0.0390371</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:41</td>
<td style="text-align: right;">0.0361141</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B1</td>
<td style="text-align: right;">0.0357569</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:42</td>
<td style="text-align: right;">0.0345105</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:40</td>
<td style="text-align: right;">0.0325662</td>
</tr>
<tr class="even">
<td style="text-align: left;">0A:24:D8:D9:24:70</td>
<td style="text-align: right;">0.0294768</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:31</td>
<td style="text-align: right;">0.0270697</td>
</tr>
<tr class="even">
<td style="text-align: left;">EA:7B:9B:D8:56:34</td>
<td style="text-align: right;">0.0267738</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
<td style="text-align: right;">0.0245499</td>
</tr>
<tr class="even">
<td style="text-align: left;">10:50:72:00:11:08</td>
<td style="text-align: right;">0.0240144</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:47:D2</td>
<td style="text-align: right;">0.0199093</td>
</tr>
<tr class="even">
<td style="text-align: left;">EA:D8:D1:77:C8:08</td>
<td style="text-align: right;">0.0120120</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
<td style="text-align: right;">0.0115607</td>
</tr>
<tr class="even">
<td style="text-align: left;">56:99:98:EE:5A:4E</td>
<td style="text-align: right;">0.0068097</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:30</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:48:FF:D2</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E0</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2A:E8:A2:02:01:73</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:3C:92</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">14:EB:B6:6A:76:37</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">CE:B3:FF:84:45:FC</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:54:72</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:AB:0A:00:10:10</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DB:F5:F2</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">BE:FD:EF:18:92:44</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:49:31</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">CE:C3:F7:A4:7E:B3</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:33:12</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DB:FC:F2</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:8F</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">DC:09:4C:32:34:9B</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:F0:90</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:49:04:52</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:50</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:49:04:40</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:54:B0</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:49:00:B0</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:33:10</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DB:F5:F0</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">8A:A3:03:73:52:08</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">22:C9:7F:A9:BA:9C</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">92:12:38:E5:7E:1E</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:0B:B0</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">82:CD:7D:04:17:3B</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:54:B2</td>
<td style="text-align: right;">0.0000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">C6:BC:37:7A:67:0D</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">12:48:F9:CF:58:8E</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:48:FF:D0</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E2:37:BF:8F:6A:7B</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">8A:4E:75:44:5A:F6</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:03:7A:1A:18:56</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:09:9A:12:55:04</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:3A:B0</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:0B:B2</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:3C:80</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:44:31</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">A6:F7:05:31:E8:EE</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">12:54:1A:C6:FF:71</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:03:30</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">B2:1B:0C:67:0A:BD</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:72:D0</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:41</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F1:1A:E1</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:44:32</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:48:B4:D2</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">AE:3E:7F:C8:BC:8E</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">02:B3:45:5A:05:93</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:00:00:00:00:00</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:3C:90</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">30:B4:B8:11:C0:90</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:EF</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:03:32</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:53:7A:99:98:56</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:03:7F:10:17:56</td>
<td style="text-align: right;">NaN</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:0D:97:6B:93:DF</td>
<td style="text-align: right;">NaN</td>
</tr>
</tbody>
</table>

#### Данные клиентов

1.  **Определить производителя для каждого обнаруженного устройства**

``` r
data_2 %>%
  filter(BSSID != "(not associated)") %>%
  mutate(maker = substr(BSSID, 1, 8)) %>%
  select(maker) %>%
  unique() %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">maker</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">BE:F1:71</td>
</tr>
<tr class="even">
<td style="text-align: left;">1E:93:E3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99</td>
</tr>
<tr class="even">
<td style="text-align: left;">0C:80:63</td>
</tr>
<tr class="odd">
<td style="text-align: left;">0A:C5:E1</td>
</tr>
<tr class="even">
<td style="text-align: left;">9A:75:A8</td>
</tr>
<tr class="odd">
<td style="text-align: left;">8A:A3:03</td>
</tr>
<tr class="even">
<td style="text-align: left;">4A:EC:1E</td>
</tr>
<tr class="odd">
<td style="text-align: left;">08:3A:2F</td>
</tr>
<tr class="even">
<td style="text-align: left;">6E:C7:EC</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2A:E8:A2</td>
</tr>
<tr class="even">
<td style="text-align: left;">56:C5:2B</td>
</tr>
<tr class="odd">
<td style="text-align: left;">9A:9F:06</td>
</tr>
<tr class="even">
<td style="text-align: left;">12:48:F9</td>
</tr>
<tr class="odd">
<td style="text-align: left;">AA:F4:3F</td>
</tr>
<tr class="even">
<td style="text-align: left;">3A:70:96</td>
</tr>
<tr class="odd">
<td style="text-align: left;">8E:55:4A</td>
</tr>
<tr class="even">
<td style="text-align: left;">5E:C7:C0</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E2:37:BF</td>
</tr>
<tr class="even">
<td style="text-align: left;">96:FF:FC</td>
</tr>
<tr class="odd">
<td style="text-align: left;">CE:B3:FF</td>
</tr>
<tr class="even">
<td style="text-align: left;">76:70:AF</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:AB:0A</td>
</tr>
<tr class="even">
<td style="text-align: left;">8E:1F:94</td>
</tr>
<tr class="odd">
<td style="text-align: left;">EA:7B:9B</td>
</tr>
<tr class="even">
<td style="text-align: left;">BE:FD:EF</td>
</tr>
<tr class="odd">
<td style="text-align: left;">7E:3A:10</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3</td>
</tr>
<tr class="even">
<td style="text-align: left;">3A:DA:00</td>
</tr>
<tr class="odd">
<td style="text-align: left;">92:F5:7B</td>
</tr>
<tr class="even">
<td style="text-align: left;">DC:09:4C</td>
</tr>
<tr class="odd">
<td style="text-align: left;">22:C9:7F</td>
</tr>
<tr class="even">
<td style="text-align: left;">92:12:38</td>
</tr>
<tr class="odd">
<td style="text-align: left;">B2:1B:0C</td>
</tr>
<tr class="even">
<td style="text-align: left;">1E:C2:8E</td>
</tr>
<tr class="odd">
<td style="text-align: left;">A2:64:E8</td>
</tr>
<tr class="even">
<td style="text-align: left;">A6:02:B9</td>
</tr>
<tr class="odd">
<td style="text-align: left;">AE:3E:7F</td>
</tr>
<tr class="even">
<td style="text-align: left;">B6:C4:55</td>
</tr>
<tr class="odd">
<td style="text-align: left;">86:DF:BF</td>
</tr>
<tr class="even">
<td style="text-align: left;">02:67:F1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">36:46:53</td>
</tr>
<tr class="even">
<td style="text-align: left;">82:CD:7D</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:03:7F</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:0D:97</td>
</tr>
</tbody>
</table>

    00:03:7F Atheros Communications, Inc.
    00:0D:97 Hitachi Energy USA Inc.
    00:23:EB Cisco Systems, Inc
    00:25:00 Apple, Inc.
    00:26:99 Cisco Systems, Inc
    08:3A:2F Guangzhou Juan Intelligent Tech Joint Stock Co.,Ltd
    0C:80:63 Tp-Link Technologies Co.,Ltd.
    DC:09:4C Huawei Technologies Co.,Ltd
    E0:D9:E3 Eltex Enterprise Ltd.
    E8:28:C1 Eltex Enterprise Ltd.

    Остальные - неизвестны

1.  **Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес**

Если второй символ адреса это 2, 6, A или E, то это рандомизированный
адрес.

``` r
data_2 %>%
  select(BSSID) %>%
  filter(BSSID != "(not associated)" & !grepl("^.[2|6|A|E].+", BSSID)) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">BSSID</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="even">
<td style="text-align: left;">0C:80:63:A9:6E:EE</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
</tr>
<tr class="odd">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="even">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:32</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:50</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:3C:92</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">0C:80:63:A9:6E:EE</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="even">
<td style="text-align: left;">08:3A:2F:56:35:FE</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:AB:0A:00:10:10</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C8:30</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DB:F5:F2</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:FF:F2</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:40</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E1</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:23:EB:E3:49:31</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:40</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:40</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:B2:41</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:40</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:33:12</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">DC:09:4C:32:34:9B</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:F0:90</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E0:D9:E3:49:04:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:F0:90</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:41</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DD:04:52</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DE:74:32</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:33:10</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E0:D9:E3:49:04:41</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C6:B2</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DD:04:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:F0:90</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:C8:32</td>
</tr>
<tr class="even">
<td style="text-align: left;">E8:28:C1:DC:B2:50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:BA:75:80</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:0B:B0</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">E8:28:C1:DC:54:B2</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:03:7F:10:17:56</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:0D:97:6B:93:DF</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:26:99:F2:7A:E2</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
<tr class="even">
<td style="text-align: left;">00:25:00:FF:94:73</td>
</tr>
</tbody>
</table>

1.  **Кластеризовать запросы от устройств к точкам доступа по их именам.
    Определить время появления устройства в зоне радиовидимости и время
    выхода его из нее**

``` r
data_2 %>%
  separate_rows(Probed_ESSIDs, sep=",") %>%
  group_by(Probed_ESSIDs) %>%
  summarise(first_time = min(First_time_seen, na.rm = TRUE),
            last_time = max(Last_time_seen, na.rm = TRUE))
```

    # A tibble: 152 × 3
       Probed_ESSIDs                  first_time          last_time          
       <chr>                          <dttm>              <dttm>             
     1 " podval"                      2023-07-28 10:44:38 2023-07-28 10:44:38
     2 "-D-13-"                       2023-07-28 09:14:42 2023-07-28 10:26:42
     3 "1"                            2023-07-28 10:36:12 2023-07-28 11:56:13
     4 "107"                          2023-07-28 10:29:43 2023-07-28 10:29:43
     5 "531"                          2023-07-28 10:57:04 2023-07-28 10:57:04
     6 "AAAAADVpTWoADwFlRedmi 4X"     2023-07-28 09:34:20 2023-07-28 11:44:40
     7 "AAAAAOB/CC0ADwGkRedmi 3S"     2023-07-28 09:34:20 2023-07-28 11:44:40
     8 "AKADO-D967"                   2023-07-28 10:31:55 2023-07-28 10:31:55
     9 "AQAAAB6zaIoATwEURedmi Note 5" 2023-07-28 10:25:11 2023-07-28 11:51:48
    10 "ASUS"                         2023-07-28 10:31:13 2023-07-28 10:31:13
    # ℹ 142 more rows

1.  **Оценить стабильность уровня сигнала внури кластера во времени.
    Выявить наиболее стабильный кластер.**

``` r
data_2 %>%
  separate_rows(Probed_ESSIDs, sep=",") %>%
  group_by(Probed_ESSIDs) %>%
  summarise(sd = sd(Power, na.rm = TRUE),
            mean = mean(Power, na.rm = TRUE)) %>%
  arrange(sd) %>%
  head(1) %>%
  knitr::kable()
```

<table>
<thead>
<tr class="header">
<th style="text-align: left;">Probed_ESSIDs</th>
<th style="text-align: right;">sd</th>
<th style="text-align: right;">mean</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">CPPK_FREE</td>
<td style="text-align: right;">0</td>
<td style="text-align: right;">-67</td>
</tr>
</tbody>
</table>

## ️Оценка результата

Был проведён анализ журналов с помощью программного пакета `dplyr`

## ️Вывод

В результате выполнения работы были:

-   получены знания о методах исследования радиоэлектронной обстановки
-   закреплены практические навыки использования языка программирования
    R для обработки данных
-   закреплены знания основных функций обработки данных экосистемы
    `tidyverse` языка R
