# Мой вариант Ubuntu с LXDE

## Устанавливаем **Ubuntu server**

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
* Software: OpenSSH server

## Ставим дополнительные пакеты

Базовый GUI:
```
sudo apt-get install nano dialog xorg lxde-core
```

Менеджер графической сессии для входа в систему (в lubuntu используется `lightdm`; настройки можно произвести в файле `/etc/lxdm/xldm.conf`):
```
sudo apt-get install lxdm
```

Набор стандартных приложений:
```
sudo apt-get install lxternminal lxinput lxsession-edit lxshortcut lxtask lxappearance lxsession-default-apps menu
```

Менеджер всплывающих уведомлений — ставим `xfce4-notifyd` вместо `notification-daemon`, т. к. удобнее и работает лучше (можно настраивать через `xfce4-notifyd-config`):
```
sudo apt-get install xfce4-notifyd
```

Некоторые дополнительные приложения — блокнот, просмотр изображений, создание скриншотов, работа с архивами:
```
sudo apt-get install leafpad gpicview scrot xarchiver p7zip-full p7zip-rar zip unzip unar unrar-free
```

Конфигурация мониторов — `arandr` вместо `lxrandr`, т. к. даёт больше возможностей:
```
sudo apt-get install arandr
```

Выполнение команды (`Alt+F2`), вместо стандартного, который требует `lxpanel`:
```
sudo apt-get install gmrun
```

## Сеть

Если не нужно настраивать сеть через графический менеджер, то можно оставить как есть и настраивать вручную через `/etc/network/interfaces`.

Устанавливаем менеджер (если не нужно, пакеты pptp можно не ставить, также могут пригодиться другие пакеты, например vpn):
```
sudo apt-get install network-manager network-manager-gnome network-manager-pptp network-manager-pptp-gnome
```

В файле `/etc/network/interfaces` комментируем параметры для текущей сети (`eth0`) — чтобы активным остался только `lo`, т. е. эти две строки:
```
auto lo
iface lo inet loopback
```

## PolicyKit

Для выполнения привилегированных действий требуется настроить PolicyKit:
```
sudo apt-get install lxpolkit
```

Далее запускаем `lxsession-default-apps` и прописываем `Core applications\Polkit agent: lxpolkit`.

В файле `/etc/pam.d/lxdm`, перед
```
session required        pam_limits.so
```
добавить
```
session required        pam_loginuid.so
```
И ещё заменить
```
@include common-session-noninteractive
```
на
```
@include common-session
```

## Управление пакетами

Графические утилиты для установки и обновления пакетов (можно и не ставить и обходиться `apt`):

* `synaptic` — обычная установка пакетов из списка.
* `lubuntu-software-center` — центр приложений Ubuntu.
* `software-properties-gtk` — настройка источников пакетов.
* `ubuntu-extras-keyring` — ключи для источника *extras* (независимых разработчиков).
* `update-manager`, `update-notifier` — автоматический мониторинг обновлений.

```
sudo apt-get install synaptic lubuntu-software-center software-properties-gtk update-manager update-notifier ubuntu-extras-keyring
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
sudo apt-get install xxkb
```

Поместить [.xxkbrc](.xxkbrc) в домашнюю директорию — включено отображение языка флагом, отключено сохранение раскладки для отдельных окон.

Добавить `xxkb` в автозагрузку — создать файл `~/.config/autostart/xxkb-autostart.desktop` (задержка перед запуском нужна, чтобы окружение уже было готово):
```
[Desktop Entry] 
Type=Application
Name=XXKB keyboard layout indicator
Comment=Keyboard state indicator and switcher for xkb
Exec=sh -c "sleep 5s && xxkb"
OnlyShowIn=LXDE;
StartupNotify=false
Terminal=false
Hidden=false
```

Альтернативный вариант — установить `xfce4-xkb-plugin` и получить графическую утилиту для настройки раскладок.

## Синхронизация времени

Если нужно синхронизировать время с интернетом, то нужно поставить NTP-демон:
```
sudo apt-get install ntp
```

Чтобы форсировать ресинхронизацию для случаев, когда компьютер долго не перезагружается, можно добавить перезапуск сервиса в CRON — создаём файл `/etc/cron.weekly/ntp-restart`:
```
#!/bin/sh
service ntp restart > /dev/null 2>&1 &
```

Если нужно перенастроить временную зону, то можно воспользоваться:
```
sudo dpkg-reconfigure tzdata
```

## Звук

Для установки PulseAudio:
```
sudo apt-get install pulseaudio pavucontrol pasystray
```

Для индикатора звука нужно добавить `pasystray` в автозапуск.

Если вместо PulseAudio хочется использовать Alsa, то ставим другие пакеты:
```
sudo apt-get install xfce4-mixer gstreamer0.10-alsa
```

Можно поставить `pnmixer` в качестве индикатора звука для alsa, или воспользоваться стандартным апплетом `xfce4-mixer`.

Проверить звук: `aplay /usr/share/sounds/alsa/Noise.wav`

## Панель рабочего стола

Устанавливаем панель от XFCE4 и удобное меню к нему, т. к. это меню имеет функцию поиска, избранного и удобные настройки:
```
sudo apt-get install xfce4-panel xfce4-whiskermenu-plugin
```

В файле `~/.config/lxsession/LXDE/autostart` заменяем
```
@lxpanel --profile LXDE
```
на
```
@xfce4-panel
```

Whisker Menu Properties:

* Appearance
  * Icon: Location Icons/start-here
  * Show application descriptions: yes
* Behavior
  * Switch categories by hovering: no
  * Include favorites in recently used: yes
  * Display recently used by default: no
  * All Settings: [uncheck]
  * Lock Screen: lxlock
  * Switch Users: [uncheck]
  * Log Out: lxsession-logout
  * Edit Applications: [uncheck]


## Тема оформления

Поместить файл [themerc](themerc) в директорию `~/.themes/OB-Ambiance-Colors/openbox-3/`.

Шрифты, курсоры и тема Ubuntu:
```
sudo apt-get install ttf-ubuntu-font-family dmz-cursor-theme light-themes
```

Для минимизированной полосы прокрутки, как в Ubuntu:
```
sudo apt-get install overlay-scrollbar
```

## Композитный менеджер окон

Можно не ставить, но даёт некоторые дополнительные возможности приложениям для отрисовки.

```
sudo apt-get install compton
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
sudo apt-get update
sudo apt-get install skippy-xd
```

Поместить файл [skippy-xd.rc](skippy-xd.rc) в директорию `~/.config/skippy-xd/`.

## Настройки сессии

### Менеджер сессии

Добавляем в LXDM отчистку сессии после выхода пользователя — редактируем файл `/etc/lxdm/PostLogout`:
```bash
#!/bin/sh

# Kills all your processes when you log out.
killall --user $USER -TERM

# Set's the desktop background to solid black. Useful if you have multiple monitors.
xsetroot -solid black
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
  * `Shift`+`Win`+`F11` или `Win`+`Numpad0` — Переключение состояния «развёрнутый без обрамления».
  * `Win`+`Numpad9` — Разместить в правом верхнем углу.
  * `Win`+`Numpad8` — Разместить сверху.
  * `Win`+`Numpad7` — Разместить в левом верхнем углу.
  * `Win`+`Numpad6` — Разместить справа.
  * `Win`+`Numpad5` — Разместить по центру.
  * `Win`+`Numpad4` — Разместить слева.
  * `Win`+`Numpad3` — Разместить в правом нижнем углу.
  * `Win`+`Numpad2` — Разместить снизу.
  * `Win`+`Numpad1` — Разместить в левом нижнем углу.
* Переключение окон:
  * `Alt`+`Tab` — Следующее окно.
  * `Shift`+`Alt`+`Tab` — Предыдущее окно.
  * `Ctrl`+`Alt`+`Tab` — Следующая панель.
  * `Win`+`Tab` — Обзор всех окон.
* Управление рабочим окружением:
  * `Alt`+`F2` или `Win`+`R` — Выполнить команду.
  * `Ctrl`+`Escape` — Открыть меню приложений.
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
  * Launcher manager: gmrun
  * Screenshot manager: scrot
  * Image viewer: Image Viewer
  * Text editor: Leafpad
  * Archive: Xarchiver
  * Task monitor: Task Manager
* Core applications
  * Window Manager: openbox
  * Polkit agent: lxpolkit
  * Lock screen manager: lxlock
  * Audio manager: pavucontrol или xfce4-mixer (смотря что было установлено)
  * Clipboard manager: lxclipboard
  * Security (keyring): ssh-agent
  * Proxy: build-in
* Autostart
  * Manual
    * @xscreensaver -no-splash
    * @xfce4-panel
    * @pcmanfm --desktop --profile LXDE
    * @/usr/lib/policykit-1-gnome-authentication-agent-1
  * Known Applications
    * Update Notifier
    * Network
    * Keyboard state indicator and switcher for xkb
    * Compton

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

* Display
  * Mode: Deskbar
  * Lock panel: yes
  * Automatically show and hide panel: yes
  * Don't reserve space on borders: yes
  * Row Size (pixels): 30
  * Number of rows: 2
  * Length (%): 100
  * Automatically increase the length: yes
* Appearance (если включён композитинг)
  * Style: Solid color
  * Alpha: 80
  * Color: #8C8C8C (эти три опции задают тёмный фон, который можно использовать с Ubuntu-Mono-Dark, но можно это и не менять и оставить стандартный светлый)
  * Opacity Enter: 100
  * Opacity Leave: 30
* Items
  * Whisker Menu
    * Display: Icon and title
    * Title: Menu
  * Workspace Switcher
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
  * Notification Area
    * Maximum icon size (px): 22
    * Show frame: no
  * Clock

## Для приложений, использующих Qt

Устанавливаем утилиту настройки:
```
sudo apt-get install qt4-qtconfig
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

Утилита настройки меню приложений:
```
sudo apt-get install menulibre
```

Установка локальных пакетов DEB (не из репозитария):
```
sudo apt-get install gdebi
```

Web-браузер (без лишних пакетов):
```
sudo apt-get install firefox --no-install-recommends
```

Просмотр PDF:
```
sudo apt-get install mupdf
```
MuPDF очень быстрый и лёгкий, но он вообще не даёт ни какого интерфейса, так что для продвинутой работы с PDF может пригодиться Evince:
```
sudo apt-get install evince
```

Медиа-проигрыватель. SMPlayer тянет за собой Qt, но с `--no-install-recommends` зависимостей получается немного. Можно было бы поставить `gnome-mplayer`, но он пытается поставить даже более странные вещи. А ещё, вместо всего этого можно поставить `vlc`, но зависимостей у него тоже немало, и Qt он тоже тянет.
```
sudo apt-get install mplayer2
sudo apt-get install smplayer --no-install-recommends
```

Аудио-проигрыватель с каталогизацией (отдельные файлы отлично играет и smplayer, так что если не нужно работать с плейлистами или коллекциями, то можно не ставить):
```
add-apt-repository ppa:starws-box/deadbeef-player
sudo apt-get update
sudo apt-get install deadbeef
```

Для режима сна и других функций управления питанием:
```
sudo apt-get install upower
```

Если нужно полноценное управление питанием, то можно установить:
```
sudo apt-get install xfce4-power-manager
```

Если нужно графическое управление пользователями (хотя вряд ли — это разовая операция):
```
sudo apt-get install gnome-system-tools
```

Таблица символов:
```
sudo apt-get install gucharmap --no-install-recommends
```

Калькулятор (использует Qt, но других зависимостей нет, заметно удобнее и функциональней, чем `galculator`):
```
sudo apt-get install speedcrunch
```
