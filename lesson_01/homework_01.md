# Linux FHS lab

Отчёт по занятию 3 «Файловая система Linux» (ОП.05, Операционные системы и среды).

## System

Вывод команды `cat /etc/os-release | head -4`:

PRETTY_NAME="Debian GNU/Linux 13 (trixie)"
NAME="Debian GNU/Linux"
VERSION_ID="13"
VERSION="13 (trixie)"

## Root filesystem

Вывод команды `findmnt / -o TARGET,SOURCE,FSTYPE`:

TARGET  SOURCE      FSTYPE
/       /dev/sda1   ext4

---

## 01. Корневой каталог

![root](./screenshots/01-root.png)

Что я проверяла: содержимое корневого каталога `/` и разницу между абсолютным и относительным путём.

Команды:

ls -l /
pwd
cat /etc/hostname
cat ./config/app.conf

Результат: `/` — единственная точка начала единого дерева каталогов. Абсолютный путь начинается с `/` (например `/etc/hostname`), относительный — от текущего каталога (например `./config/app.conf`).

---

## 02. Домашний каталог

![home](./screenshots/02-home.png)

Что я проверяла: где находится домашний каталог и как отображаются скрытые файлы.

Команды:

cd ~
pwd
ls -la

Результат: домашний каталог пользователя — `/root` (для root). Скрытые файлы начинаются с точки и видны только при `ls -a`.

---

## 03. Каталог /etc и учётная запись

![system](./screenshots/03-system.png)

Что я проверяла: сведения о системе и о пользователе.

Команды:

cat /etc/os-release
id
echo $HOME
echo $SHELL

Результат: Debian, UID=0 (root), домашний каталог `/root`, оболочка `/bin/bash`.

---

## 04. Программы и PATH

![programs](./screenshots/04-programs.png)

Что я проверяла: как оболочка находит программы.

Команды:

echo $PATH
which ls
type ls
type -a echo

Результат: оболочка ищет исполняемые файлы в каталогах из переменной `PATH` по порядку. `which` показывает путь первого найденного, `type` — как именно оболочка понимает команду (встроенная, алиас, файл).

---

## 05. Изменяемые данные

![var](./screenshots/05-var.png)

Что я проверяла: назначение каталогов `/var/lib`, `/var/log`, `/var/cache` и `/run`.

Команды:

ls -l /var/lib /var/log /var/cache /run

Результат:

- `/var/lib` — постоянные изменяемые данные служб;
- `/var/log` — журналы;
- `/var/cache` — кэш приложений;
- `/run` — сведения о работающих службах, до перезагрузки.

---

## 06. Устройства

![dev](./screenshots/06-dev.png)

Что я проверяла: файлы устройств и код завершения команды.

Команды:

ls -l /dev/sd* /dev/tty* /dev/null
ls /dev/null
echo $?

Результат: в `ls -l` первый символ обозначает тип: `b` — блочное устройство (диски), `c` — символьное (терминалы, `/dev/null`). Команда завершилась с кодом `0` — успех.

---

## 07. Файлы /proc

![proc](./screenshots/07-proc.png)

Что я проверяла: `/proc` — виртуальная файловая система ядра.

Команды:

cat /proc/cpuinfo | head
cat /proc/meminfo | head
cat /proc/uptime
cp /proc/uptime uptime.bak
sleep 3
cat uptime.bak
cat /proc/uptime

Результат: `/proc/uptime` при каждом чтении показывает новое значение (время работы системы), а `uptime.bak` — снимок на момент копирования. Поэтому копия не меняется, а оригинал обновляется.

---

## 08. Сетевые интерфейсы

![sys](./screenshots/08-sys.png)

Что я проверяла: `/sys` — виртуальная ФС с информацией об устройствах.

Команды:

ls /sys/class/net/
ip a

Результат: в системе есть интерфейсы `lo` (петля) и `enp0s3` (основной сетевой адаптер VirtualBox). Список в `/sys/class/net` совпадает с выводом `ip a`.

---

## 09. inode и ссылки

![links](./screenshots/09-links.png)

Что я проверяла: разницу между жёсткой и символической ссылкой.

Команды:

echo hi > a.txt
ln a.txt a_hard.txt
ln -s a.txt a_sym.txt
ls -li
stat a.txt a_hard.txt a_sym.txt

Результат: у `a.txt` и `a_hard.txt` одинаковый inode — это два имени одного и того же файла. У `a_sym.txt` отдельный inode, а содержимое — путь на `a.txt`. Жёсткая ссылка живёт, пока есть хотя бы одно имя; символическая ломается, если удалить цель.

---

## 10. Место на диске

![space](./screenshots/10-space.png)

Что я проверяла: разницу между `df` и `du`.

Команды:

df -h
du -sh ~/linux-fhs-lab

Результат: `df` показывает занятое место на файловых системах (по разделам), а `du` — суммарный размер конкретного каталога.

---

## 11. Монтирование tmpfs

![mount-before](./screenshots/11-mount-before.png)

Что я проверяла: как точка монтирования перекрывает содержимое каталога.

Команды:

mkdir -p /mnt/tmpfs-lab
printf "hello\n" > /mnt/tmpfs-lab/before.txt
ls -l /mnt/tmpfs-lab
mount -t tmpfs tmpfs /mnt/tmpfs-lab
ls -l /mnt/tmpfs-lab

Результат: до монтирования виден `before.txt`. После `mount` содержимое каталога пусто — tmpfs перекрыл старый каталог, `before.txt` «спрятался» под точкой монтирования.

---

## 12. Отключение tmpfs

![mount-after](./screenshots/12-mount-after.png)

Что я проверяла: что происходит с файлами в tmpfs после отмонтирования.

Команды:

touch /mnt/tmpfs-lab/inside.txt
ls -l /mnt/tmpfs-lab
umount /mnt/tmpfs-lab
ls -l /mnt/tmpfs-lab

Результат: `inside.txt` существовал только в tmpfs (в ОЗУ) и после `umount` исчез. Старый `before.txt` снова стал виден — он никуда не девался, просто был перекрыт.

---

## 13. Файлы shop-api

![model](./screenshots/13-model.png)

Что я проверяла: размещение файлов приложения по правилам FHS.

Команды:

mkdir -p model-root/{usr/local/bin,etc/shop-api,var/lib/shop-api,var/log/shop-api,run/shop-api}
cp config/app.conf model-root/etc/shop-api/
cp data/info.txt model-root/var/lib/shop-api/
printf "service started\n" > model-root/var/log/shop-api/app.log
find model-root -type f

Результат: файлы службы распределены по каталогам согласно типу данных:

| Файл | Каталог | Основание |
|---|---|---|
| исполняемый файл | `/usr/local/bin` | программы вне пакетов дистрибутива |
| настройки | `/etc/shop-api` | конфигурационные файлы |
| база данных | `/var/lib/shop-api` | постоянные изменяемые данные |
| журнал | `/var/log/shop-api` | журналы служб |
| номер процесса | `/run/shop-api` | сведения о работающей службе, до перезагрузки |
| промежуточные файлы | `/tmp` | временные данные |

---

## Итоговая задача: пути службы shop-api

| Путь из задания | Оставить / Заменить | Правильный путь | Основание |
|---|---|---|---|
| `/home/admin/shop.log` | Заменить | `/var/log/shop-api/app.log` | Журналы служб — в `/var/log`, а не в домашнем каталоге (часть 5). |
| `/usr/bin/config.json` | Заменить | `/etc/shop-api/config.json` | Конфигурационные файлы — в `/etc` (часть 4). |
| `/tmp/database.db` | Заменить | `/var/lib/shop-api/database.db` | Постоянные изменяемые данные — в `/var/lib`; `/tmp` очищается (часть 5). |
| `/etc/shop-api/app.conf` | Оставить | — | Конфиги службы в `/etc/<служба>` (часть 4). |
| `/var/log/shop-api/access.log` | Оставить | — | Журналы службы в `/var/log/<служба>` (часть 5). |
| `/var/lib/shop-api/database.db` | Оставить | — | Постоянные данные в `/var/lib/<служба>` (часть 5). |

---

## Домашнее задание: служба backup-agent

Таблица каталогов для службы резервного копирования `backup-agent` по правилам FHS.

| Артефакт | Каталог | Основание |
|---|---|---|
| Исполняемый файл программы | `/usr/local/bin/backup-agent` | Программы вне пакетов дистрибутива устанавливаются в `/usr/local/bin`. |
| Настройки | `/etc/backup-agent/` | Конфигурационные файлы служб — в `/etc/<служба>`. |
| Журнал | `/var/log/backup-agent/` | Журналы служб — в `/var/log/<служба>`. |
| Список сделанных копий (метаданные/индекс) | `/var/lib/backup-agent/` | Постоянные изменяемые данные службы — в `/var/lib/<служба>`. |
| Номер процесса (PID-файл) | `/run/backup-agent/` | Сведения о работающей службе — в `/run`, до перезагрузки. |
| Сами резервные копии | `/mnt/backup/` (постоянный носитель) | Резервные копии хранятся на отдельном смонтированном постоянном носителе; `/tmp` и `/run` для этого не подходят, а `/var` — каталог самой системы. |

Обоснование:

- Программа — `/usr/local/bin`, потому что это не пакет дистрибутива.
- Настройки — `/etc/backup-agent`, чтобы при обновлении программы их не затереть.
- Журнал — `/var/log/backup-agent`, стандартное место для журналов служб.
- Список копий — `/var/lib/backup-agent`, это изменяемые, но постоянные данные службы.
- PID-файл — `/run/backup-agent`, нужен только во время работы; `/run` очищается при перезагрузке.
- Резервные копии — на отдельном носителе (`/mnt/backup`), потому что `/tmp` может быть очищен в любой момент, `/run` — только про работающую службу, `/var` — каталог системы, а не внешнее хранилище.

---

## Результат практики-теста

Ссылка: https://algorthimization-course-vvodnoe.vercel.app/op05-operacionnye-sistemy-i-sredy/testy/tema-03/index.html

Код результата: OS03.eyJ2IjoxLCJuIjoi0J_QtdGA0YTQuNC70YzQtdCy0LAg0JDRgNC40L3QsCIsImciOiLQoNCf0J4xIiwiYSI6WyI1MDc4ODYyMyIsIjAxNjQzIiwiMDEyMzQ1IiwiMTExMDEwMDAiLCIwMDAwMTExMTEiLCIwMjM1Nzg5IiwiNzY1MDI3NjAiLCIwIiwiMCIsIjAiLCIwMTIiLCIwIiwiMDEiLCIwMTIzNDUwMSIsIjAxMjMiLCIwIiwiMCIsIjAxMjMiLCIxMTAwMDExIiwiMDEyMzQiLCIzNDU2IiwiMjQiLCIzNDYiLCIwIiwiMDEyMzQ1IiwiNSIsIjAxMjM0NSIsIjAiLCIwMTc4IiwiMDIiLCIwMzQiLCIwMzQiLCIwMThiY2RlIiwiNSIsIjE1Njg5IiwiMDEiLCI0MDY1MjMiLCIyMjMxMTAwMCIsIjAiLCIwMTIzNDUiXSwicyI6MzQsInQiOjQwLCJtIjo1LCJkIjoiMjAyNi0xMC0xM1QxNjo1NyIsInUiOjI1LCJvIjowLCJ4IjoxfQ.d9f796b2da