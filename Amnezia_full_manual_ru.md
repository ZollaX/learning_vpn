# AmneziaVPN (Self‑Hosted) — Полное руководство для пользователей из России
*Последнее обновление: 2025-05-03*

> Это руководство объединяет официальную документацию Amnezia Docs, репозитории GitHub и практический опыт развёртывания сервисов в России. Используя данный файл, вы сможете в любой момент самостоятельно установить и настроить AmneziaVPN (с протоколом **AmneziaWG**) на собственном VPS и управлять им из‑за рубежа или из‑внутри Российской Федерации.

---

## Содержание
1. [Общее представление](#общее-представление)  
2. [Архитектура приложения](#архитектура-приложения)  
3. [Выбор VPS, подходящего под РКН‑блокировки](#выбор-vps-под-rkn)  
4. [Подготовка среды (Linux Manjaro и Windows)](#подготовка-среды)  
5. [Пошаговое развёртывание сервера](#пошаговое-развёртывание-сервера)  
6. [Настройка клиента Amnezia под Россию](#настройка-клиента)  
7. [Управление серверами и совместный доступ](#управление-и-шаринг)  
8. [Продвинутая маскировка и обход DPI](#masking)  
9. [Резервное копирование, обновление, удаление](#maintenance)  
10. [Частые ошибки и их устранение](#faq)  
11. [Полезные ресурсы](#res)  

---

## Общее представление
**AmneziaVPN** — свободный много‑протокольный VPN‑клиент с функцией автоматической установки своего VPN‑сервера (self‑hosted).  
Ключевые особенности:

* Установка сервера «в один клик» через SSH + Docker.  
* Поддержка протоколов: AmneziaWG, WireGuard, OpenVPN, OpenVPN+Cloak, ShadowSocks, IPSec (IKEv2), XRay Reality.  
* Инструменты маскировки трафика (junk‑packets, magic‑headers, cloak, reality).  
* Кроссплатформенные клиенты (Windows, macOS, Linux, iOS, Android, Android TV).  
* Возможность делиться подключением в виде QR‑кода, файла конфигурации или текстового ключа.  

---

## Архитектура приложения
```
┌───────────────┐      SSH       ┌─────────────────────┐
│ Amnezia Client│ ─────────────▶ │ VPS (Docker Engine) │
│  (GUI / CLI) │                │  amnezia-service    │
└───────────────┘                │  protocol containers│
      ▲ VPN tunnel (WG/OVPN/…)   └─────────────────────┘
```
* **amnezia-service** — базовый контейнер, управляющий API и жизненным циклом протоколов.  
* На каждый протокол (AmneziaWG, Shadowsocks и т.д.) поднимается отдельный контейнер.  
* Все компоненты устанавливаются автоматически мастер‑скриптом при первом подключении.

---

## Выбор VPS под RKN
1. **Виртуализация** — только **KVM**: OpenVZ не поддерживает WireGuard/AmneziaWG.
2. **Расположение** — Нидерланды, Германия, Исландия.  
3. **Минимальные ресурсы** — 1 vCPU, 1 ГБ RAM, 20 ГБ SSD.  
4. **Провайдеры, работающие с российскими картами**: *Inferno Solutions*, *TimeWeb Cloud*, *FirstVDS*.  
5. **Порты** — убедитесь, что UDP‑порт (по умолчанию 51820 или случайный) открыт во внешнем фаерволе.

---

## Подготовка среды
### Linux Manjaro (клиент)
```bash
sudo pacman -Syu --needed wireguard-tools libxcb libxinerama libxcursor iptables
mkdir -p ~/apps && cd ~/apps
wget https://github.com/amnezia-vpn/amnezia-client/releases/latest/download/AmneziaVPN_x86_64.AppImage -O AmneziaVPN.AppImage
chmod +x AmneziaVPN.AppImage
./AmneziaVPN.AppImage &
```

### Windows (клиент)
1. Скачайте **AmneziaVPN_Setup_x64.exe** с GitHub Releases.  
2. Запустите установщик, оставив все опции по умолчанию.  
3. После установки проверьте наличие драйвера *Wintun*.

---

## Пошаговое развёртывание сервера
1. **Создайте VPS** и запишите `IP`, `root` пароль.  
2. Запустите AmneziaVPN → **Let’s Get Started** → *Self‑hosted VPN*.  
3. Введите данные `IP:22`, `root`, пароль. Если SSH порт изменён — укажите через двоеточие (`203.0.113.10:2222`).  
4. Выберите **High control** — будет установлен **AmneziaWG**.  
5. Дождитесь завершения скрипта (≈ 2‑3 мин); containers + API key сохраняются локально.  
6. Нажмите **Connect** — трафик пойдёт через VPS.

---

## Настройка клиента
### Split‑tunnel
*Settings → Split VPN tunneling* — добавьте сайты банков, «Госуслуги» и т.д., чтобы открывались без VPN.

### Изменение обфускации AmneziaWG
```
⚙  → Protocols → AmneziaWG → Edit
S1/S2   # размер паддинга auth‑пакетов
Jc      # количество junk‑пакетов до рукопожатия
Jmin/Jmax  # диапазон размера junk‑пакетов
MH1‑MH4    # magic‑headers
```
При массовой блокировке поменяйте значения, затем *Save & Restart*.

---

## Управление и шаринг
* Share → **QR‑код / Text key / Config file** — передайте настройки второму устройству.  
* **Sharing server management rights** — выдайте одноразовый API token тех.поддержке, не раскрывая пароль root.  

---

## <a name="masking"></a>Продвинутая маскировка и обход DPI
| Сценарий | Рекомендуемый протокол | Комментарий |
|----------|------------------------|-------------|
| Провайдер блокирует WireGuard UDP | **AmneziaWG** | Измените `MH*`, `S1/S2` |
| DPI режет TCP 443 OpenVPN | **OpenVPN+Cloak** | Cloak маскирует под HTTPS |
| Требуется прокси | **Shadowsocks** | Конфиг `SS://` можно дать в ShadowRocket |
| Нужен VLess/Reality | **XRay Reality** | Поддержка TLS fronting |

---

## <a name="maintenance"></a>Резервное копирование, обновление, удаление
### Бэкап
*Menu → Backup and restore → Backup* → сохраняется `.vpn` файл.

### Обновление клиента
```bash
./AmneziaVPN.AppImage --update   # Linux
```
Windows — Settings → *Check for updates*.

### Обновление контейнеров на VPS
*Server ⚙ → Check for updates*.

### Полное удаление
```bash
docker compose -f /opt/amnezia/docker-compose.yml down --remove-orphans
rm -rf /opt/amnezia
```

---

## <a name="faq"></a>Частые ошибки
| Симптом | Решение |
|---------|---------|
| GUI не запускается (Linux) | Установите `libxcb-xinerama0` и `libxcb-cursor0` |
| Кнопка Connect неактивна | Включите IPv6: `sudo sysctl -w net.ipv6.conf.all.disable_ipv6=0` |
| «undefined symbol: wl_proxy_marshal_flags» | Запустите сессии X11 вместо Wayland |
| Нет трафика через VPN | Проверьте, открыт ли UDP‑порт в VPS‑фаерволе и маршруты split‑tunnel |

---

## <a name="res"></a>Полезные ресурсы
* Документация: <https://docs.amnezia.org>  
* Репозитории: <https://github.com/amnezia-vpn>  
* Телеграм‑чат: `@amnezia_vpn`  
* Reddit‑сообщество: `r/AmneziaVPN`  

---

> Сохраните этот файл — он позволит восстановить рабочий VPN даже при полном блокировании общедоступных сервисов.
