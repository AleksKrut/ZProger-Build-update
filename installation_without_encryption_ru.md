<!-- install.md -->
<!-- Этот файл можно сохранить как install.md и использовать на GitHub, он сочетает Markdown и HTML для лучшего оформления -->

<div align="center">
  <h1>🚀 Установка Arch Linux + BSPWM (без шифрования)</h1>
  <p>Полная инструкция от разметки диска до готового рабочего стола</p>
</div>

<details>
<summary><strong>📋 Требования</strong></summary>

- Образ Arch Linux (актуальный)  
- Виртуальная машина (VirtualBox) или реальный компьютер с UEFI  
- Установочный носитель (флешка / ISO)  
- Доступ в интернет

</details>

<details>
<summary><strong>⚙️ Настройка VirtualBox (если используется)</strong></summary>

1. **Включите EFI**  
   `Настройки → Система → Материнская плата → Включить EFI`

2. **Отключите загрузку с ISO после установки**  
   После первого запуска уберите образ из `Носители`.

3. **Порядок загрузки**  
   Поставьте жёсткий диск первым в `Система → Порядок загрузки`.

4. **Включите 3D-ускорение**  
   `Дисплей → Включить 3D-ускорение` (для графики).

</details>

<details>
<summary><strong>💾 Загрузка с установочного носителя</strong></summary>

Все команды выполняются от **root**.

</details>

<details>
<summary><strong>🌐 Подключение к Wi‑Fi (при необходимости)</strong></summary>

```bash
iwctl
device list
station <устройство> scan
station <устройство> get-networks
station <устройство> connect <SSID>
exit
ping google.com  # проверка
