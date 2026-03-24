# 🚀 Установка Proger Build (bspwm) на Arch Linux без шифрования

Оригинальное видео: [YouTube](https://youtu.be/9zewiGf7j-A)  
Репозиторий конфигурации: [GitHub](https://github.com/Zproger/bspwm-dotfiles)

---

## Полезные команды 
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

Если всё правильно, потребуется ввести пароль root. Задайте его на сервере:

```
passwd
```

Вводите пароль дважды (символы не отображаются).

</details>
<details> 
    <summary
        ><strong>📦 Установка пакетов и редактора</strong></summary>

```
pacman -Sy micro dhcpcd iwd
```
---
</details>

## После того как вы все подготовили начинаем оснавную установку

<details>
    <summary>
        <strong>💽 Разметка диска (UEFI GPT без шифрования)</strong></summary>

Просмотр дисков:

```
lsblk
```
Если используется SSD, разделы будут /dev/nvme0n1p1 и /dev/nvme0n1p2.
В примере диск – /dev/sda.
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
    <summary><strong>🔧 Форматирование разделов</strong></summary>

  ```
  mkfs.fat -F 32 /dev/sda1
  mkfs.ext4 /dev/sda2
  ```
</details>
<details> <summary><strong>⚙️ Установка базовых пакетов</strong></summary>

```
pacstrap -K /mnt base linux linux-firmware base-devel dhcpcd net-tools iproute2 networkmanager vim micro efibootmgr iwd
```
</details>

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

```
arch-chroot /mnt
```
Для изменения локализации раскомментируйте строки ru_RU.UTF-8 UTF-8 и en_US.UTF-8 UTF-8

```
sed -i 's/^#\(ru_RU.UTF-8\)/\1/; s/^#\(en_US.UTF-8\)/\1/' /etc/locale.gen
```
>Если вам только `en_US.UTF-8 UTF-8` нужен то используйте команду `micro /etc/locale.gen` через поиск найдите `en_US.UTF-8 UTF-8` и раскоментируйте.

Генерируем локали

```
locale-gen
```

Настраиваем время

```
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc
```

Указать имя хоста

```
echo “arch” > /etc/hostname
```

Создания пароля для `root` пользователя
```
passwd
```
>Если вы в начале уже установили пароль для `root` данную функцию не обязательно выполнять <br> также этой командой можно изменять пароль для созданных пользователей `sudo passwd имя пользователя` 
