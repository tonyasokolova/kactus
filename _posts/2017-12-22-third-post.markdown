---
title:  "Перехват трафика"
date:   2017-12-22 15:20:00
description: Как пользоваться Burp Suite. 
---

И так, в данной статье рассмотрим как осуществляется перехват трафика с помощью такой программы, как Burp Suite.

**Burp Suite** — это интегрированная платформа, предназначенная для проведения аудита веб-приложения, как в ручном, так и в автоматических режимах. Данный инструмент представляет из себя проксирующий механизм, перехватывающий и обрабатывающий все поступающие от браузера запросы. Имеется возможность установки сертификата burp для анализа https соединений.

### Настройка Burp Suite

*Настройка в Burp Suite*

Первое, что мы делаем, это запускаем сам Burp Suite. После чего во вкладке **Proxy** переходим во вкладку **Options**, как показано на картинке ниже, и проверяем верный ли интерфейс у нас стоит (должен быть 127.0.0.1:8080):

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp1.PNG)

Если же нужного интерфейса у нас нет, то нажимаем на **Add**:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp2.PNG)

Перед нами выводится окно. Там мы прописываем нужный нам порт (в нашем случае это 8080), выбираем нужный адрес (а именно 127.0.0.1) и нажимаем **ok**:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp3.PNG)

*Настройка браузера*

Настройки Burp Suite недостаточно. Чтобы перехватить трафик, нам необходимо настроить браузер.

И так, открываем браузер и заходим в настройки. Находим в нашем браузере настройки сети (в разных браузерах может быть по-разному):

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp4.PNG)

В появившемся окне вводим настройки такие же как на картинке ниже и нажимаем **ok**:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp5.PNG)

PS Все настройки были воспроизведены на ОС kali linux и в браузере Firefox.
