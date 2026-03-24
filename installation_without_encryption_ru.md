<div class="container">
    <div align="center">
        <h1>🚀 Установка Proger Build (bspwm) на Arch Linux без шифрования</h1>
        <p>Оригинальное видео: <a href="https://youtu.be/9zewiGf7j-A">YouTube</a><br>
        Репозиторий конфигурации: <a href="https://github.com/Zproger/bspwm-dotfiles">GitHub</a></p>
    </div>
    <details>
        <summary><strong>📌 Перед началом</strong></summary>
        Установите необходимые пакеты и редактор <code>micro</code>.<br>
        Если у вас есть второй компьютер, вы можете подключиться по SSH и вставлять команды.
    </details>
    <details>
        <summary><strong>🖥️ Подготовка к подключению по SSH</strong></summary>
        Рекомендуется использовать <strong>MobaXterm</strong> (<a href="https://mobaxterm.mobatek.net/">скачать</a>).<br>
        На сервере установите <code>openssh</code>:
        <pre><code>pacman -Sy --noconfirm openssh</code></pre>
        Узнайте IP:
        <pre><code>ip address</code></pre>
        Найдите строку <code>enp4s0</code> – там будет серверный IP.<br><br>
        <strong>Настройка MobaXterm:</strong>
        <ul>
            <li><code>Session</code> → <code>SSH</code></li>
            <li><code>Remote host</code>: ваш IP</li>
            <li><code>Username</code>: root</li>
            <li><code>Port</code>: 22</li>
            <li>OK</li>
        </ul>
        Если всё правильно, потребуется ввести пароль root. Задайте его на сервере:
        <pre><code>passwd</code></pre>
        Вводите пароль дважды (символы не отображаются).
    </details>
    <details>
        <summary><strong>📦 Установка пакетов и редактора</strong></summary>
        <pre><code>pacman -Sy micro
pacman -S dhcpcd iwd</code></pre>
        Выход из micro: <code>CTRL+Q</code><br>
        Очистка консоли: <code>clear</code>
    </details>
    <details>
        <summary><strong>💽 Разметка диска (UEFI GPT без шифрования)</strong></summary>
        Просмотр дисков:
        <pre><code>lsblk</code></pre>
        Если используется SSD, разделы будут <code>/dev/nvme0n1p1</code> и <code>/dev/nvme0n1p2</code>.<br>
        В примере диск – <code>/dev/sda</code>.
        <pre><code>parted /dev/sda
mklabel gpt
mkpart ESP fat32 1MiB 512MiB
set 1 boot on
mkpart primary ext4 512MiB 100%
quit</code></pre>
    </details>
    <details>
        <summary><strong>🔧 Форматирование разделов</strong></summary>
        <pre><code>mkfs.fat -F 32 /dev/sda1
mkfs.ext4 /dev/sda2</code></pre>
    </details>
    <details>
        <summary><strong>📂 Монтирование</strong></summary>
        <pre><code>mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot</code></pre>
    </details>
    <details>
        <summary><strong>⚙️ Установка базовых пакетов</strong></summary>
        <pre><code>pacstrap -K /mnt base linux linux-firmware base-devel dhcpcd net-tools iproute2 networkmanager vim micro efibootmgr iwd</code></pre>
    </details>
    <details>
        <summary><strong>📄 Генерация fstab</strong></summary>
        <pre><code>genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab</code></pre>
    </details>
    <details>
        <summary><strong>🔧 Настройка системы (chroot)</strong></summary>
        <pre><code>arch-chroot /mnt</code></pre>
        <strong>Локали:</strong>
        <pre><code>micro /etc/locale.gen</code></pre>
        Раскомментируйте <code>ru_RU.UTF-8 UTF-8</code> и <code>en_US.UTF-8 UTF-8</code>.<br>
        <pre><code>locale-gen
ln -sf /usr/share/zoneinfo/Europe/Kiev /etc/localtime
hwclock --systohc
echo "arch" > /etc/hostname
passwd
useradd -m -G wheel,users,video -s /bin/bash user
passwd user
systemctl enable dhcpcd
systemctl enable iwd.service</code></pre>
        <strong>Настройка mkinitcpio (без шифрования):</strong>
        <pre><code>micro /etc/mkinitcpio.conf</code></pre>
        Убедитесь, что в строке <code>HOOKS</code> нет <code>encrypt</code> и <code>lvm2</code>. Пример:
        <pre><code>HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block filesystems fsck)</code></pre>
        <pre><code>mkinitcpio -P</code></pre>
    
    <details>
        <summary><strong>💿 Установка загрузчика</strong></summary>
        <pre><code>bootctl install
cat > /boot/loader/loader.conf << EOF
timeout 3
default arch
EOF
cat > /boot/loader/entries/arch.conf << EOF
title Arch Linux by ZProger
linux /vmlinuz-linux
initrd /initramfs-linux.img
options root=UUID=$(blkid -s UUID -o value /dev/sda2) rw
EOF</code></pre>
        <pre><code>sudo EDITOR=micro visudo</code></pre>
        Раскомментируйте <code>%wheel ALL=(ALL:ALL) ALL</code>.<br>
        <pre><code>exit
umount -R /mnt
reboot</code></pre>
    </details>
    <details>
        <summary><strong>🔄 После перезагрузки (SSH)</strong></summary>
        <pre><code>sudo systemctl enable sshd
sudo systemctl start sshd</code></pre>
    </details>
    <details>
        <summary><strong>🎨 Установка оболочки BSPWM</strong></summary>
        Авторизуйтесь как пользователь <code>user</code>.<br>
        <pre><code>sudo pacman -Syu
sudo pacman -S xorg bspwm sxhkd xorg-xinit xterm git python3
micro /etc/X11/xinit/xinitrc</code></pre>
        Добавьте в конец файла:
        <pre><code>exec bspwm</code></pre>
        <pre><code>git clone https://github.com/Zproger/bspwm-dotfiles.git
cd bspwm-dotfiles
python3 Builder/install.py</code></pre>
        В меню выберите <code>dotfiles</code>, обновление баз, <code>BASE_PACKAGES</code> и по желанию остальное.<br>
        Запуск:
        <pre><code>startx</code></pre>
    </details>
    <div class="footer">
        📜 Инструкция подготовлена на основе видео ZProger.<br>
        При возникновении проблем смотрите оригинальное видео.
    </div>
</div>
</body>
</html>
