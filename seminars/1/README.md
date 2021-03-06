## Семинар 1.
### Знакомство с командной строкой Linux. Работа с Hadoop HDFS.

На данном семинаре будут рассмотрены:

- Основные команды Linux, необходимые для комфортной работы на других семинарах, а также для выполнения домашних работ.
- Основные команды Hadoop HDFS,  необходимые для выполнения манипуляций с данными в файловой системе hdfs.

После данного семинара студент должен уметь:

- подключаться к кластеру в терминальном режиме (по протоколу ssh)
- получать доступ к портам кластера через ssh порт 22
- передавать и забирать файлы с кластера
- получать справку по командам на кластере
- создавать сессии, запускать процессы в сессии, отключаться и подключаться к своим сессиям
- передавать вывод одного процесса на вход другого процесса, фильтровать вывод процесса
- управлять запущенными процессами (просмотр, завершение), в том числе пакетное управление процессами
- просматривать и изменять права на файлы
- просматривать содержимое файлов, редактировать в текстовые файлы в терминальном режиме
- находить необходимое содержимое файлов
- пакетное изменение содержимого файлов
- добавлять, удалять, просматривать, вычислять занимаемый размер, изменять права на файлы в файловой системе hdfs

#### Материалы семинара
1. Для подключения к кластеру в терминальном режиме необходимо установить ssh соединение к  `hadoop2.yandex.ru` (по умолчанию используется порт 22, но можно указать опцией -p)

        $ ssh user@hadoop2.yandex.ru
Для os windows рекомендуется использовать или утилиту putty (http://the.earth.li/~sgtatham/putty/0.63/htmldoc/), или cygwin (https://cygwin.com/cygwin-ug-net.html)

2. Для возможности открытия портов кластера на своей машине необходимо использовать port forwarding (https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding) для ssh соединения, например, для forward конкретного порта необходимо использовать опцию -L, для динамического forward портов, необходимо использовать опцию -D:
        $ ssh user@hadoop2.yandex.ru -L 50070:hadoop2.yandex.ru:50070
        $ ssh user@hadoop2.yandex.ru -D 1234

3. Для передачи файлов на кластер рекомендуется использовать или различные командные менеджеры или утилиту scp, например,
        $ scp file.txt user@hadoop2.yandex.ru:/home/user/
        $ scp user@hadoop2.yandex.ru:/home/user/file.txt .
4. Для просмотра содержимого файла используют утилиты `cat, head, tail`:

        $ tail -f log.log | grep ERROR

5. Для получения справки по команде необходимо запустить команду man

        $ man cat

6. При открытии терминала начинается сессия пользователя. Все процессы пользователей запускаются в текущей сессии. Сессия заканчивается при отключении от терминала. Для создания сессии вне открытого терминала (для того, чтобы сессия не заканчивалась после отключения терминала) необходимо  использовать специальные утилиты, например, `tmux` (https://wiki.archlinux.org/index.php/Tmux_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9) ) или `screen` (https://www.gnu.org/software/screen/manual/screen.html).

7. Посмотреть запущенные процессы возможно через утилиту ps

        $ ps aux
Для передачи вывода одного процесса на вход другого процесса использую pipe - вертикальная черта в командной строке.
    - У процесса есть два стандартных вывода - stdout и stderr.
Для фильтрации выводимых процессов, например, по имени, используют утилиту grep. При этом используют передачу вывода через  pipe:

        $ ps aux | grep python
    - У каждого процесса есть уникальный идентификатор pid. При запуске, процесс создает в директории /proc папку, в котором хранится контекст необходимый для работы процесса. Например, в директории /proc/($PID)/status хранится статус процесса.

    - Для остановки процесса используют команду kill, аргументом, которой передают pid процесса. Процессу посылается сигнал для завершения. Для “жесткого” завершения процесса используют опцию -9

            $ kill 1234
    - Для получения только pid отфильтрованных процессов, например, возможно воспользоваться утилитами cut и tr

            $ps aux | grep python | tr -s ' ' | cut -d' ' -f 2
    - Для пакетной обработки вывода возможно использовать, например, команду `xargs`. Так, для пакетного завершения процессов, отфильтрованных в предыдущем пункте, необходимо выполнить

            $ps aux | grep python | tr -s ' ' | cut -d' ' -f 2 | xargs -L 1 -I {} sh -c “kill {}”

8. Для каждого файла в файловой системе есть владелец и выставлены права, которые определяются 9 битами. Эти биты разбиты на 3 группы: права для владельца, права для группы, в которой состоит владелец, и права для всех остальных (подробнее http://help.ubuntu.ru/wiki/%D1%81%D1%82%D0%B0%D0%BD%D0%B4%D0%B0%D1%80%D1%82%D0%BD%D1%8B%D0%B5_%D0%BF%D1%80%D0%B0%D0%B2%D0%B0_unix). Для изменения прав используется утилита chmod. Для изменения владельца файла используется утилита `chown`.

9. Для поиска файлов используется команда `find`.

        $ find . -type f
Типичный пример изменения прав на файлы и директории:

        $ find . -type f | xargs -L1 -I {} sh -c “chmod 644 {}”
        $ find . -type d | xargs -L1 -I {} sh -c “chmod 755 {}”
10. Для редактирования файлов в терминальном режиме используют различные редакторы (`vim, nano`). Удобно настроить удаленное редактирование файлов на кластере, через различные утилиты. Например, для sublimeText  возможно использовать утилиту sftp/ftp.

11. Для нахождения необходимого содержимого в файлах используют утилиту `grep` с различными опциями для поиска по контексту.

        $ grep text file.txt
        $ grep '^text' file.txt
        $ grep 'text$' file.txt
        $ grep '^text$' file.txt
        $ grep '\^s' file.txt
        $ grep '[Tt]ext' file.txt
        $ grep 'B[oO][bB]' file.txt
        $ grep '^$' file.txt
        $ grep '[0-9][0-9]' file.txt

11. Для пакетного изменения содержимого файлов возможно использовать утилиту `sed`.

        $ sed -i ‘s/http/https/g’ *.html

12. Для работы с hadoop hdfs используется утилита hdfs dfs (https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html)

    - Для просмотра текущей директории в hdfs необходимо выполнить

            $ hdfs dfs -ls .
    - Для создания директории в относительном пути в hdfs  (/user/<USERNAME>) необходимо выполнить

            $ hdfs dfs -mkdir my_dir
    - Для переноса файла в распределенную систему hdfs необходимо выполнить

            $ hdfs dfs -put file.txt my_dir/
    - Для взятия файла из hdfs на машину кластере необходимо выполнить

            $ hdfs dfs -get my_dir/file.txt .
    - Для изменения прав в файлах в hdfs необходимо выполнить

            $ hdfs dfs -chmod 644 my_dir/file.txt
    - Для вывода содержимого файла в hdfs в stdout необходимо выполнить

            $ hdfs dfs -cat my_dir/file.txt
            $ hdfs dfs -text my_dir/file.txt
    - Для сохранения содержимого файла в hdfs на машине кластера необходимо выполнить

            $ hdfs dfs -cat my_dir/file.txt > local_file.txt (то же самое что и hdfs dfs -get my_dir/file.txt)
            $ hdfs dfs -cat my_dir/file.txt >> local_file.txt (не то же самое что и hdfs dfs -get my_dir/file.txt)



