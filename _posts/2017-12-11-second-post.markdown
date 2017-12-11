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

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

{% highlight javascript %}
class Welcome extends React.Component \{
  render() \{
    return <h1>hELLO, \{this.props.name\}</h1>;
  \}
\}
{% endhighlight %}

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/react1.jpg)

Обратите внимание на синтаксис в операторе return: это _**JSX**_, расширение синтаксиса JavaScript, разработанное командой ReactJS. Этот формат используется для упрощения написания компонентов ReactJS. На этапе сборки-препроцессинга JSX транслируется в обычный JavaScript код. Следующие два примера идентичны: 

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/react2.jpg)

Новые элементы создаются с помощью функции createElement():

![index page](https://raw.githubusercontent.com/tonyasokolova/tonyasokolova.github.io/master/assets/images/react3.jpg)
