---
title:  "Основные проблемы безопасности при использовании ReactJS"
date:   2017-12-11 16:20:00
description: В данной статье рассмотрены основные уязвимости, которые могут совершать разработчики при создании React-приложений, а также уязвимости типа “CSS injection” в CSS-in-JS библиотеках и способы их эксплуатации
---

React — популярная javascript-библиотека для создания UI.

В данной статье рассмотрим основные уязвимости, которые могут совершать разработчики при создании React-приложений. В React реализована защита по умолчанию от уязвимостей типа HTML-injection, так что разработчики зачастую “забывают” о других возможностях проведения XSS-атак. Также рассмотрим уязвимости типа “CSS injection” в CSS-in-JS библиотеках и способы их эксплуатации. 

## Компоненты, свойства и элементы

Компоненты позволяют разделить UI на независимые, повторно используемые части и работать с каждой из них отдельно.

Концептуально, компоненты подобны JavaScript функциям. Они принимают произвольные данные (называемые “props”) и возвращают React-элементы, описывающие что должно появиться на экране. Базовый компонент выглядит следующим образом:

{% highlight react %}
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
{% endhighlight %}

Обратите внимание на синтаксис в операторе return: это _**JSX**_, расширение синтаксиса JavaScript, разработанное командой ReactJS. Этот формат используется для упрощения написания компонентов ReactJS. На этапе сборки-препроцессинга JSX транслируется в обычный JavaScript код. Следующие два примера идентичны: 

{% highlight react %}
// JSX 

const element = (
  <h1 className="greeting">
  Hello, world!
  </h1>
);

// Transpliled to createElement() call 

const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
{% endhighlight %}

Новые элементы создаются с помощью функции createElement():

{% highlight react %}
React.createElement(
  type,
  [props],
  [...children]
)
{% endhighlight %}

Эта функция принимает три аргумента:
*	Аргумент “type” может быть либо строкой имени тега (например, 'div' или 'span'), либо типом компонента React (класс или функция);
*	Вторым аргументом отправляется свойство. “props” содержит список атрибутов, например: class, id и т.д.;
*	“children” содержит дочерние выражения нового элемента.

Контроль над этими аргументами даёт злоумышленнику возможность реализовать различные вектора атак.


## Внедрение дочерних узлов

В марте 2015 года Даниель Лешеминан опубликовал статью <a href="http://danlec.com/blog/xss-via-a-spoofed-react-element">«XSS через поддельный элемент React»</a>, в которой сообщалось о хранимой XSS в HackerOne. Проблема была вызвана тем, что веб-приложение HackerOne передавало объект, доступный для изменения пользователем, в качестве дочернего аргумента в функцию React.createElement(). Предположительно, уязвимый код выглядит следующим образом:

* JSX:
{% highlight react %}
/* Retrieve a user-supplied, stored value from the server and parsed it as JSON for whatever reason.

attacker_supplied_value = JSON.parse(some_user_input)
*/

render() {
  return <span>{attacker_supplied_value}</span>;
}
{% endhighlight %}

* JavaScript:
{% highlight react %}
React.createElement("span", null, attacker_supplied_value);
{% endhighlight %}

Когда «attacker_supplied_value» — строка, как ожидалось, на странице появлялся обычный span-элемент. Однако, функция «createElement()» в текущей версии ReactJS принимает простые объекты, переданные как дочерние. Даниель нашел уязвимость, передавая JSON объект. Он передал в нем поле «dangerouslySetInnerHTML», которое позволило вставить необработанный HTML в вывод React. Окончательный proof-of-concept выглядел так:

{% highlight react %}
{
  _isReactElement: true,
  _store: {}, 
  type: "body",
  props: {
    dangerouslySetInnerHTML: {
      __html:
      "<h1>Arbitrary HTML</h1>
      <script>alert(`No CSP Support :(`)</script>
      <a href='http://danlec.com'>link</a>"
    }
  }
}
{% endhighlight %}

В ноябре 2015 года данную проблему решили следующим образом: элементы React были теперь отмечены атрибутом «$$typeof: Symbol.for('react.element')». Поскольку невозможно ссылаться на глобальный JavaScript символ из внедренного объекта, метод инъекции дочерних элементов не работает. 


## Внедрение Props

Рассмотрим следующий код:

{% highlight react %}
// Parse attacker-supplied JSON for some reason and pass
// the resulting object as props.
// Don't do this at home unless you are a trained expert!

attacker_props = JSON.parse(stored_value)

React.createElement("span", attacker_props);
{% endhighlight %}

Здесь мы можем внедрить произвольное свойство (props) в новый элемент. Мы можем использовать следующий код для установки «dangerouslySetInnerHTML»:

{% highlight react %}
{"dangerouslySetInnerHTML" : { "__html": "<img src=x/
onerror='alert(localStorage.access_token)'>"}}
{% endhighlight %}


## "Классическая" XSS

Некоторые традиционные XSS вектора также могут быть исполнены в приложениях с движком React. В данном разделе описаны анти-шаблоны.

### Явная настройка «dangerouslySetInnerHTML»

Разработчики могут нарочно установить свойство «dangerouslySetInnerHTML»:

{% highlight react %}
<div dangerouslySetInnerHTML={user_supplied} />
{% endhighlight %}

Контроль над значением этого свойства дает злоумышленнику внедрить любой JavaScript код.

## Внедряемые атрибуты

Контроль над атрибутом «href» динамически сгенерированного тега «a» дает возможность внедрить «javascript:». Некоторые другие атрибуты, такие как «formaction» в HTML5, также работают в современном браузере.

{% highlight react %}
<a href={userinput}>Link</a>

<button form="name" formaction={userinput}>
{% endhighlight %}
  
Следующий вектор также будет работать в современных браузерах:

{% highlight react %}
<link rel="import" href={user_supplied}>
{% endhighlight %}


## Рендеринг HTML на стороне сервера

Для улучшения начального времени загрузки страницы в последнее время наблюдается тенденция создавать ReactJS страницы с предварительным рендерингом на сервере. В ноябре 2016 года <a href="https://medium.com/node-security/the-most-common-xss-vulnerability-in-react-js-applications-2bdffbcc1fa0">Эмилия Смит</a> отметила, что официальный пример Redux кода для серверного рендеринга приводит к XSS, так как состояние клиента было соединено со страницей, так как клиентская часть подготавливается к рендерингу без санитизации данных (с тех пор пример был исправлен).

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/react4.jpg)

Вывод: если сделать пререндеринг HTML страницы на стороне сервера, то можно заметить те же типы XSS, как и в “обычных” веб-приложениях.

Вредоносная полезная нагрузка:

{% highlight react %}
preloadedState = {
  attacker_supplied :
    "xss</script><script>alert(1)</script>"
}
{% endhighlight %}

## CSS-in-JS

CSS-in-JS — новая технология, которая полностью устраняет необходимость в именах классов CSS.Это позволяет добавлять стили непосредственно к компонентам, используя всю мощь CSS. Но к сожалению, также открывается возможность атакам типа «инъекции».

Атаки с использованием CSS являются серьезной проблемой безопасности.

Если разработчик не будет следовать одному простому правилу: «никогда не интерполируйте ввод пользователя в свои таблицы стилей», то он может непреднамеренно разрешить злоумышленникам анализировать данные пользователей, украсть их учетные данные, выполнить произвольный код на JavaScript и т.д.

### Password stealing color

Предположим, что нужно разрешить пользователям выбирать цвет своей страницы профиля. Простым CSS это было бы довольно трудно, но CSS-in-JS упрощает работу. Бэкэнд-разработчик обработал API, и теперь есть цветовая поддержка в стилизованных компонентах.

Поскольку приложение является одностраничным, форма логина открывается как оверлей над профилем. И так как бэкэнд-разработчик сохранил цвет в текстовом поле без проверки, злоумышленник теперь может установить цвет, который может украсть некоторые пароли пользователей:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/react5.jpg)

Это работает при использовании селекторов атрибутов в поле пароля для изменения фонового изображения в зависимости от текущего ввода. После ввода пароля в консоли браузера выводится следующее:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/react6.jpg)

Хотя данная атака не может украсть все пароли, но несколько украденных паролей более чем достаточно.

## Кража данных

Данная атака на основе CSS заключается в злоупотреблении unicode-диапазоном @font-face. С помощью этой техники, злоумышленник может читать текст страницы:

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/react7.jpg)

Когда вы получаете доступ к этой странице, браузер извлекает «?A» и «?B», так как текстовая часть чувствительной информации содержит символы «A» и «B». Но не извлекается «?C», так как там не содержится символ «C». Это означает, что мы смогли прочитать «A» и «B».

### Выводы

Несмотря на то, что ReactJS вполне безопасен по дизайну, плохая практика программирования может привести к уязвимостям безопасности.
* Пентестерам: пробуйте внедрить JavaScript, JSON и CSS везде где можно и смотрите результат;
*	Разработчикам: никогда не используйте «eval()» или «dangerouslySetInnerHTML». Избегайте разбора пользовательского JSON. Никогда не интерполируйте ввод пользователя в свои таблицы стилей.

P.S. Данная статья была написана на основе статей <a href="https://medium.com/dailyjs/exploiting-script-injection-flaws-in-reactjs-883fb1fe36c1">Бернарда Мюллера</a> и <a href="https://reactarmory.com/answers/how-can-i-use-css-in-js-securely">Джеймса Нельсона</a>.
