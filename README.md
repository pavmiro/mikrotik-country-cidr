# 🌍 MikroTik Country Address Lists

[![Auto-updated](https://img.shields.io/badge/обновляется-ежедневно-brightgreen)]()
[![Countries](https://img.shields.io/badge/страны-RU-blue)](https://github.com/pavmiro/mikrotik-country-cidr/tree/main/RU)
[![Source](https://img.shields.io/badge/источник-RIPE%20NCC-orange)](https://ftp.ripe.net/ripe/stats/)
[![License: MIT](https://img.shields.io/badge/license-MIT-lightgrey)](./LICENSE)

Автоматически формируемые списки IP‑адресов по странам для **MikroTik RouterOS**.
Готовые файлы `.rsc` можно сразу импортировать и использовать в firewall, маршрутизации, QoS и других сценариях.

---

## ⚙️ Как это работает

1. **Источник данных** – официальная статистика RIPE NCC:  
   [ftp://ftp.ripe.net/ripe/stats/](ftp://ftp.ripe.net/ripe/stats/)

2. **Фильтрация** – выбираются только IPv4‑блоки, принадлежащие стране `RU`.

3. **Вычисление CIDR** – на основе диапазона и количества адресов вычисляется префикс:  
   `префикс = 32 - log₂(количество_адресов)` с округлением вниз (`int`).  

Используется следующая bash‑команда (запускается из crontab):

```bash
wget -c -O - ftp://ftp.ripe.net/ripe/stats/`date -d "yesterday" "+%Y"`/delegated-ripencc-`date -d "yesterday" "+%Y%m%d"`.bz2 | bzcat | grep -i "|RU|ipv4|" | awk -F '|' '{print $4 "/" (32 - log($5)/log(2))}' > delegated-ripencc-latest.txt
```
## 💾 Динамические адрес-листы и долговечность Flash

По умолчанию команды `/ip firewall address-list add ...` создают **статические** записи.  
Каждое такое добавление немедленно сохраняется на flash‑память роутера.  
Для больших списков (тысячи префиксов) это вызывает интенсивную запись и может сократить срок службы накопителя.

**Решение – динамические адрес-листы с параметром `timeout`.**  
Динамические записи хранятся только в оперативной памяти и не пишутся во flash, пока не истечёт таймаут.  
Максимально возможное значение `timeout` в RouterOS – **35 недель** (`35w`), что составляет около 8 месяцев.  
Этого более чем достаточно для ежедневной работы – запись можно обновлять самостоятельно или автоматически при перезагрузке роутера.

## 🚀 Пример использования

Скачивание и импорт списка для России (файл от 20 июня 2026):

```routeros
/tool fetch url=https://github.com/pavmiro/mikrotik-country-cidr/raw/refs/heads/main/RU/RIPE-RU-20-Jun-2026.rsc
/import RIPE-RU-20-Jun-2026.rsc
```
Автоматическое скачивание и импорт списка при загрузке роутера:

```routeros
/system scheduler
add name=schedule1 start-time=startup on-event="/delay 15\r\
    \n/tool/fetch url=https://github.com/pavmiro/mikrotik-country-cidr/raw/refs/heads/main/RIPE-RU-latest.rsc\r\
    \n/import RIPE-RU-latest.rsc"
```
