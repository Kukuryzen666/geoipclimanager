# GeoIP nftables CLI Manager 🛡️

<div align="center">

![Python](https://img.shields.io/badge/Python-3.6%2B-blue?logo=python&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Linux-orange?logo=linux&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
![Dependencies](https://img.shields.io/badge/Dependencies-Zero-brightgreen)
![nftables](https://img.shields.io/badge/Firewall-nftables-red)

**Production-ready утилита для управления доступом к Linux-серверам на базе nftables.**  
Ограничивает доступ (SSH, HTTP/S) только для абонентов конкретных провайдеров по BGP ASN или точечных IP-адресов.

Разработано в парадигме **Zero Trust** и **Zero Dependencies** — работает на чистом Python 3 без сторонних библиотек.

</div>

---

## ✨ Особенности

| Фича | Описание |
|------|----------|
| 🚀 **Zero Dependencies** | Использует только стандартную библиотеку Python (`urllib`, `ipaddress`, `json`) — никакого `pip install` |
| 🧠 **BGP-агрегация** | Скачивает префиксы из официального API [RIPEstat](https://stat.ripe.net/) и схлопывает пересекающиеся CIDR для $O(1)$ производительности ядра |
| 🛠 **Interactive TUI** | Удобное консольное меню для управления ASN, подсетями и правилами проброса портов |
| ⚙️ **Self-Healing** | Автоматически патчит и восстанавливает `/etc/nftables.conf` при повреждениях (Zero-Touch Provisioning) |
| 🕒 **Cron-менеджер** | Встроенная настройка автообновления IP-баз раз в день / неделю / месяц |
| 🎯 **Гранулярный контроль** | Точечные правила `IP → Протокол → Порт`, изолированные от общих GeoIP-правил |

---

## 🖥 Поддерживаемые ОС

- Debian 11 / 12 / 13
- Ubuntu 20.04 / 22.04 / 24.04
- Любой дистрибутив с поддержкой `nftables`

---

## 🚀 Установка

```bash
# 1. Установите nftables (если ещё не установлен)
sudo apt update && sudo apt install nftables python3 -y

# 2. Скачайте скрипт в исполняемую директорию
sudo curl -o /usr/local/bin/update_isp_ips.py \
  https://github.com/Kukuryzen666/geoipclimanager/releases/download/Release/update_isp_ips.py

# 3. Выдайте права на исполнение
sudo chmod +x /usr/local/bin/update_isp_ips.py

# 4. Запустите CLI (обязательно от root)
sudo update_isp_ips.py
```

> [!IMPORTANT]
> Скрипт требует прав **root**. Все изменения применяются при помощи `systemctl reload nftables` без разрыва текущих соединений.

---

## 🗂 Использование

При запуске без аргументов открывается интерактивное меню:

```
=== ТЕКУЩАЯ КОНФИГУРАЦИЯ ===
Провайдеры (ASNs):
  [MTS]: AS48000, AS8359
  [Beeline (VimpelCom)]: AS3216
  [MGTS]: AS25513

Статус брандмауэра: АКТИВЕН
Автообновление (Cron): ВКЛЮЧЕНО [Ежедневно]
============================

1. Добавить ASN провайдера
2. Удалить ASN провайдера
3. Добавить ручную подсеть в общий белый список
4. Удалить ручную подсеть
5. Добавить точечный доступ (IP -> Порт)
6. Удалить точечный доступ (IP -> Порт)
7. Настроить автообновление (Cron)
8. Применить изменения (Обновить Firewall сейчас)
9. Проверить статус блокировок (Диагностика)
0. Выход
```

### Применение правил

Внесите нужные изменения через пункты **1–6**, затем нажмите **8 (Применить изменения)**:

```
Скрипт :
  1. Скачивает свежие префиксы из RIPEstat API
  2. Агрегирует и схлопывает перекрывающиеся CIDR
  3. Генерирует .nft-файлы
  4. Выполняет мягкий reload: systemctl reload nftables
```

---

## 🏗 Архитектура

Все данные хранятся и генерируются в `/etc/nftables/`:

```
/etc/nftables/
├── geoip_config.json   # Состояние конфигурации (ASN и ручные правила)
├── ru_ips.nft          # Сгенерированный белый список IP-адресов
└── custom_ports.nft    # Гранулярные правила для точечных доступов
```

Все логи (включая срабатывания от Cron) пишутся в:

```
/home/root/geoip_update.log
```

---

## 🔍 Диагностика

Пункт меню **9** делает дамп прямо из оперативной памяти ядра Linux через `nft list set`, позволяя убедиться, что актуальные IP-адреса действительно загружены в Firewall:

```bash
# Эквивалент ручной проверки:
sudo nft list set inet filter allowed_ips
```

---

## 📐 Схема работы

```
Пользователь вводит ASN
        │
        ▼
  RIPEstat API
  (BGP префиксы)
        │
        ▼
  CIDR-агрегация
  (Python ipaddress)
        │
        ▼
  Генерация .nft файлов
        │
        ▼
  systemctl reload nftables
  (без разрыва соединений)
```

---

## 📄 Лицензия

Распространяется под лицензией **MIT License**.  
Вы можете свободно использовать, изменять и распространять данный код.

> [!WARNING]
> Автор не несёт ответственности за возможные сбои в работе сети при некорректной конфигурации брандмауэра.

---

## 🙏 Благодарности

Код писал: Gemini 3.1 Pro

Readme.md составлял: Claude Sonnet 4.6

Данные о префиксах автономных систем предоставляются публичным **[RIPEstat Data API](https://stat.ripe.net/docs/data_api)**.
