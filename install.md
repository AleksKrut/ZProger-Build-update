# Установка Proger Build (bspwm) на Arch Linux

Оригинальное видео доступно по ссылке [YouTube](https://youtu.be/9zewiGf7j-A),  
репозиторий с конфигурацией — [GitHub](https://github.com/Zproger/bspwm-dotfiles/tree/main).

---

## Перед началом установки **Proger Build** необходимо установить несколько пакетов и редактор `micro`.

Если у вас есть второй компьютер или ноутбук, вы можете подключиться к серверу по SSH и просто вставлять нужные команды — это делается легко.

---

### :computer: Подготовка к подключению по SSH.

Для удобного SSH-подключения рекомендуется использовать **MobaXterm**.
1. Скачайте и установите MobaXterm с [официального сайта](https://mobaxterm.mobatek.net/).
2. На сервере также потребуется установить плагин (или пакет) для корректной работы.  
   Код для установки плагина на сервере:
   ```bash
     pacman -Sy --noconfirm openssh
   ```
после нужно узнать какой у вас IP для этого вводите команду.
   ```bash
   ip address
   ```
Вам необходима строчка enp4s0 там будет прописан ваш сервый IP.

---

### :computer: Как будут выполнены все деуствия по установке **MobaXterm** и план=-гина ssh открываем программу и переходив в кладку session, выбираем пункт SSH и начинаем заполнять

   - `Remote host* ВАШ IP`
   - `Username root`
   - `Port 22`
   
   По окончанию нажимаете ОК. Если все правильно то в терминале у вас потребует ввести пароль от **root**, его задаем на сервере через команду 
    ```bash
   passwd
      ```
   Ввести нужно его два раза. не пугайтесь если на эеране ничвего не появляеться, в linux не показываеться как вводиться пароль.

---
    
### :computer: Пакеты и редактор которые требуеться установить до начала работы с сервером.   

   Установка micro
   >вставлять код в консоль сочетание клавиш shift+ins
      ```bash
   pacman -Sy micro
      ```
   для выхода из редактора micro сочетание клавиш CTRL+Q
   
   установка пакеда dhcpcd
      ```bash
   pacman -S dhcpcd 
      ```
   
   Установка пакета iwd
      ```
   pacman -S iwd
      ```
   Для чистки консоли используется команда 
    ```
   clear
      ```

---

### :computer: Разметка диска под UEFI GPT с шифрованием

   Если вы используете SSD, тогда ваши разделы будут выглядеть примерно так:
   - `/dev/nvme0n1p1`
   - `/dev/nvme0n1p2`
   
   В таком случае замените `/dev/sda` на `/dev/nvme0n1`.
   А разделы `/dev/sda1` и `/dev/sda2` на `/dev/nvme0n1p1` и `/dev/nvme0n1p2`.
   
   для просмотра дисков введите следующую команду 

  ```bash
   lsdlk
   ```
   
  у вас появиться 
  
 - [x] NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
 - [x] loop0    7:0    0 955.7M  1 loop /run/archiso/airootfs
 - [x] sda      8:0    0 465.8G  0 disk
 - [x] └─sda1   8:1    0 465.8G  0 part
 - [x] sdb      8:16   1   7.6G  0 disk
 - [x] └─sdb1   8:17   1   7.6G  0 part
   
   >выбераете название диска то которое вам необходима. У  меня это будет sda

  Как определились с диском приступаем к разметке.
   
   ```bash
   parted /dev/sda
   ```
   ```bash
   mklabel  gpt
   ```
   >пропишите Yes
   ```bash
   mkpart ESP fat32
   ```
   - `start: 1Mib`
   - `end: 512Mib`
   
   ```bash
   set 1 boot on
   ```
   
   ```bash
   mkpart primary
   ```
   - `file system (нажимаем ENTER)`
   - `start: 513Mib`
   - `end: 100%`
   ```bash
   quit
   ```

---

### :computer: Шифруем раздел который подготавливался ранее


```bash
cryptsetup luksFormat /dev/sda2
```
- `sda2 – раздел с шифрованием`
- `вводим YES большими буквами`
- `вводим пароль 2 раза`

Открываем зашифрованный раздел
```bash
cryptsetup open /dev/sda2 luks
```
>нужно будет ввести пароль который создавали командой cryptsetup luksFormat /dev/sda2

Проверяем разделы
```bash
ls /dev/mapper/*
```
Создаем логические разделы внутри зашифрованного раздела
```bash
pvcreate /dev/mapper/luks
```
```bash
vgcreate main /dev/mapper/luks
```
100% зашифрованного раздела помещаем в логический раздел root
```bash
lvcreate -l 100%FREE main -n root
```
Посмотреть все логические разделы
```bash
lvs
```

---

### :computer: Подготовка разделов и монтирование

Форматируем раздел под ext4
```bash
mkfs.ext4 /dev/mapper/main-root
```
Форматируем boot раздел под Fat32, на физ.разделе /dev/sda1 лежит boot
```bash
mkfs.fat -F 32 /dev/sda1
```
Монтируем разделы для установки системы
```bash
mount /dev/mapper/main-root /mnt
```
```bash
mkdir /mnt/boot
```
Монтируем раздел с boot в текущую рабочую папку
```bash
mount /dev/sda1 /mnt/boot
```

---

### :computer: Сборка ядра и базовых софтов

Устанавливаем базовые софты
```bash
pacstrap -K /mnt base linux linux-firmware base-devel lvm2 dhcpcd net-tools iproute2 networkmanager vim micro efibootmgr iwd
```
Генерируем fstab
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
```bash
cat /mnt/etc/fstab
```
Настройка системы
```bash
arch-chroot /mnt
```
>Нужно раскомментировать ru_RU и en_US в формате UTF-8 в этом файле, для поиска ипользуеться сочитание клавиш ctrl+F
```bash
micro /etc/locale.gen
```
Генерируем локали
```bash
locale-gen
```
Настраиваем время
```bash
ln -sf /usr/share/zoneinfo/Europe/Kiev /etc/localtime
```
```bash
hwclock --systohc
```
Указать имя хоста
```bash
echo “arch” > /etc/hostname
```
Укажите пароль для root пользователя
>этот шаг мы уже сделали в самом начале.
```bash
passwd
```
Добавляем нового пользователя и настраиваем права
```bash
useradd -m -G wheel,users,video -s /bin/bash user
```
```bash
passwd user
```
```bash
systemctl enable dhcpcd
```
```bash
systemctl enable iwd.service
```
```bash
micro /etc/mkinitcpio.conf
```
>Пересборка ядра. Найдите строку HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block filesystems fsck)
и замените на: HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block filesystems encrypt lvm2 fsck)

Запустить процесс пересборки ядра
```bash
mkinitcpio -p linux
```

---

### :computer: Установка загрузчика

```bash
bootctl install --path=/boot
```
```bash
cd /boot/loader
```
```bash
micro loader.conf
```
Вставляем в loader.conf следующий конфиг:
```bash
timeout 3
default arch
```
Создаем конфигурацию для запуска
```bash
cd /boot/loader/entries
```
```bash
micro arch.conf
```
Вставляем в arch.conf следующее:
>UUID можно узнать командой blkid
```bash
title Arch Linux by ZProger
linux /vmlinuz-linux
initrd /initramfs-linux.img
options rw cryptdevice=UUID=uuid_от_/dev/sda2:main root=/dev/mapper/main-root
```
сохраняем и выходим, после вводим команду 
```bash
blkid >> arch.conf
```
и заходим обрантно в arch.conf для замены uuid_от_/dev/sda2 на тот который будет в редакторе.

Выдаем права на sudo
```bash
sudo EDITOR=micro visudo
```
После открытия раскомментируйте %wheel ALL=(ALL:ALL) ALL

Выходим из системы и перезагружаемся
```bash
Ctrl+D
```
или
```bash
exit
```
дальше вводим 
```bash
umount -R /mnt
```
и
```bash
reboot
```

---

### :computer:После перезагрузки для повторного подклбючения по SSH введиет следующие команды 

```bash
sudo systemctl enable sshd 
```
>ключить автозапуск при загрузке
```bash
sudo systemctl start sshd
```
>запустить службу сейчас

---

### :computer:Устанавливаем оболочку

Если при загрузке системы вы получаете ошибки или у вас открывается окно от iso образа Arch'a, тогда необходимо отмонтировать образ или вытащить флешку.
Также убедитесь что загрузка идет под EFI, особенно это касается виртуальных машин.

Перед выполнением этих команд, авторизуйтесь в пользователя user. На этапе загрузки система попросит ввести пароль для дешифровки области жесткого диска,
и в дальнейшем вам будет предложено войти в пользователя, введя логин и пароль. После авторизации выполняем следующее:

```bash
sudo pacman -Syu
```
```bash
sudo pacman -S xorg bspwm sxhkd xorg-xinit xterm git python3
```
Настройка xinitrc
```bash
micro /etc/X11/xinit/xinitrc
```
Отключите любые другие строки exec и добавьте в конец файла строку:
```bash
exec bspwm
```

Загрузите репозиторий локально, но перед выполнением билдера я рекомендую перейти в `Builder/packages.py` и посмотреть пакеты, которые будут установлены.
Я не советую редактировать `BASE_PACKAGES`, так как они необходимы для правильной работы оболочки, однако вы свободно можете редактировать другие виды пакетов.
На этапе билдера вам будет предложено установить `DEV_PACKAGES`, они не нужны для системы, но могут быть полезны для разработки. Выбирайте пункты на свое усмотрение.

и выполните сборку оболочки используя данные команды:
```bash
git clone https://github.com/Zproger/bspwm-dotfiles.git
```
```bash
cd bspwm-dotfiles
```
```bash
python3 Builder/install.py
```

В меню необходимо предоставить разрешение на установку `dotfiles`, обновление баз, установку `BASE_PACKAGES`. Остальные пункты выбирайте самостоятельно.
Такое разделение опций позволяет выполнить только необходимое действие, к примеру лишь заменить `dotfiles` либо установить актуальные `DEV_PACKAGES` пакеты.

Если вы все сделали правильно, то после запуска вы получите готовую оболочку BSPWM. Не забудьте включить 3D-ускорение, если вы работаете под виртуальной машиной.
```bash
startx
```

Из-за разного железа / разных дистрибутивов и прочих моментов, могут быть небольшие проблемы в отображении иконок, в работе с батареей / яркостью. Решение этих
проблем было показано в [данном видео](https://youtu.be/9zewiGf7j-A).
