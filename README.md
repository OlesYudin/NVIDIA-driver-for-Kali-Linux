# Установка драйверов NVIDIA на Kali Linux 2022

## 1. Проверка видеокарты

Проверить какая видеокарта используется

```bash
lspci | grep -E "VGA|3D"
```

## 2. Отключить проприетарный драйвер NOUVEAU

Что бы отключить **NOEVEAU**, нужно отключить его, обновить конфиг и перезагрузить машину

```bash
sudo echo -e "blacklist nouveau\noptions nouveau modeset=0\nalias nouveau off" | sudo tee --append /etc/modprobe.d/blacklist-nouveau.conf
sudo update-initramfs -u
sudo reboot
```

## 3. Проверка драйвера NOUVEAU

После перезагрузки нужно узнать, работает ли драйвер **NOEVEAU**

```bash
lsmod | grep -i nouveau
```

## 4. Установка драйверов NVIDIA

После отключения **NOEVEAU**, можно установить драйвера **NVIDIA**
Скорее всего драйвера будут установленны не самые свежие, но за то стабильные

```bash
sudo apt-get install nvidia-driver nvidia-xconfig nvidia-prime
```

## 5. Проверка шины видеокарты

Нужно проверить на какой шине находится дискретная видеокарта, для того что бы записать эти значения в конфиг (следующий шаг)

```bash
nvidia-xconfig --query-gpu-info | grep 'BusID'
```

Нужно запомнить значения BusID к примеру `PCI:3:0:0`

## 6. Создание 'Xconfig'

**Xconfig** нужен для того, что бы система понимала как работать с видеокартой, следующий пример заставляет систему работать с дискретной видеокартой.

- Конфиг должен хранится по следующему пути: `/etc/X11/xorg.conf `
- Значение `PCI:x:x:x` нужно поменять на свое, взятое из предыдущего шага.

```bash
Section "ServerLayout"
    Identifier "layout"
    Screen 0 "nvidia"
    Inactive "intel"
EndSection

Section "Device"
    Identifier "nvidia"
    Driver "nvidia"
    BusID "PCI:x:x:x"
EndSection

Section "Screen"
    Identifier "nvidia"
    Device "nvidia"
    Option "AllowEmptyInitialConfiguration"
EndSection

Section "Device"
    Identifier "intel"
    Driver "modesetting"
EndSection

Section "Screen"
    Identifier "intel"
    Device "intel"
EndSection
```

Если Вас не устраивает данный конфиг, его можно переделать под себя по следующей [инструкции](http://us.download.nvidia.com/XFree86/Linux-x86/375.39/README/randr14.html "инструкции")

## 7. Создания скрипта для DisplayManager

Скрипт должен подходить под ваше рабочее окружение. Все скрипты для рабочих окружений можно найти [тут](https://wiki.archlinux.org/title/NVIDIA_Optimus#Display_Managers "тут").

Проверить какой у Вас DisplayManager

```bash
cat /etc/X11/default-display-manager
```

### 8. Скрипт для LightDM

```bash
sudo nano /etc/lightdm/display_setup.sh

# Script:
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```

### 9. Скрипт для SDDM (Default for KDE)

```bash
sudo nano /usr/share/sddm/scripts/Xsetup

# Script:
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```

### 10. Скрипт для GDM

```bash
sudo nano /usr/share/gdm/greeter/autostart/optimus.desktop
sudo nano /etc/xdg/autostart/optimus.desktop

# Script:
[Desktop Entry]
Type=Application
Name=Optimus
Exec=sh -c "xrandr --setprovideroutputsource modesetting NVIDIA-0; xrandr --auto"
NoDisplay=true
X-GNOME-Autostart-Phase=DisplayServer
```

## 11. Перезагрузка системы

После создания скриптов и конфигов, нужно перезагрузить систему

```bash
sudo reboot
```

## 12. Проверка использования драйверов NVIDIA

Если ваша система использует драйвера **NVIDIA**, вывод следующей команды будет таким `direct rendering: Yes`

```bash
glxinfo | grep -i "direct rendering"
```

## 13. Устанвка CUDA toolkit

Теперь можно установить **CUDA**

```bash
sudo apt-get install ocl-icd-libopencl1 nvidia-cuda-toolkit
```

## 14. Обновления GRUB

Если у вас несколько ОС на одной машине, нужно обновить GRUB.

### Проверка использования PRIME NVIDIA

Все значения должны быть "1" (Пример: `PRIME Synchronization: 1`)

```bash
xrandr --verbose | grep PRIME
```

### 15. Обновление GRUB

Конфиг для **GRUB** находится по следующему пути: `/etc/default/grub`

Нужно изменить следующей поле на код ниже:

```bash
....
GRUB_CMDLINE_LINUX_DEFAULT="quiet nvidia-drm.modeset=1"
...
```

После сохранения, нужно обновить **GRUB**

```bash
sudo update-grub
sudo reboot
```

После перезагрузки, проверьте используется ли **PRIME NVIDIA**

```bash
xrandr --verbose|grep PRIME
```

Вывод должен быть следующим:

```bash
PRIME Synchronization: 1
PRIME Synchronization: 1
```

# Проблемы которые могут возникнуть

## 1. Черный экран

Если после перезагрузки у Вас показывается черный экран, нужно:

1. Перейти в терминал с псевдографикой (ALT+CTRL+F2 или ALT+CTRL+F3 и так далее)
2. Удалить созданные ранее конфиги, скорее всего они были созданы не правильно и ииз-за них не работает дисплей

```bash
sudo apt-get remove --purge nvidia*
sudo rm -rf /etc/X11/xorg.conf

# Для LightDM
sudo rm -rf /etc/lightdm/lightdm.conf

# ДляSDDM
sudo rm -rf /usr/share/sddm/scripts/Xsetup

# Для GDM
sudo rm -rf /usr/share/gdm/greeter/autostart/optimus.desktop
sudo rm -rf /etc/xdg/autostart/optimus.desktop
```

3. Перезагрузить систему

```bash
sudo reboot
```

## 2. Не устанавливается драйвер NVIDIA

1. Проверьте, действительно не используетеся **NOUVEAU**

```bash
lsmod | grep -i nouveau
```

2. Если используется, отключите его перейдя к [шагу 2]("шагу 2")

```bash
sudo echo -e "blacklist nouveau\noptions nouveau modeset=0\nalias nouveau off" > /etc/modprobe.d/blacklist-nouveau.conf
sudo update-initramfs -u
```

3. Перезагрузите систему

```bash
sudo reboot
```
