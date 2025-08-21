# Podkop + ByeDPI на OpenWRT

Гайд по установке и настройке **Podkop** вместе с **ByeDPI** на OpenWRT.

---

## Установка ByeDPI

### 1. Узнайте архитектуру устройства

```sh
opkg print-architecture
awk -F\' '/DISTRIB_ARCH/ {print $2}' /etc/openwrt_release
```

### 2. Скачайте нужный пакет

Замените `aarch64_cortex-a53` на свою архитектуру:

```sh
(cd /tmp && curl -LO https://github.com/DPITrickster/ByeDPI-OpenWrt/releases/download/v0.17.2-24.10/byedpi_0.17.2-r1_aarch64_cortex-a53.ipk)
```

### 3. Установите пакет

(при необходимости удалите старую версию)

```sh
opkg remove byedpi
opkg install /tmp/byedpi_0.17-r1_aarch64_cortex-a53.ipk
```

### 4. Настройте конфиг ByeDPI

Откройте файл:

```sh
vi /etc/config/byedpi
```

Добавьте рабочую стратегию (пример):

```sh
config byedpi
    option enabled '1'
    option options '-o 2 --auto=t,r,a,s -d 2'
```

⚠️ Подберите параметры под своего провайдера (см. [ByeByeDPI](https://github.com/romanvht/ByeByeDPI) или [ByeDPI Manager](https://github.com/romanvht/ByeDPIManager)).

### 5. Включите сервис

```sh
/etc/init.d/byedpi enable
/etc/init.d/byedpi start
```

### 6. Для OpenWRT 24.10 отключите использование `dnsmasq` как локального резолвера

(иначе могут быть проблемы с DNS для приложений на роутере):

```sh
uci set dhcp.@dnsmasq[0].localuse='0'
uci commit dhcp
/etc/init.d/dnsmasq restart
```

---

## Настройка Podkop

### 7. Добавьте Outbound для ByeDPI

Откройте [настройки Outbound](https://podkop.net/docs/ownoutbound/) и добавьте:

```json
{
  "type": "socks",
  "server": "127.0.0.1",
  "server_port": 1080
}
```

Если вы изменили порт ByeDPI, замените `1080` на ваш.

---

## Финальные шаги

### 8. Перезагрузите роутер

```sh
reboot
```

### 9. Проверьте работу ByeDPI

```sh
ps | grep byedpi
netstat -tulnp | grep 1080
```

Если процессы активны — всё работает.

---

## Вы великолепны 🚀
