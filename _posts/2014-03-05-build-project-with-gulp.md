---
layout:     post
title:      сборка проекта с Gulp
date:       2014-03-05
summary:    Заметка о том, как собрать проект при помощи Gulp в JavaScript.
tags: Gulp JavaScript NodeJS
---

[Gulp](http://gulpjs.com/) это система потоковой сборки проектов на ```JavaScript```. При помощи этого инструмента можно автоматизировать рутинные задачи в web разработке. Какие именно? Например проверку js файлов на наличие ошибок, или потенциальных ошибок, минификацию js, css, компиляцию scss в css и тд..

Попробуем решить небольшую задачу, на примере которой можно увидеть всю прелесть gulp'а..

И так, будем проверять js файлы на наличие ошибок jshint'ом, сжимать js файлы, компилировать scss файлы в css, сжимать css файлы. Поехали..


#### Установка

Для установки ```gulp``` нам понадобится установленный npm - менеджер пакетов для ```nodejs``` (ставится автоматически при установке nodejs).

устанавливать ```gulp``` будем глобально, чтобы иметь к нему доступ из консоли. Также нужно будет его установить локально в папке проекта

{% highlight bash %}
npm install gulp -g

mkdir gulp && cd $_

npm install gulp
{% endhighlight %}

Проверяем

![check result](http://i.imgur.com/IdjQslN.png)



Для поставленной задачи нам также понадобятся плагины:

+ [gulp-cssmin](https://www.npmjs.org/package/gulp-cssmin) - минификатор для css файлов
+ [gulp-uglify](https://www.npmjs.org/package/gulp-uglify) - минификатор для js файлов
+ [gulp-jshint](https://www.npmjs.org/package/gulp-jshint) - инструмент для проверки js файлов на ошибки
+ [jshint-stylish](https://www.npmjs.org/package/jshint-stylish) - красивый репортер для jshint
+ [gulp-rename](https://www.npmjs.org/package/gulp-rename) - переименовывание файлов
+ [gulp-sass](https://www.npmjs.org/package/gulp-sass) - компиляция scss файлов в css

Устанавливаем
{% highlight bash %}
npm install gulp-cssmin gulp-uglify gulp-jshint jshint-stylish gulp-rename gulp-sass
{% endhighlight %}

В корне нашего проекта появилась директория ```node_modules```, которая содержит в себе все установленные плагины для gulp и сам gulp

Далее создадим структуру нашего проекта как показано на скриншоте

![project structure](http://i.imgur.com/XAr5fNi.png)

```gulpfile.js``` - конфигурационный файл для ```gulp```, пока пустой.
В директории ```assets``` будут лежать все наши js, css, scss файлы. Создаем только ```main.js``` и ```main.scss``` файлы, минифицированные файлы ```gulp``` будет создавать сам. В ```index.html``` подключаем будущие минифицированные файлы ```main.min.css``` и ```main.min.js```

Содержимое файла ```index.html```

{% gist dmaslov/22160743825b5d0c86d7 %}



#### Создание конфигурационного файла

Для ```gulp``` конфигурационным файлом является ```gulpfile.js```.
Сразу привожу финальный вид файла, ниже разберем что к чему

{% gist dmaslov/e8b995654c353262eeff %}


Проверим что получилось. В терминале:

![gulp run](http://i.imgur.com/vTAkN6r.png)

Видим что ```gulp``` создал файлы, которых не было изначально

![gulp run result](http://i.imgur.com/SyZvHBW.png)

Что произошло.. ```gulp``` нашел ```main.js``` файл, создал минифицированную версию с приставкой ```.min```, нашел ```main.scss``` файл, скомпилировал его в css и создал минифицированную версию с приставкой ```.min```

Задача выполнена, разберем подробней что происходит в нашем ```gulpfile.js```


Ниже мы подключаем установленные плагины и определяем поисковую таблицу для путей к нашим файлам.

{% gist dmaslov/4484d1f67cad1825d1df %}


Так выглядит описание типичной задачи. Тут мы видим, что последовательность действий связанна через метод ```.pipe()```, который передает поток данных по цепочке.

{% gist dmaslov/3dff189c64368cafd5f1 %}


Рассмотрим основные методы ```gulp API```:

+ .task() - определяет задачу
+ .src(glob) - принимает [glob](https://github.com/isaacs/node-glob) для обработки и возвращает входящий поток
+ .dest(dest) - принимает путь к файлу и возвращает исходящий поток
+ .run(tasks) - запускает задачу (deprecated)
+ .watch(glob) - отслеживает изменения в файлах

По-умолчанию (если в терминале вызвать комманду ```gulp```) запускается задача ```default```, мы можем запустить конкретную описанную в ```gulpfile.js``` задачу

{% highlight bash %}
gulp task_name
{% endhighlight %}

Что происходит дальше в нашем ```gulpfile.js```:

запускается задача ```default```, которая запускает задачи ```jshint```, ```js-minifier```, ```sass-compile``` и ```watch```

задача ```watch``` отслеживает изменения в файлах и выполняет соответствующие задачи, если файлы изменились

задача ```jshint``` проверяет js файлы на ошибки и в случае, если такие обнаружены, возвращает ошибку в терминал

задача ```js-minifier``` минифицирует js файлы

{% gist dmaslov/061b6298554752377169 %}


задача ```sass-compile``` компилирует scss файлы в css файлы. Тут есть одна отличительная особенность в коде. Дело в том, что метод ```.watch()``` в ```gulp API``` лишь следит за изменениями в файлах, и не сработает, если файл создается в процессе.. Мы же создаем css файл на основе scss и минифицируем его. Обычным способом задача минификации не вызовется, так как изменений в самом css файле не было, он создавался в процессе. Обойти этот неудобный момент поможет небольшой хак

{% gist dmaslov/e54a2ed1b12934d3e134 %}


тут присутствует метод ```.on('end', fn)```. Простыми словами, когда сработает событие ```end``` (создастся css файл) вызываем функцию ```cssMin()```, которая создаст минифицированный css файл.
