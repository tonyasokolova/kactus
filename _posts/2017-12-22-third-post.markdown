---
title:  "Перехват трафика с помощью Burp Suite"
date:   2017-12-22 12:50:00
description: Как настроить браузер и Burp Suite, чтобы перехватить трафик 
---

И так, в данной статье рассмотрим как осуществляется перехват трафика с помощью такой программы, как Burp Suite.

**Burp Suite** — это интегрированная платформа, предназначенная для проведения аудита веб-приложения, как в ручном, так и в автоматических режимах. Данный инструмент представляет из себя проксирующий механизм, перехватывающий и обрабатывающий все поступающие от браузера запросы. Имеется возможность установки сертификата burp для анализа https соединений.

### Настройка Burp Suite

*Настройка в Burp Suite*

Первое, что мы делаем, это запускаем сам Burp Suite. После чего во вкладке **Proxy** переходим во вкладку **Options**, как показано на картинке ниже, и проверяем верный ли интерфейс у нас стоит (должен быть 127.0.0.1:8080):

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp1.PNG)

Если же нужного интерфейса у нас нет, то нажимаем на **Add**:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp2.PNG)

Перед нами выводится окно. Там мы прописываем нужный нам порт (в нашем случае это 8080), выбираем нужный адрес (а именно 127.0.0.1) и нажимаем **OK**:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp3.PNG)

*Настройка браузера*

Настройки Burp Suite недостаточно. Чтобы перехватить трафик, нам необходимо настроить браузер.

И так, открываем браузер и заходим в настройки. Находим в нашем браузере настройки сети (в разных браузерах может быть по-разному):

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp4.PNG)

В появившемся окне вводим настройки такие же как на картинке ниже и нажимаем **OK**:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp5.PNG)

*Перехват http трафика*

Всё, теперь у нас всё настроено, можем перехватывать http трафик.

В burp переходим во вкладку **Intercept** и включаем перехват:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp6.PNG)

Ищем любой незащищенный сайт с протоколом http и заходим на него в браузере. Видим, что сайт долго прогружается, это значит что запрос перехвачен с помощью burp.

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp8.PNG)

Заходим в burp и смотрим, что мы получили некоторые данные - это и есть перехваченный запрос (см. картинку ниже). Он содержит стартовую строку и заголовки. Чтобы посмотреть дальше (следующие запросы, если они есть), нужно нажать **Forward**. Также можно выключить Intercept  и просматривать трафик во вкладке **HTTP history**.

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp7.PNG)

*Перехват https трафика*

Большинство нужных нам сайтов защищены, а именно используют https. Как быть в этом случае? Всё довольно просто. Нам надо сделать дополнительные настройки, а именно загрузить сертификат бурпа в браузер.

И так, скачать сертификат можно двумя способами.

**Способ 1.** В браузере в адресной строке вводим либо http://burp, либо 127.0.0.1:8080 и нажимаем на **CA Certificate**:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp9.PNG)

Далее выскакивает окно, где нажимаем сохранить: 

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp10.PNG)

**Способ 2.** В Burp Suite зайти во вкладку **Options** и нажать на **Generate CA certificate**:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp11.PNG)

Теперь этот сертификат надо загрузить в браузер. Опять же в настройках ищем место, где говорится про сертификаты, нажимаем на просмотр сертификатов:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp12.PNG)

 И добавляем наш сертификат (нажимаем на **import**, ищем куда мы сохранили сертификат и добавляем):
 
![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp13.PNG) 

Теперь мы можем перехватывать https трафик.

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp16.PNG) 
![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/burp15.PNG) 

### Другие возможности Burp Suite

Можно собирать информацию об архитектуре веб-приложения, используя spider, также есть возможность перебирать пароли с помощью Intruder, сканировать приложение на наличие уязвимостей и т.д. Но подробности всех этих возможностей будут описываться в других статьях.

### Итоги

Подведем итоги. В данной статье мы познакомились с программой Burp Suite. И узнали как перехватывать http и https трафик с помощью данной программы (настроили программу и наш браузер для перехвата данных).

PS Все настройки в данной статье были воспроизведены на ОС kali linux и в браузере Firefox.
