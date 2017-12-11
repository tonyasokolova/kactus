---
title:  "Основные проблемы безопасности при использовании ReactJS"
date:   2017-12-11 16:20:00
description: В данной статье рассмотрены основные уязвимости, которые могут совершать разработчики при создании React-приложений, а также уязвимости типа “CSS injection” в CSS-in-JS библиотеках и способы их эксплуатации.
---

React — популярная javascript-библиотека для создания UI.

В данной статье рассмотрим основные уязвимости, которые могут совершать разработчики при создании React-приложений. В React реализована защита по умолчанию от уязвимостей типа HTML-injection, так что разработчики зачастую “забывают” о других возможностях проведения XSS-атак. Также рассмотрим уязвимости типа “CSS injection” в CSS-in-JS библиотеках и способы их эксплуатации. 

### Компоненты, свойства и элементы

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


### Внедрение дочерних узлов

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
