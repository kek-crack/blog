# Курим base64

Википедия говорит, что надо взять исходный текст, поделить на секции по 24 бита, а их в свою очередь на четыре секции по 6 бит. Cтандарт [RFC4648](https://datatracker.ietf.org/doc/html/rfc4648) говорит, что для кодирования используется строка:

```
ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=
```

Если в последней секции меньше 24 бит, то добиваем символом ```'='```.

Каждая шестибитная секция является индексом символа в строке для кодирования.

![base64 wiki](https://user-images.githubusercontent.com/74491315/147855198-50254165-2159-4d0f-b33d-c88e972a575b.png)

Сперва дефайним строку для кодирования:

```c
#define BASE64 "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/="
```
Так представлены буквы (ASCII) в десятичной системе счисления:

```
'M' => 77
'a' => 97
'n' => 110
```

С помощью битовых операций получаем индексы:

```
77 >> 2 => 19 (BASE64[19] == 'T')
((77 & 3) << 4) | (97 >> 4) => 22 (BASE64[22] == 'W')
(97 & 15) << 2) | ((110 >> 6) => 5 (BASE64[5] == 'F')
110 & 63 => 46 (BASE64[46] == 'u')
```

Итоговый текст программы будет выглядить так:

```c
#include <stdio.h>
#include <string.h>

#define BASE64 "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/="

void base64encode(char *s) {
    int len = strlen(s);
    int final = len % 3;
    char sym_enc[5];

    for (int i = 0; i < len; i += 3) {
        sym_enc[0] = BASE64[s[i] >> 2];
        sym_enc[1] = BASE64[((s[i] & 3) << 4)
                    | (s[i + 1] >> 4)];
        sym_enc[2] = BASE64[((s[i + 1] & 15) << 2)
                    | ((s[i + 2] >> 6))];
        sym_enc[3] = BASE64[s[i + 2] & 63];
        sym_enc[4] = 0;

        if (i >= len - 3) {
          if (final == 1)
            sym_enc[2] = sym_enc[3] = BASE64[64];
          else if (final == 2)
            sym_enc[3] = BASE64[64];
        }

        printf("%s", sym_enc);
    }
}

int main(int argc, char **argv) {base64encode(argv[1]);}
```

Вот и все. Совсем не сложно :)
