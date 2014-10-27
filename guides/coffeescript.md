# Coffee Script

## Общие советы

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
