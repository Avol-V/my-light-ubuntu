# Мой вариант Ubuntu с LXDE

## Устанавливаем **Ubuntu Server 16.04**

Я решил взять серверную версию за основу — она уже достаточно полно настроена, но в ней нет GUI, что нам и требуется.

* Язык установщика: `English`
* Запускаем установщик: `Install Ubuntu Server`

Проходим процесс установки (Английский язык системы и локаль, Московское время, переключение раскладки по Caps Lock, ставим дополнительно SSH-сервер, чтобы можно было подключаться удалённо).

* Language: English
* Location: other/Europe/Russian Federation
* Locale: en_US.UTF-8
* Keyboard: No/Russian/Russian/Caps Lock
* Encrypt: No
* Timezone (Europe/Moscow): Yes
* Software: Standard system utilities, OpenSSH server

## Ставим дополнительные пакеты

Базовый GUI:
```
sudo apt install nano dialog xorg
sudo apt install gksu lxterminal xscreensaver lxde-core lxde-common lxsession --no-install-recommends
```

Менеджер графической сессии для входа в систему (в lubuntu используется `lightdm`; настройки можно произвести в файле `/etc/lxdm/xldm.conf`):
```
sudo apt install lxdm
```

Набор стандартных приложений:
```
sudo apt install lxinput lxsession-edit lxshortcut lxtask lxappearance lxsession-default-apps menu  --no-install-recommends
```

Менеджер всплывающих уведомлений — ставим `xfce4-notifyd` вместо `notification-daemon`, т. к. удобнее и работает лучше (можно настраивать через `xfce4-notifyd-config`):
```
sudo apt install xfce4-notifyd
```

Некоторые дополнительные приложения — блокнот, просмотр изображений, создание скриншотов, работа с буфером обмена, работа с архивами:
```
sudo apt install leafpad gpicview scrot xclip xarchiver p7zip-full p7zip-rar zip unzip unar unrar-free
```

Конфигурация мониторов — `arandr` вместо `lxrandr`, т. к. даёт больше возможностей:
```
sudo apt install arandr
```

Выполнение команды (`Alt+F2`), вместо стандартного, который требует `lxpanel`:
```
sudo apt install gmrun
```

## Сеть

Если не нужно настраивать сеть через графический менеджер, то можно оставить как есть и настраивать вручную через `/etc/network/interfaces`.

Устанавливаем менеджер:
```
sudo apt install network-manager network-manager-gnome --no-install-recommends
```

Чтобы работал запрос пароля:
```
sudo apt install gnome-keyring
```

Если нужно PPTP:
```
sudo apt install network-manager-pptp network-manager-pptp-gnome --no-install-recommends
```
Аналогично для VPN.

В файле `/etc/network/interfaces` комментируем параметры для текущей сети (`eth0`, или другие) — чтобы активным остался только `lo`, т. е. эти две строки:
```
auto lo
iface lo inet loopback
```

## PolicyKit

Для выполнения привилегированных действий требуется настроить PolicyKit:
```
sudo apt install lxpolkit
```

Он должен запускаться с системой, проверить что в `lxsession-default-apps` присутствует `Core applications\Polkit agent: lxpolkit`.

## Управление пакетами

Графические утилиты для установки и обновления пакетов (можно и не ставить и обходиться `apt`):

* `synaptic` — графическая оболочка для установки пакетов.
* `update-notifier` — автоматический мониторинг обновлений (устанавливает и некоторые сопутствующие приложения).

```
sudo apt-get install synaptic update-notifier --no-install-recommends
```

## Раскладка клавиатуры

Настраиваем раскладку, файл `/etc/default/keyboard`:
```
XKBMODEL="pc105"
XKBLAYOUT="us,ru"
XKBVARIANT=","
XKBOPTIONS="grp:caps_toggle,grp_led:caps,compose:rwin,lv3:ralt_switch,nbsp:level3n,misc:typo,terminate:ctrl_alt_bksp"
```

Индикатор раскладки:
```
sudo apt install xxkb
```

Поместить [.xxkbrc](.xxkbrc) в домашнюю директорию — включено отображение языка флагом, отключено сохранение раскладки для отдельных окон.

Добавить `xxkb` в автозагрузку — создать файл `~/.config/autostart/xxkb-autostart.desktop` (задержка перед запуском нужна, чтобы окружение уже было готово):
```
[Desktop Entry] 
Type=Application
Name=XXKB keyboard layout indicator
Comment=Keyboard state indicator and switcher for xkb
Exec=sh -c "sleep 30s && xxkb"
OnlyShowIn=LXDE;
StartupNotify=false
Terminal=false
Hidden=false
```

Альтернативный вариант — установить `xfce4-xkb-plugin` и получить графическую утилиту для настройки раскладок.

## Синхронизация времени

Если нужно синхронизировать время с интернетом, то нужно поставить NTP-демон:
```
sudo apt install ntp
```

Чтобы форсировать ресинхронизацию для случаев, когда компьютер долго не перезагружается, можно добавить перезапуск сервиса в CRON — создаём файл `/etc/cron.weekly/ntp-restart`:
```
#!/bin/sh
service ntp stop > /dev/null 2>&1 &
ntpdate -s ntp21.vniiftri.ru
service ntp start > /dev/null 2>&1 &
```

Если нужно перенастроить временную зону, то можно воспользоваться:
```
sudo dpkg-reconfigure tzdata
```

## Звук

Для установки PulseAudio:
```
sudo apt install pulseaudio pavucontrol xfce4-pulseaudio-plugin
```

Даже для PulseAudio стоит поставить набор утилит Alsa (управление звуком, воспроизведение с консоли):
```
sudo apt install alsa-utils
```

Если вместо PulseAudio хочется использовать Alsa, то ставим другие пакеты:
```
sudo apt install gstreamer0.10-alsa pnmixer
```

Проверить звук: `aplay /usr/share/sounds/alsa/Noise.wav`

## Панель рабочего стола

Устанавливаем панель от XFCE4 и удобное меню к нему, т. к. это меню имеет функцию поиска, избранного и удобные настройки:
```
sudo apt install xfce4-panel xfce4-whiskermenu-plugin
```

В файле `~/.config/lxsession/LXDE/autostart` заменяем
```
@lxpanel --profile LXDE
```
на
```
@xfce4-panel
```

## Тема оформления

Поместить файл [themerc](themerc) в директорию `~/.themes/OB-Ambiance-Colors/openbox-3/`.

Шрифты, курсоры и тема Ubuntu:
```
sudo apt install ttf-ubuntu-font-family dmz-cursor-theme light-themes
```

Для минимизированной полосы прокрутки, как в Ubuntu:
```
sudo apt install overlay-scrollbar overlay-scrollbar-gtk2
```

Дополнительные темы GTK — Clearlooks нужна для тема по умолчанию у LXDM:
```
sudo apt install gtk2-engines
```

## Композитный менеджер окон

Лучше не ставить — и без него всё хорошо, но он даёт некоторые дополнительные возможности приложениям для отрисовки.

```
sudo apt install compton
```

Поместить файл [.compton.conf](.compton.conf) в домашнюю директорию.

Добавить в автозагрузку — создать файл `~/.config/autostart/compton.desktop`:
```
[Desktop Entry] 
Type=Application
Name=Compton
Comment=X11 compositor
Exec=compton -b
OnlyShowIn=LXDE;
StartupNotify=false
Terminal=false
Hidden=false
```

Для оптимизации настройки стоит ознакомиться с https://github.com/chjj/compton/wiki/perf-guide — может потребоваться подстройка под конкретные драйвера видеокарты.

## Утилита для обзора всех окон

Необязательна, но лёгкая, постоянно не висит и может быть полезна.

```
sudo add-apt-repository ppa:landronimirc/skippy-xd-daily
sudo apt update
sudo apt install skippy-xd
```

Поместить файл [skippy-xd.rc](skippy-xd.rc) в директорию `~/.config/skippy-xd/`.

## Настройки сессии

### Менеджер сессии

Включаем в LXDM отчистку запущенных приложений после выхода пользователя — в файле `/etc/lxdm/PostLogout` раскомментировать:
```bash
ps --user $USER | awk 'NR > 1 {print $1}' | xargs -t kill
```

### Менеджер окон

Поместить файл [lxde-rc.xml](lxde-rc.xml) в директорию `~/.config/openbox/`.
Применить настройки без перезагрузки можно командой `openbox --reconfigure`.

Используемые горячие клавиши:

* Переключение рабочих столов:
  * `Win`+`Left` — Переключиться на рабочий стол слева.
  * `Win`+`Right` — Переключиться на рабочий стол справа.
  * `Win`+`Up` — Переключиться на рабочий стол выше.
  * `Win`+`Down` — Переключиться на рабочий стол ниже.
  * `Shift`+`Win`+`Left` — Переместить окно на рабочий стол слева.
  * `Shift`+`Win`+`Right` — Переместить окно на рабочий стол справа.
  * `Shift`+`Win`+`Up` — Переместить окно на рабочий стол сверху.
  * `Shift`+`Win`+`Down` — Переместить окно на рабочий стол снизу.
  * `Win`+`F1` — Переключиться на рабочий стол 1.
  * `Win`+`F2` — Переключиться на рабочий стол 2.
  * `Win`+`F3` — Переключиться на рабочий стол 3.
  * `Win`+`F4` — Переключиться на рабочий стол 4.
  * `Win`+`D` — Показать рабочий стол.
* Управление окнами:
  * `Alt`+`F4` — Закрыть.
  * `Alt`+`Escape` — Переключиться на окно ниже.
  * `Alt`+`Пробел` — Открыть меню управления окном.
  * `Win`+`F11` — Переключение полноэкранного режима.
  * `Win`+`Numpad0` или `Alt`+`Win`+`F11` — Включение состояния «развёрнутый без обрамления».
  * `Ctrl`+`Win`+`Numpad0` — Развернуть без обрамления, используя все мониторы.
  * `Win`+`Numpad /` или `Alt`+`Win`+`\` — Переключить обрамление.
  * `Win`+`Numpad *` — Переместить в центр.
  * `Win`+`Numpad +` или `Alt`+`Win`+`=` — Переместить на следующий монитор.
  * `Win`+`Numpad -` или `Alt`+`Win`+`-` — Переместить на предыдущий монитор.
  * `Win`+`Numpad9` или `Alt`+`Win`+`]` — Разместить в правом верхнем углу.
  * `Win`+`Numpad8` или `Alt`+`Win`+`[` — Разместить сверху.
  * `Win`+`Numpad7` или `Alt`+`Win`+`p` — Разместить в левом верхнем углу.
  * `Win`+`Numpad6` или `Alt`+`Win`+`'` — Разместить справа.
  * `Win`+`Numpad5` или `Alt`+`Win`+`;` — Разместить по центру.
  * `Win`+`Numpad4` или `Alt`+`Win`+`l` — Разместить слева.
  * `Win`+`Numpad3` или `Alt`+`Win`+`/` — Разместить в правом нижнем углу.
  * `Win`+`Numpad2` или `Alt`+`Win`+`.` — Разместить снизу.
  * `Win`+`Numpad1` или `Alt`+`Win`+`,` — Разместить в левом нижнем углу.
* Переключение окон:
  * `Alt`+`Tab` — Следующее окно.
  * `Shift`+`Alt`+`Tab` — Предыдущее окно.
  * `Ctrl`+`Alt`+`Tab` — Следующая панель.
  * `Win`+`Tab` — Обзор всех окон.
* Управление рабочим окружением:
  * `Alt`+`F2` или `Win`+`R` — Выполнить команду.
  * `Ctrl`+`Escape` или `Win`+`Пробел` — Открыть меню приложений.
  * `Ctrl`+`Alt`+`L` или `Win`+`L` — Заблокировать экран.
  * `Ctrl`+`Win`+`X` — Режим завершения работы окон по нажатию курсором.
* Запуск приложений:
  * `Crtl`+`Alt`+`D` или `Win`+`E` — Запустить менеджер файлов.
  * `Ctrl`+`Alt`+`Delete` — Запустить менеджер задач.
  * `Ctrl`+`Alt`+`T` — Запустить эмулятор терминала.
  * `PrintScreen` — Снимок экрана.
  * `Alt`+`PrintScreen` — Снимок выбранного окна или области.
* Медиа-клавиши:
  * Увеличить громкость.
  * Уменьшить громкость.
  * Включить/выключить звук.
  * Запустить web-браузер.
  * Запустить калькулятор.
  * Запустить менеджер файлов.
  * Запустить эмулятор терминала.

Управление мышью:

* В любой области окна:
  * `Win`+`Левая кнопка` — Перемещение окна.
  * `Win`+`Правая кнопка` — Изменение размеров окна.
  * `Win`+`Средняя кнопка` — Переключиться на окно ниже.
* Заголовок окна:
  * Двойной клик левой кнопки — Переключение развёрнутого состояния.
  * Нажатие правой кнопкой — Открыть меню управления окном.
* Кнопка «Развернуть окно»:
  * Нажатие левой кнопки — Переключение развёрнутого состояния.
  * Нажатие средней кнопки — Переключение развёрнутого состояния по вертикали.
  * Нажатие правой кнопки — Переключение развёрнутого состояния по горизонтали.
* Рабочий стол:
  * Колесо мыши вверх — Предыдущий рабочий стол.
  * Колесо мыши вниз — Следующий рабочий стол.

### Тема оформления

В утилите `lxappearance`:

* Widget
  * Theme: Ambiance
  * Default font: Ubuntu, Regular, 11
* Icon Theme: Ubuntu-Mono-Light (можно пробовать и Ubuntu-Mono-Dark, но есть проблемы с тем, чтобы сделать панель тёмной)
* Mouse Cursor: DMZ (White)

В утилите `lxsession-default-apps` проверить настройки:

* Launching applications
  * File Namager: File Manager PCManFM
  * Terminal manager: LXTerminal
  * Webbrowser: Firefox Web Browser
  * Launcher manager: gmrun
  * Screenshot manager: scrot
  * PDF Reader: MuPDF
  * Video player: SMPlayer
  * Audio player: SMPlayer
  * Image viewer: Image Viewer
  * Text editor: Leafpad
  * Archive: Xarchiver
  * Task monitor: Task Manager
* Core applications
  * Window Manager: openbox
  * Polkit agent: lxpolkit
  * Lock screen manager: lxlock
  * Quit manager: lxsession-logout
  * Clipboard manager: lxclipboard
  * Security (keyring): ssh-agent
  * Proxy: build-in
* Autostart
  * Manual
    * @xfce4-panel
    * @pcmanfm --desktop --profile LXDE
    * @xscreensaver -no-splash
  * Known Applications
    * Power Manager
    * Update Notifier
    * PulseAudio Sound System
    * Network
    * XXKB keyboard layout indicator

Настройки `lxterminal`:

* Style
  * Terminal font: Ubuntu Mono, Regular, 12
  * Background: #2B2B2B
  * Foreground: #EEEEEC
  * Allow bold font: yes
  * Cursor blink: yes
  * Cursor style: Block
  * Audible bell: yes
* Advanced
  * Disable menu shortcut key (F10 by default): yes
  * Disable using Alt-n for tabs and menu: yes

Вертикальная панель (Panel Preferences):

Поместить файл [.gtkrc-2.0.mine](.gtkrc-2.0.mine) в домашнюю директорию.
В нём заданы дополнительные настройки автоскрытия панели — задержка перед открытием (500 мс), перед скрытием (300 мс) и видимая часть после скрытия (1 пиксел, 0 задать нельзя).
Изменения вступают в силу после logout'а или перезагрузки.

* Display
  * Mode: Deskbar
  * Lock panel: yes
  * Automatically show and hide panel: Intelligently
  * Don't reserve space on borders: yes
  * Row Size (pixels): 30
  * Number of rows: 1
  * Length (%): 100
  * Automatically increase the length: yes
* Appearance
  * Style: None
* Items
  * Whisker Menu
    * Appearance
      * Display: Icon
      * Title: Menu
      * Icon: Location Icons/start-here
      * Use a single panel row: no
      * Show generic application names: no
      * Show application descriptions: yes
      * Show menu hierarchy: no
      * Item icons size: Small
      * Category icon size: Smaller
      * Background opacity: 100
    * Behavior
      * Switch categories by hovering: no
      * Position search entry next to panel button: no
      * Position categories next to panel button: no
      * Amount of items: 10
      * Ignore favorites: no
      * Display by default: no
    * Commands
      * All Settings: [uncheck]
      * Lock Screen: lxlock
      * Switch Users: [uncheck]
      * Log Out: lxsession-logout
      * Edit Applications: menulibre
  * Workspace Switcher
    * Number of rows: 2
    * Show miniature view: yes
    * Switch workspaces using the mouse wheel: yes
  * Window Buttons
    * Show button labels: no
    * Show flat buttons: yes
    * Show handle: no
    * Sorting order: Group title and timestamp
    * Window grouping: Never
    * Middle click action: Nothing
    * Restore minimized windows to current workspace: no
    * Draw window frame when hovering a button: no (или yes, т. к. может быть удобно — подсвечивает, где находится соответствующее приложению окно)
    * Switch windows using the mouse wheel: yes
    * Show windows from all workspaces or viewports: no
    * Show only minimized windows: no
    * Show windows from all monitors: yes
  * Separator
    * Style: Transparent
    * Expand: yes
  * PulseAudio Plugin
    * Enable keyboard shortcuts for volume control: yes
    * Show notifications when volume changes: yes
    * Audio Mixer: pavucontrol
  * Notification Area
    * Maximum icon size (px): 22
    * Show frame: no
  * Power Manager Plugin
  * Clock
    * Layout: Digital
    * Clock Options/Format: %H%n%M

## Для приложений, использующих Qt

Устанавливаем утилиту настройки:
```
sudo apt install qt4-qtconfig --no-install-recommends
```

В настройках `qtconfig-qt4`:

* Appearance
  * Select GUI Style: GTK+
  * Button Background: 242,241,240
  * Window Background: 242,241,240
* Fonts
  * Family: Ubuntu
  * Style: Medium
  * Point Size: 11

Сохраняем (File/Save).

## Дополнительные приложения

### Система

Утилита настройки меню приложений:
```
sudo apt install menulibre
```

Установка локальных пакетов DEB (не из репозитария):
```
sudo apt install gdebi --no-install-recommends
```

Если нужно графическое управление пользователями (хотя вряд ли — это разовая операция):
```
sudo apt install gnome-system-tools
```

Монтирование через SSH (+ графический запрос пароля):
```
sudo apt install sshfs
sudo apt install ssh-askpass-gnome ssh-askpass
```

Доступ к ftp, sftp, smb через менеджер файлов:
```
sudo apt install gvfs-fuse gvfs-backends --no-install-recommends
```

Драйвера nvidia нужно обязательно ставить с `--no-install-recommends`, иначе они потянут за собой половину окружения Gnome:
```
sudo apt install nvidia-352 --no-install-recommends
sudo apt install nvidia-settings --no-install-recommends
```

Node.js (название дистрибутива можно узнать через `lsb_release -c -s`, здесь — `xenial`):
```
curl -s https://deb.nodesource.com/gpgkey/nodesource.gpg.key | sudo apt-key add -
sudo sh -c "echo '# NodeSource Node.js 6.x repo (https://github.com/nodesource/distributions)' > /etc/apt/sources.list.d/nodesource.list"
sudo sh -c "echo 'deb https://deb.nodesource.com/node_6.x xenial main' >> /etc/apt/sources.list.d/nodesource.list"
sudo sh -c "echo 'deb-src https://deb.nodesource.com/node_6.x xenial main' >> /etc/apt/sources.list.d/nodesource.list"
sudo apt update
sudo apt install nodejs
```

### Управление электропитанием

Для режима сна и других функций управления питанием:
```
sudo apt install upower
```

Если нужно полноценное управление питанием, то можно установить виджет для панели (управление яркостью экрана, отключение экрана, переход в режим сна):
```
sudo apt install xfce4-power-manager
```

Дополнительные возможности по энергосбережению (переключение режимов для различных устройств и служб в зависимости от источника питания — сеть или батарея):
```
sudo apt install tlp
```

Просмотр состояния энергопотребления:
```
sudo apt install powertop
```

### Офис

Web-браузер (без лишних пакетов):
```
sudo apt install firefox --no-install-recommends
```
Ставим в Firefox «назад» по Backspace:
```
about:config
browser.backspace_action = 0
```

Альтернативный web-браузер (Chromium):
```
sudo apt install chromium-browser
```

Просмотр PDF:
```
sudo apt install mupdf
```
MuPDF очень быстрый и лёгкий, но он вообще не даёт ни какого интерфейса, так что для продвинутой работы с PDF может пригодиться Evince:
```
sudo apt install evince
```

Таблица символов:
```
sudo apt install gucharmap --no-install-recommends
```

Калькулятор (использует Qt, но других зависимостей нет, заметно удобнее и функциональней, чем `galculator`):
```
sudo apt install speedcrunch
```

Шрифты (Специальные символы, расширенный Unicode, замена times new roman + arial + courier new):
```
sudo apt install fonts-stix fonts-lyx
sudo apt install unifont ttf-ancient-fonts
sudo apt install ttf-liberation
```

Офисный пакет:
```
sudo apt install libreoffice --no-install-recommends
sudo apt install libreoffice-gtk libreoffice-style-human
```
В меню Tools/Options/LibreOffice/View «Icons size and style» поставить в «Small Human».  
Установить словарь http://extensions.libreoffice.org/extension-center/russian-dictionary-pack для проверки правописания.

Редактор кода:
```
sudo apt install geany
```
Тёмная тема оформления: поместить https://github.com/geany/geany-themes/blob/master/colorschemes/darcula.conf в `~/.config/geany/colorschemes/`.

### Мультимедиа

Медиа-проигрыватель:
```
sudo apt install smplayer --no-install-recommends
```

Альтернативный медиа-проигрыватель (VLC):
```
sudo apt install vlc
```

Аудио-проигрыватель с каталогизацией (отдельные файлы отлично играет и smplayer, так что если не нужно работать с плейлистами или коллекциями, то можно не ставить):
```
sudo add-apt-repository ppa:starws-box/deadbeef-player
sudo apt update
sudo apt install deadbeef
```

### Графика

Просмотр большого количества форматов изображений, в том числе и PSD:
```
sudo apt install nomacs
```

Просмотр изображений, правка, массовая правка:
```
sudo apt install gthumb --no-install-recommends
```

Графический редактор:
```
sudo apt install gimp --no-install-recommends
```

## Дополнительные настройки

### /tmp в оперативной памяти

В `/etc/fstab`:
```
# /tmp on RAM
tmpfs /tmp tmpfs nosuid,nodev,mode=1777,size=50% 0 0
```

Разрешает использовать до 50% оперативной памяти.  
Выставление `noexec` может вызвать проблемы, т. к. программы могут помещать туда временные скрипты.

Добавляем периодическую отчистку мусора в /tmp — в файле `/etc/cron.daily/tmp-gc`:
```bash
#!/bin/sh
find /tmp/ -type f -atime +3 -size +4k -and -not -exec fuser -s {} ';' -and -exec rm {} ';'
```
