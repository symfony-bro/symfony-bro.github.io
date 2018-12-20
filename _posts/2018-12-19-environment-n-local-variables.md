---
layout: post
title:  "Environment & local variables"
categories: [android]
tags: [synchronization, environment, gradle]
---

# Environment & local variables

Сегодня моё терпение кончилось.
Да, внезапно это произошло, хотя, я думал, что продержусь еще месяц-другой.
У кого есть такие строки в коде, тот поймет о чем пойдет речь:
<!--more-->

 ```java
//    private final String API_ENDPOINT = "http://192.168.1.111:8080/";
    private final String API_ENDPOINT = "https://prod.endpoint.com/";
```

Задача, я так думаю, встречается в 99% случаев разработки андроид-приложений  &mdash;  нужно синхронизировать данные с сервером.
Для плодотворной разработки полезно иметь локальный инстанс бэкенда.
Возможно вы не хотите похерить какие-то данные на проде, или параллельно с разработкой приложения нужно дописывать апиху, либо и то и другое, как в моём случае.
В рузультате имеем два адреса: локальный и продакшн (тестовый тоже не исключён).
Когдя я работаю, то комментирую прод адрес, а когда билдю релиз &mdash; локальный.
Очевидно, что это зашквар и наступил момент что-то делать.
Несмотря на, так сказать, высокочастотность кейса, быстро нагуглить решение не удалось.
Но оно есть и состоит из двух шагов.

**Первое** &mdash; сделать конфиг зависящий от окружения.
Для этого в `build.gradle (app module)` в ноде `android.buildTypes` нужно добавить ноды `debug` и `release` (release обычно уже присутствует), например:
```
android {
    compileSdkVersion 26
    // ...
    
    buildTypes {
        debug {
            buildConfigField "String", "API_ENDPOINT", "\"http://192.168.1.111:8080/\""
        }
        release {
            // ...
            buildConfigField "String", "API_ENDPOINT", "\"https://prod.endpoint.com/\""
        }
    }
}
```
Теперь нужный адрес доступен нам из `BuildConfig.API_ENDPOINT` и меняется автоматически в зависимости от окружения.
Но `build.gradle` находится в системе контроля версий, а локальный адрес бэкенда у разработчиков можем быть разным,
поэтому хотелось бы вынести его в какой-то локальный конфиг. Такой конфиг уже есть &mdash; это файл `local.properties`.

И наш **второй шаг** &mdash; перенести туда адрес локального сервера. Добавляем строку:
```ini
api.endpoint="http://192.168.1.111:8080/"
```
Теперь `api.endpoint` можно читать в `build.gradle`. Я нашел такой способ:
```
apply plugin: 'com.android.application'
// ...
def props = new Properties()
props.load(project.rootProject.file("local.properties").newDataInputStream())

android {
    compileSdkVersion 26
    // ...
    
    buildTypes {
        debug {
            buildConfigField "String", "API_ENDPOINT", props.getProperty("api.endpoint")
        }
        release {
            // ...
            buildConfigField "String", "API_ENDPOINT", "\"https://prod.endpoint.com/\""
        }
    }
}
```
Решено!