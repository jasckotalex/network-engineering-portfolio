# EVE-NG в VMware Workstation: установка образов Cisco

[English version](README.md)

Следуйте этой инструкции, чтобы запустить EVE-NG Community в VMware Workstation, передать образы виртуальных устройств Cisco через WinSCP и проверить запуск узлов через консоли.

> Бинарные файлы образов Cisco являются проприетарным ПО и не включены в репозиторий. Используйте только образы, на которые у вас есть соответствующие права.

## 1. Подготовка компьютера

Установите на Windows следующие программы:

- VMware Workstation Pro или Player для запуска виртуальной машины EVE-NG.
- WinSCP для передачи файлов на сервер EVE-NG.
- PuTTY для SSH-подключения к серверу.
- EVE-NG Windows Client Pack для связи консоли устройств в браузере с локальными терминальными приложениями и средствами захвата трафика. Загрузите пакет, совместимый с установленной версией EVE-NG, с [официальной страницы EVE-NG](https://www.eve-ng.net/index.php/download/).

Процессор хоста должен поддерживать аппаратную виртуализацию. Виртуализация должна быть включена в BIOS/UEFI и доступна виртуальной машине EVE-NG. В документации EVE-NG указаны VMware Workstation 16 или новее и VT-x/EPT (или соответствующие поддерживаемые функции виртуализации AMD). [Поддерживаемые системы](https://www.eve-ng.net/index.php/supported-hardware-and-software-systems/) · [Установка виртуальной машины](https://www.eve-ng.net/index.php/documentation/installation/virtual-machine-install/)

## 2. Установка или запуск виртуальной машины EVE-NG

1. Загрузите установочный образ EVE-NG Community с [официальной страницы загрузки](https://www.eve-ng.net/index.php/download/).
2. Создайте или импортируйте виртуальную машину EVE-NG в VMware, следуя официальной инструкции. Если EVE-NG уже установлена, переходите к следующему шагу.
3. В настройках VMware проверьте, что сетевой адаптер виртуальной машины подключён и для гостевой системы доступна вложенная аппаратная виртуализация.
4. Запустите виртуальную машину и дождитесь завершения загрузки.
5. Запишите IP-адрес управления, отображаемый в консоли. Он потребуется для подключения через браузер, WinSCP и PuTTY.

![Настройки процессора и виртуализации VMware](assets/01-vmware-cpu-settings.png)
*Настройки виртуальной машины VMware: сетевой адаптер NAT и параметр виртуализации Intel VT-x/EPT.*

## 3. Установка PuTTY и EVE-NG Client Pack

1. Загрузите PuTTY с [официальной страницы](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) и установите пакет Windows, соответствующий архитектуре компьютера.
2. Загрузите Windows Client Pack для установленной версии EVE-NG с [официальной страницы загрузки EVE-NG](https://www.eve-ng.net/index.php/download/) и установите его. На официальной странице указано, что Client Pack V3 совместим с версиями EVE-NG до 6.x.
3. При входе в EVE-NG выберите **Native console** («Локальная консоль»), если появится соответствующий список. Client Pack устанавливает локальные обработчики, которые позволяют браузеру открывать консоли устройств в терминальных программах.

![Настройки SSH-подключения PuTTY](assets/02-putty-ssh-session-settings.png)
*Профиль PuTTY: выберите SSH и порт 22; при подключении укажите IP-адрес управления EVE-NG.*

## 4. Передача QEMU-образов через WinSCP

В этой инструкции используются QEMU-образы в формате QCOW2, а не Cisco IOL-файлы с расширением `.bin`. Создайте соответствующие папки образов на сервере EVE-NG или используйте уже созданные. Диск внутри каждой папки должен называться `virtioa.qcow2`.

1. Откройте WinSCP и создайте подключение:
   - Протокол передачи: **SFTP**
   - Имя хоста: IP-адрес, показанный в консоли виртуальной машины EVE-NG
   - Порт: `22`
   - Имя пользователя и пароль: разрешённые учётные данные сервера EVE-NG
2. Подключитесь и откройте в удалённой панели `/opt/unetlab/addons/qemu/`.
3. Откройте папку нужного устройства и скопируйте туда лицензированный QCOW2-диск, назвав его `virtioa.qcow2`.
4. Используйте следующие пути и имена папок:

   ```text
   /opt/unetlab/addons/qemu/asav-9.5.3-9/virtioa.qcow2
   /opt/unetlab/addons/qemu/vios-router/virtioa.qcow2
   /opt/unetlab/addons/qemu/viosl2-switch/virtioa.qcow2
   ```

   Префикс папки должен соответствовать типу устройства: `asav-`, `vios-` или `viosl2-`. Для конкретного типа образа проверяйте имя диска в [официальной таблице EVE-NG](https://www.eve-ng.net/index.php/documentation/qemu-image-namings/). Для этих трёх образов используется `virtioa.qcow2`.

![Папки образов Cisco в WinSCP](assets/06-winscp-image-folders.png)
*Каталоги образов на сервере EVE-NG.*

![Имя диска образа ASAv](assets/07-asav-image-disk.png)
*Папка ASAv с файлом `virtioa.qcow2`.*

![Имя диска образа vIOS L2](assets/08-viosl2-image-disk.png)
*Папка коммутатора vIOS L2 с файлом `virtioa.qcow2`.*

![Имя диска образа маршрутизатора vIOS](assets/09-vios-router-image-disk.png)
*Папка маршрутизатора vIOS с файлом `virtioa.qcow2`.*

## 5. Настройка прав на образы через PuTTY

1. Откройте PuTTY, введите IP-адрес управления EVE-NG, выберите SSH и порт `22`, затем нажмите **Open** («Открыть»).
2. При первом подключении проверьте и подтвердите ключ сервера.
3. Войдите с разрешённой учётной записью сервера.
4. Выполните команду EVE-NG для настройки прав на образы:

   ```bash
   /opt/unetlab/wrappers/unl_wrapper -a fixpermissions
   ```

Нужно вводить полную команду; одного слова `fixpermissions` недостаточно. В инструкциях EVE-NG эта команда используется после добавления образов. [Инструкция по Cisco vIOS](https://www.eve-ng.net/index.php/documentation/howtos/howto-add-cisco-vios-from-virl/)

## 6. Создание лабораторной и добавление узлов

1. Откройте браузер и перейдите по адресу `http://<IP-адрес-EVE-NG>/`.
2. Войдите в веб-интерфейс и выберите **Native console**, если появится выбор типа консоли.
3. Выберите **Add new lab** («Добавить лабораторную»), укажите имя и сохраните.
4. Откройте лабораторную. Щёлкните правой кнопкой мыши по рабочему полю и выберите **Node** («Узел») либо воспользуйтесь кнопкой добавления узла.
5. Добавьте шаблоны **Cisco ASAv**, **Cisco vIOS Router** и **Cisco vIOS Switch**. Сохраните каждый узел, выбрав соответствующий образ.
6. Запустите узлы и дождитесь завершения загрузки.
7. Щёлкните левой кнопкой мыши по запущенному устройству, чтобы открыть консоль. При установленном EVE-NG Client Pack и выбранном режиме **Native console** должна запуститься настроенная локальная терминальная программа.

В списке шаблонов выберите коммутатор Cisco, маршрутизатор Cisco и ASAv:

![Шаблон Cisco vIOS Switch](assets/10-add-switch-template.png)
![Шаблон Cisco vIOS Router](assets/11-add-router-template.png)
![Шаблон Cisco ASAv](assets/12-add-asav-template.png)

![Три устройства Cisco добавлены в топологию EVE-NG](assets/13-three-node-topology.png)
*Добавьте три узла в топологию лабораторной.*

![Устройства Cisco запущены в EVE-NG](assets/14-nodes-started.png)
*Значки ▶ показывают, что узлы запущены.*

## 7. Проверка результата

При успешном запуске образов в консолях устройств отображаются следующие приглашения командной строки:

| Устройство | Приглашение в консоли |
| --- | --- |
| Cisco ASAv | `ciscoasa>` |
| Маршрутизатор Cisco vIOS | `Router>` |
| L2-коммутатор Cisco vIOS | `Switch>` |

Эти приглашения подтверждают, что EVE-NG распознаёт и запускает три образа.

![Приглашение консоли ASAv](assets/15-asav-console.png)
![Приглашение консоли коммутатора vIOS L2](assets/16-viosl2-console.png)
![Приглашение консоли маршрутизатора vIOS](assets/17-vios-router-console.png)

## Примечание по устранению неполадок с Cisco IOL

Для Cisco IOL-образов `.bin` используется отдельный механизм EVE-NG и требуется корректная конфигурация лицензии IOL. В документации EVE-NG IOL и QEMU-образы рассматриваются как отдельные типы; в рекомендациях Cisco Community настройка лицензии указана как один из шагов диагностики, если узел IOL останавливается при запуске. Также нужно проверять расположение образа и права доступа. При подготовке образов я выполнял команду, связанную с лицензированием, однако точная команда и состояние файла лицензии не зафиксированы, поэтому нельзя достоверно указать единственную причину сбоя. Модель процессора Intel Core i7-13650HX и 64-разрядная архитектура сами по себе не доказывают, что причиной был процессор. В этой инструкции используются QCOW2-образы vIOS/ASAv, которые успешно запускаются в лабораторной среде. ([Руководство EVE-NG Community](https://www.eve-ng.net/wp-content/uploads/2024/04/EVE-CE-BOOK-6.0-2024.pdf) · [Обсуждение диагностики запуска IOL в Cisco Community](https://community.cisco.com/t5/cisco-software-discussions/when-i-start-iol-image-router-it-stops-within-few-seconds-on-eve/td-p/4310879))

## Источники

- [Загрузки EVE-NG, включая Windows Client Pack](https://www.eve-ng.net/index.php/download/)
- [Установка виртуальной машины EVE-NG](https://www.eve-ng.net/index.php/documentation/installation/virtual-machine-install/)
- [Поддерживаемое оборудование и ПО EVE-NG](https://www.eve-ng.net/index.php/supported-hardware-and-software-systems/)
- [Таблица имён QEMU-образов EVE-NG](https://www.eve-ng.net/index.php/documentation/qemu-image-namings/)
- [Инструкция EVE-NG по Cisco vIOS](https://www.eve-ng.net/index.php/documentation/howtos/howto-add-cisco-vios-from-virl/)
- [Официальная загрузка PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
