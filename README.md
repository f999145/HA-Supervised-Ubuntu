- Плата: OrangePi 5 Plus
- Образ взят с GitHub [Joshua-Riek](https://github.com/Joshua-Riek/ubuntu-rockchip)
- Ubuntu desktope 24.04 [image](https://github.com/Joshua-Riek/ubuntu-rockchip/releases/download/v2.3.0/ubuntu-24.04-preinstalled-desktop-arm64-orangepi-5-plus.img.xz)
- За основу взял [этот скрипт действий](https://pimylifeup.com/ubuntu-home-assistant/)


# Подготовка Ubuntu к запуску под руководством Home Assistant
## Обновление пакетов

```bash
sudo apt update
sudo apt upgrade
```

## Установка нужных пакетов

```bash
sudo apt-get install nfs-kernel-server
sudo apt install apparmor jq wget curl udisks2 libglib2.0-bin network-manager dbus lsb-release systemd-journal-remote binutils -y
```

## Устанавливаем Docker

```bash
curl -fsSL get.docker.com | sh
```

## Добавляем пользователя docker в группу

```bash
sudo usermod -aG docker $USER
```

## Установка OS-Agent

```bash
curl -s https://api.github.com/repos/home-assistant/os-agent/releases/latest | grep "browser_download_url.*aarch64\.deb" | cut -d : -f 2,3 | tr -d \" | wget -O os-agent-aarch64.deb -i - 
sudo dpkg -i os-agent-aarch64.deb
```

Проверка OS-Agent на ошибки (У меня есть ошибки, пока непонятно, влияют ли они на что либо)

```bash
gdbus introspect --system --dest io.hass.os --object-path /io/hass/os
```


# Загрузка и Модификация Home Assistant package для Ubuntu

## Загрузка

```bash
mkdir /tmp/homeassistant-supervised
cd /tmp/homeassistant-supervised
```

```bash
wget https://github.com/home-assistant/supervised-installer/releases/latest/download/homeassistant-supervised.deb
```

## Модификация пакета

```bash
ar x homeassistant-supervised.deb
tar xf control.tar.xz
```

```bash
nano control
```

В первоначальной версии было по другому, Я же удалил эти строчки:

```text
Pre-Depends: ...
Depends: ...
```

И вставил на место их это

```text
Depends: curl, bash, docker-ce, dbus, network-manager, apparmor, jq, systemd, os-agent, systemd-journal-remote
```

## Перепаковка архива

```bash
tar cfJ control.tar.xz postrm postinst preinst control templates
ar rcs homeassistant-supervised.deb debian-binary control.tar.xz data.tar.xz
```

# Установка самого Home Assistant в Ubuntu
Сначала устанавливаем переменную окружения “ `BYPASS_OS_CHECK`” на “`true`” таким образом, установщик не будет проверять, что мы не запускаем Debian.

```bash
sudo BYPASS_OS_CHECK=true dpkg -i ./homeassistant-supervised.deb
```

На случай этой ошибки:
> dpkg: error processing package homeassistant-supervised (--install):
> 
 >    installed homeassistant-supervised package post-installation script subprocess returned error exit status 2
 >    
>Errors were encountered while processing:
>
 >    homeassistant-supervised
 
```bash
sudo apt --fix-broken install
```

# P.S.

Нужно будет остановить контейнер "hassio_audio",

Он блокирует аудио, из-за чего неработает воспроизведение видео.
