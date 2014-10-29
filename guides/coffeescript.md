# Coffee Script

## Общие советы

* Самый простой способ понять, что именно делает данное выражение в coffeescript - протранслировать его в js. На 
официальном сайте языка есть специальный интерфейс для данной задачи.

  * Зайдите на [coffeescript.org](http://coffeescript.org/)
  * Нажмите "TRY COFFEESCRIPT"
  * Введите изучаемое выражение в открывшемся редакторе
  * Проанализируйте сгенерированный javascript-код.
  * Profit! :smile: 

* При описании объектов более элегантно выглядит компактная запись

```coffeescript
# Обычная запись
value = getValue()
obj = {
  value: value
}

# Компактная запись
value = getValue()
obj = {value}

# Для нескольких полей такая запись тоже подходит
firstName = @get 'firstName'
lastName = @get 'lastName'
userData = {firstName, lastName}

# Можно использовать компактную запись вместе с обычной
firstName = @get 'firstName'
lastName = @get 'lastName'
userData = {
  firstName
  lastName
  age: getUserAge()
}
```

* Описывая обработчики событий, помните, что в coffeescript всё возвращает значение, даже если это не указано явно. 
В браузере для обработчиков некоторых событий действует правило, согласно которому, если функция-обработчик возвращает 
`false`, для данного события отменяется реакция браузера по умолчанию 
([подробнее про действие по умолчанию](http://learn.javascript.ru/default-browser-action)).

```coffeescript
# Данный код содержит side effect
someStatusVar = true
($ '.pretty-and-long-selector-for-our-element').on
  click: ->
    someStatusVar = false 
    # значение последнего выражения в строке равно false
    # данная функция вернет false,
    # что приведет к отмене действия по умолчанию

#  Исправленный вариант
someStatusVar = true
($ '.pretty-and-long-selector-for-our-element').on
  click: ->
    someStatusVar = false
    true 
    # можно вернуть все, что угодно, кроме false
```
