# 🔐 AmneziaWG для Manjaro Linux

[![Актуальная версия](https://img.shields.io/badge/Актуально-2025--04--30-brightgreen.svg)](https://github.com/amnezia-vpn/amneziawg-go)
[![Manjaro Support](https://img.shields.io/badge/Manjaro-Поддерживается-success)](https://manjaro.org/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

**AmneziaWG** — это форк WireGuard-Go с продвинутыми механизмами обфускации, делающими VPN-трафик неотличимым от обычного UDP‑трафика. Сохраняя производительность оригинального WireGuard, он обеспечивает надёжную защиту от систем DPI.

## 📑 Содержание

- [🔐 AmneziaWG для Manjaro Linux](#-amneziawg-для-manjaro-linux)
  - [📑 Содержание](#-содержание)
  - [✨ Особенности](#-особенности)
  - [⚡ Быстрый старт](#-быстрый-старт)
  - [📋 Системные требования](#-системные-требования)
  - [🛠 Установка](#-установка)
    - [Клиент на Manjaro](#клиент-на-manjaro)
    - [Сервер](#сервер)
      - [4.1 Покупаем VPS](#41-покупаем-vps)
      - [4.2 Первичное обновление](#42-первичное-обновление)
      - [4.3 Автоматическая установка через Amnezia Client](#43-автоматическая-установка-через-amnezia-client)
  - [⚙️ Конфигурация](#️-конфигурация)
    - [Обфускация](#обфускация)
    - [Split-Tunneling](#split-tunneling)
    - [Kill-Switch](#kill-switch)
  - [�� Дополнительные сервисы](#-дополнительные-сервисы)
  - [🔄 Обновление и обслуживание](#-обновление-и-обслуживание)
    - [Клиент](#клиент)
    - [Сервер](#сервер-1)
  - [🚨 Устранение неполадок](#-устранение-неполадок)
  - [📚 Справочник команд](#-справочник-команд)
  - [🔗 Полезные ссылки](#-полезные-ссылки)

## ✨ Особенности

| Характеристика | WireGuard® | **AmneziaWG** |
|---------------|-----------|---------------|
| Производительность | Высокая | **Высокая** |
| Настройки | Минимум | Минимум + обфускация |
| Обнаружение DPI | **Да** | **Нет** |
| Поддержка в AmneziaVPN | ✓ | **✓** |

**Ключевые улучшения:**
- 🔒 Рандомизация заголовков пакетов
- 📦 Настраиваемые размеры аутентификации
- 🌫 Генерация "маскировочных" пакетов
- ⚡ Сохранение производительности оригинала

## ⚡ Быстрый старт

```bash
# 1. Установка клиента
sudo pacman -S wireguard-tools
wget https://github.com/amnezia-vpn/amnezia-client/releases/latest/download/AmneziaVPN-x86_64.AppImage

# 2. Запуск
chmod +x AmneziaVPN-x86_64.AppImage
./AmneziaVPN-x86_64.AppImage
```

## 📋 Системные требования

| Компонент | Минимум | Рекомендуется |
|-----------|---------|---------------|
| **ОС** | Manjaro Linux | Последняя версия |
| **CPU** | 1 vCPU | 2+ vCPU |
| **RAM** | 512 MB | ≥1 GB |
| **Диск** | 10 GB | 25+ GB |
| **Сеть** | 10 Mbps | 100+ Mbps |

> ⚠️ **Важно**: OpenVZ-виртуализация не поддерживается!

## 🛠 Установка

### Клиент на Manjaro

```bash
# 1. Обновляем систему
sudo pacman -Syu

# 2. Ставим зависимости WireGuard (ядро и cli)
sudo pacman -S --needed wireguard-tools

# 3. Скачиваем последнюю версию AmneziaVPN (AppImage)
mkdir -p ~/apps
cd ~/apps
wget https://github.com/amnezia-vpn/amnezia-client/releases/latest/download/AmneziaVPN-x86_64.AppImage -O AmneziaVPN.AppImage

# 4. Делаем файл исполняемым
chmod +x AmneziaVPN.AppImage

# 5. Запускаем
./AmneziaVPN.AppImage
```

> *Совет:* перетащите AppImage в `/usr/local/bin` и создайте `.desktop`‑файл, чтобы приложение появилось в меню.

### Сервер

#### 4.1 Покупаем VPS

* Провайдер **KVM**, 1 ГБ RAM, 25 ГБ SSD.  
* Образ **Ubuntu 22.04 LTS**.  
* Получаем `IP`, `root` пароль (или SSH‑ключ).

#### 4.2 Первичное обновление

```bash
ssh root@<IP>
apt update && apt upgrade -y
reboot
```

#### 4.3 Автоматическая установка через Amnezia Client

1. Откройте AmneziaVPN → **Let's Get Started**.  
2. Выберите **Self‑Hosted VPN** и заполните:  

```
IP‑адрес:  203.0.113.10
Пользователь: root
Пароль:     *****  (или прикрепите SSH‑ключ)
Порт:       22
```

3. **Continue** → выберите уровень цензуры:  

* *High* — будет установлен **AmneziaWG** (рекомендуется).  
* *Low* — обычный WireGuard.

4. Дождитесь автоматической установки (≈ 2 мин).  

После окончания нажмите **Connect**.

## ⚙️ Конфигурация

### Обфускация

AmneziaWG изменяет:  

* **Заголовки пакетов** (Initiator→Responder, Responder→Initiator, Data)  
* Размеры аутентификационных пакетов (`S1`, `S2`)  
* Добавляет «мусорные» пакеты до рукопожатия (`Jc`, `Jmin`, `Jmax`)  

Благодаря этому DPI не может составить сигнатуру и заблокировать соединение.

### Split-Tunneling

> **Settings → Split VPN tunneling**  
> Добавьте сайты, которые должны идти **в обход** VPN (например, банк).

### Kill-Switch

*Включено по умолчанию:* трафик блокируется при обрыве туннеля.

## �� Дополнительные сервисы

| Сервис | Где включить | Что даёт |
|--------|--------------|----------|
| **SOCKS5 proxy** | *Services → SOCKS5* | Отдельный прокси‑порт с логином/паролем |
| **SFTP‑хранилище** | *Services → SFTP* | Личный облачный диск |
| **TOR‑сайт (.onion)** | *Services → TOR Site* | Скрытый сервис в сети TOR |

## 🔄 Обновление и обслуживание

### Клиент

```bash
./AmneziaVPN.AppImage --update
```

или скачайте новую AppImage.

### Сервер

В клиенте: **Server Settings → Reboot**  
Проверить версии контейнеров: **Check for updates**.

## 🚨 Устранение неполадок

| Симптом | Решение |
|---------|---------|
| Кнопка *Connect* неактивна | Проверьте, что **IPv6 включён** (`sysctl net.ipv6.conf.all.disable_ipv6`) → должно быть `0`. |
| Ошибка `wg setconf` | Убедитесь, что ядро Manjaro поддерживает WireGuard (`modprobe wireguard`). |
| Нет трафика после соединения | Отключите сторонние фаерволы (UFW, nftables) или добавьте правило `wg0` |

## 📚 Справочник команд

```bash
# Перезапуск туннеля вручную
sudo wg-quick down awg0 && sudo wg-quick up awg0

# Показ статистики
sudo wg show awg0

# Список активных обфусц‑параметров
sudo wg showconf awg0
```

## 🔗 Полезные ссылки

- 📖 [Официальная документация](https://docs.amnezia.org)
- 💻 [Исходный код AmneziaWG](https://github.com/amnezia-vpn/amneziawg-go)
- 💬 [Telegram-поддержка](https://t.me/amnezia_vpn)
- 🐛 [Баг-трекер](https://github.com/amnezia-vpn/amneziawg-go/issues)

---

<p align="center">
🛡️ <strong>Безопасность. Приватность. Свобода.</strong><br>
Создано с ❤️ для сообщества Manjaro
</p>

> **Примечание**: Сохраните этот файл — он поможет вам быстро развернуть защищённый VPN даже через несколько лет.
