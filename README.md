# mephi-linux-security-2026

- **Студент:** Григорьев Никита Андреевич
- **Уникальный номер:** `374997`
- **ОС:** `Fedora 45`

## Как выполнено каждое задание

| Раздел | Задание | Как выполнено | Артефакт |
|---|---|---|---|
| 1 | Пользователь user1, UID 1234, группа students | `groupadd students`; `useradd -m -u 1234 -G students -s /bin/bash user1`; `passwd user1` | `passwd`, `group`, `shadow` |
| 1 | Смена пароля каждые 3 месяца | `chage -M 90 user1` (поле max = 90 в `/etc/shadow`) | `shadow` |
| 2.1 | Поиск файлов с set-UID | `find / -path /proc -prune -o -type f -perm -4000 -exec ls -l {} +` | `suid_files.out` |
| 2.2 | Процессы с EUID=0 и обычным RUID | В одном терминале user1 запускает `passwd`, в другом: `ps -eo pid,ruid,euid,ruser,euser,args` с фильтром `awk '$3==0 && $2>=1000'` | `ps_privileged.out` |
| 3 | Привилегированная операция через set-UID | Чтение `/etc/shadow` (права 000; `/etc/passwd` не подходит, он и так читается всеми). `cp /usr/bin/cat /home/user1/cat`, `chown root:root`, `chmod 4755`. От user1 `/home/user1/cat /etc/shadow` выполняется успешно | `stat.out` |
| 4 | Привилегированная операция через capabilities | Смена владельца файла на произвольного пользователя. `cp /usr/bin/chown /home/user1/chown`, `setcap cap_chown+ep`. От user1 `/home/user1/chown root:root test.txt` выполняется успешно, без set-UID | `getcap.out`, `stat.out` |
| 5 | sudo: user1 может менять системное время | В `/etc/sudoers`: `user1 ALL=(root) /usr/bin/date -s *, /usr/bin/timedatectl set-time *, /usr/bin/timedatectl set-ntp *`. Аргументы ограничены намеренно: неограниченный `sudo date` позволяет читать файлы через `date -f` (GTFOBins) | `sudoers` |
| 6 | Скриншот и артефакты | Скриншот терминала с номером; `history`, `stat`, `getcap` | `mephi-screenshot.png`, `history.out` |

## Файлы в репозитории

| Файл | Раздел | Содержимое |
|---|---|---|
| `mephi-screenshot.png` | 6 | Скриншот терминала с уникальным номером |
| `history.out` | все | История выполненных команд |
| `stat.out` | 1, 3, 4 | `stat` для `/home/user1`, `/home/user1/cat`, `/home/user1/chown` |
| `getcap.out` | 4 | Capabilities утилиты `/home/user1/chown` |
| `passwd`, `shadow`, `group` | 1 | Копии `/etc/passwd`, `/etc/shadow`, `/etc/group` |
| `sudoers` | 5 | Копия `/etc/sudoers` |
| `suid_files.out` | 2.1 | Найденные файлы с set-UID |
| `ps_privileged.out` | 2.2 | Процессы с EUID=0 и RUID обычного пользователя |

## Сохранение после перезагрузки

Все изменения хранятся в файлах и переживают перезагрузку: учётные записи и политика паролей в `/etc/passwd`, `/etc/shadow` и `/etc/group`, бит set-UID в правах файла, capabilities в расширенном атрибуте `security.capability` файловой системы, правило sudo в `/etc/sudoers`.

> Пароли на этой учебной ВМ одноразовые и больше нигде не используются: `shadow` опубликован по условию задания.
