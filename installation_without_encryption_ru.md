# 🚀 Установка Proger Build (bspwm) на Arch Linux без шифрования

Оригинальное видео: [YouTube](https://youtu.be/9zewiGf7j-A)  
Репозиторий конфигурации: [GitHub](https://github.com/Zproger/bspwm-dotfiles)

---

## 💾 Полезные команды 
- Сахранить в micro <kbd>CTRL</kbd>+<kbd>S</kbd>
- Поиск в редакторе <kbd>CTRL</kbd>+<kbd>F</kbd>
- Выход из micro: <kbd>CTRL</kbd>+<kbd>Q</kbd>
- Очистка консоли: <kbd>Clear</kbd>

---

## 📌 Перед началом

Установите необходимые пакеты и редактор `micro`.  
Если у вас есть второй компьютер, вы можете подключиться по SSH и вставлять команды.

<details>
    <summary>
        <strong>
            🖥️ Подготовка к подключению по SSH
        </strong>
    </summary>

>На компьютер на котором будет производиться подклчение по ssh Рекомендуется использовать **MobaXterm** ([скачать](https://mobaxterm.mobatek.net/)). 

На запущенный сервере установите `openssh`:

```bash
pacman -Sy --noconfirm openssh
```

Узнайте IP:

```
ip a
```

Найдите строку enp4s0 – там будет серверный IP.

Настройка MobaXterm:

- Session → SSH

- Remote host: ваш IP

- Username: root

- Port: 22

OK

>Если всё правильно, потребуется ввести пароль root. Задайте его на сервере:

```
passwd
```

>Вводите пароль дважды (символы не отображаются).

</details>

<details> 
    <summary
        ><strong>📦 Установка пакетов и редактора</strong>
    </summary>
<br>

```
pacman -Sy micro dhcpcd iwd
```

</details>

## 1️⃣ Начинаем оснавную установку

<details>
    <summary>
        <strong>💽 Разметка диска (UEFI GPT без шифрования)</strong>
    </summary>

🔹Просмотр дисков:

```
lsblk
```

>Если используется SSD, разделы будут /dev/nvme0n1p1 и /dev/nvme0n1p2.

🔹В примере диск – /dev/sda.

```
parted /dev/sda
```

```
mklabel gpt
```

```
mkpart ESP fat32 1MiB 512MiB
```

```
set 1 boot on
```

```
mkpart primary ext4 512MiB 100%
```

```
quit
```

</details>

<details> 
    <summary>
        <strong>🔧 Форматирование разделов</strong>
    </summary>

  ```
  mkfs.fat -F 32 /dev/sda1
  mkfs.ext4 /dev/sda2
  ```

</details>

<details> 
    <summary>
        <strong>⚙️ Установка базовых пакетов</strong>
    </summary>

```
pacstrap -K /mnt base linux linux-firmware base-devel dhcpcd net-tools iproute2 networkmanager vim micro efibootmgr iwd
```
</details>

## 2️⃣Сборка ядра и базовых софтов


<details> 
    <summary>
        <strong>
            📄 Генерация fstab
        </strong>
    </summary>
    
```
genfstab -U /mnt >> /mnt/etc/fstab
```
```
cat /mnt/etc/fstab
```

</details>

<details> 
    <summary>
        <strong>
            🔧 Настройка системы (chroot)
        </strong>
    </summary>
<br>
🔹Переходим в режим 

```
arch-chroot /mnt
```

🔹Для изменения локализации раскомментируйте строки ru_RU.UTF-8 UTF-8 и en_US.UTF-8 UTF-8

```
sed -i 's/^#\(ru_RU.UTF-8\)/\1/; s/^#\(en_US.UTF-8\)/\1/' /etc/locale.gen
```
>Если вам только `en_US.UTF-8 UTF-8` нужен то используйте команду `micro /etc/locale.gen` через поиск найдите `en_US.UTF-8 UTF-8` и раскоментируйте.

🔹Генерируем локали

```
locale-gen
```

🔹Настраиваем время

```
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc
```

🔹Указать имя хоста

```
echo “arch” > /etc/hostname
```

🔹Создания пароля для `root` пользователя
```
passwd
```
> ⚠️ WARNING <BR>Если вы в начале уже установили пароль для `root` данную функцию не обязательно выполнять. <br> Также этой командой можно изменять пароль для созданных пользователей выполнив команду <br>
`sudo passwd имя пользователя` 

🔹Добавляем нового пользователя и настраиваем права
```
useradd -m -G wheel,users,video -s /bin/bash user
passwd user
systemctl enable dhcpcd
systemctl enable iwd.service
```
🔹Настройка mkinitcpio (без шифрования):
>Убедитесь, что в строке HOOKS нет encrypt и lvm2 Пример:<br>`HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block filesystems fsck)`
```
micro /etc/mkinitcpio.conf
```
```
mkinitcpio -P
```
</details>

## 3️⃣ Установка
<details>
    <summary>
        <strong>💿 Установка загрузчика</strong>
    </summary>
<br>

```
bootctl install

```

```
cat > /boot/loader/loader.conf << EOF
timeout 3
default arch
EOF
```

>Вставляем в loader.conf следующий конфиг<br> `timeout 3`<br>`default arch`

```
cat > /boot/loader/entries/arch.conf << EOF
title Arch Linux by ZProger
linux /vmlinuz-linux
initrd /initramfs-linux.img
options root=UUID=$(blkid -s UUID -o value /dev/sda2) rw
EOF
```
>Вставляем в arch.conf `uuid_от_/dev/sda2`

<br>

🔹Выдаем права на `sudo`

```
sudo EDITOR='sed -i "/^# %wheel ALL=(ALL:ALL) ALL/s/^# //"' visudo
```
>После ввода раскомментируется %wheel ALL=(ALL:ALL) ALL

🔹Выходим из системы <kbd>Ctrl+D</kbd>

```
umount -R /mnt
```
🔹Перезагружаемся
```
reboot
```
</details>

## 🔄 После перезагрузки включите SSH
<details>
    <summary>
        <strong>
            код для ssh
        </strong>
    </summary>

```
sudo systemctl enable sshd
sudo systemctl start sshd
```
</details>

## 4️⃣ Установка оболочки
<details>
    <summary>
        <strong>🎨 Установка оболочки BSPWM</strong>
    </summary>
<br>


>⚠️ WARNING ⚠️ <br>Если при загрузке системы вы получаете ошибки или у вас открывается окно от iso образа Arch'a, тогда необходимо отмонтировать образ или вытащить флешку. Также убедитесь что загрузка идет под EFI, особенно это касается виртуальных машин.

>❗ATTENTION❗<br>Перед выполнением этих команд, авторизуйтесь в пользователя user. На этапе загрузки система попросит ввести пароль для дешифровки области жесткого диска, и в дальнейшем вам будет предложено войти в пользователя, введя логин и пароль. После авторизации выполняем следующее:

```
sudo pacman -Syu
sudo pacman -S xorg bspwm sxhkd xorg-xinit xterm git python3
```

🔹Настройка xinitrc
>Отключите любые другие строки exec и добавьте в конец файла строку: `exec bspwm`
```
micro /etc/X11/xinit/xinitrc
```
<br>

>❗ATTENTION❗<BR>Загрузите репозиторий локально, но перед выполнением билдера я рекомендую перейти в `Builder/packages.py` и посмотреть пакеты, которые будут установлены. Я не советую редактировать `BASE_PACKAGES`, так как они необходимы для правильной работы оболочки, однако вы свободно можете редактировать другие виды пакетов. На этапе билдера вам будет предложено установить `DEV_PACKAGES`, они не нужны для системы, но могут быть полезны для разработки. Выбирайте пункты на свое усмотрение и выполните сборку оболочки используя данные команды:

```
git clone https://github.com/Zproger/bspwm-dotfiles.git
cd bspwm-dotfiles
python3 Builder/install.py
```
>❗ATTENTION❗<BR>В меню необходимо предоставить разрешение на установку `dotfiles`, обновление баз, установку `BASE_PACKAGES`. Остальные пункты выбирайте самостоятельно. Такое разделение опций позволяет выполнить только необходимое действие, к примеру лишь заменить `dotfiles` либо установить актуальные `DEV_PACKAGES` пакеты.

🔹Если вы все сделали правильно, то после запуска вы получите готовую оболочку BSPWM. Не забудьте включить 3D-ускорение, если вы работаете под виртуальной машиной

```
startx
```

🔹Из-за разного железа / разных дистрибутивов и прочих моментов, могут быть небольшие проблемы в отображении иконок, в работе с батареей / яркостью. Решение этих проблем было показано в данном видео[YouTube](https://youtu.be/9zewiGf7j-A).
