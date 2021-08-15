# Тестовый проект для демонстрации GraphQL
## Данные
### Пользователи
admin/admin<br>
mechanic/1<br>
manager/2

## Endpoint
Для работы с GraphQl необходимо быть авторизованным и использовать **header**
`"Authorization": "Bearer *your_token*"`
### GraphQL endpoint
[http://localhost:8080/graphql]()


### GraphiQL endpoint 
[http://localhost:8080/graphiql]()

## Тест кейсы
### Фильтры строк `contains`, `starts with`, `ends with`
```graphql
query{
  scr_CarList(
    filter: [
      {manufacturer: {_contains: "es"}}
    ]
  ){
    id
    manufacturer
  }
}
```
```graphql
query{
  scr_CarList(
    filter: [
      {manufacturer: {_notContains: "es"}}
    ]
  ){
    id
    manufacturer
  }
}
```
```graphql
query{
  scr_CarList(
    filter: [
      {manufacturer: {_startsWith: "v"}}
    ]
  ){
    id
    manufacturer
  }
}
```
```graphql
query{
  scr_CarList(
    filter: [
      {manufacturer: {_endsWith: "z"}}
    ]
  ){
    id
    manufacturer
  }
}
```

### Фильтры по коллекциям `in`, `not in`
```graphql
query{
  scr_CarList(
    filter: [
      {manufacturer: {_in: ["VAZ", "GAZ"]}}
    ]
  ){
    id
    manufacturer
  }
}
```
```graphql
query{
  scr_CarList(
    filter: [
      {manufacturer: {_notIn: ["VAZ", "GAZ"]}}
    ]
  ){
    id
    manufacturer
  }
}
```

### Фильтры дат
```graphql
query{
  scr_DatatypesTestEntityList(
    filter: [
      {dateAttr: {_eq: "2020-03-03"}}
    ]
  ){
    dateAttr
  }
}
```
_notEq будет аналогичен

Даты ранее указанной:
```graphql
query{
  scr_DatatypesTestEntityList(
    filter: [
      {dateAttr: {_lt: "2020-03-03"}}
    ]
  ){
    dateAttr
  }
}
```
`_gt`, `_gte`, `_lte` можно также в этом запросе продемонстрировать

Демонтрация работы с датой и временем:
```graphql
query{
  scr_DatatypesTestEntityList(
    filter: [
      {localDateTimeAttr: {_lt: "2020-03-03T03:03:03"}}
    ]
  ){
    localDateTimeAttr
  }
}
```
Полный список форматов дат указан в 
[README](https://github.com/Haulmont/jmix-graphql/blob/master/README.md#supported-date-datatypes)

### Фильтры по пустым значениям
```graphql
query{
  scr_DatatypesTestEntityList(
    filter: [
      {stringAttr: {_isNull: true}}
    ]
  ){
    id
    stringAttr
  }
}
```
```graphql
query{
  scr_DatatypesTestEntityList(
    filter: [
      {stringAttr: {_isNull: false}}
    ]
  ){
    id
    stringAttr
  }
}
```

### Сортировка по умолчанию
По умолчанию сущности сортируются по атрибуту даты последнего изменения по убыванию (если атрибут существует и не указана сортировка).
Для проверки выполняем запрос без сортировки
```graphql
query{
  scr_CarList{
    id
    manufacturer
    lastModifiedDate
  }
}
```
Затем тот же запрос, но с сортировкой - результат идентичен:
```graphql
query{
  scr_CarList(orderBy: {lastModifiedDate: DESC})
  {
    id
    manufacturer
    lastModifiedDate
  }
}
```

### Тесты в проекте
Интеграционные тесты фильтров можно посмотреть [здесь](https://github.com/Haulmont/jmix-graphql/tree/master/graphql/src/test/groovy/io/jmix/graphql/datafetcher/filter)

### Лимит запросов
Устанавливается в `application.properties`:

например, `jmix.graphql.operationRateLimitPerMinute=3`

В данном проекте закомментировано, для проверки раскомментировать, перезапустить приложение и выполнить несколько раз подряд один и тот же запрос

### Локализация
Для получения локализованных сообщений используем запрос:
```graphql
query{
  entityMessages(className: "scr_Car" locale:"ru") {
    key
    value
  }
  enumMessages(className: "com.company.scr.entity.CarType" locale: "ru"){
    key
    value
  }
}
```
В примере сразу рассмотрены сообщения сущностей и enum классов

### Количество записей `count` + фильтр
```graphql
query{
  scr_CarCount
}
```
С фильтром:
```graphql
query{
  scr_CarCount(
    filter: [
      {manufacturer: {_eq: "Mercedes"}}
    ]
  )
}
```

### Валидация сущностей
```graphql
mutation{
  upsert_scr_Car(car:{
    regNumber:"a99oo"
  })
  {
    id
    regNumber
  }
}
```
Запрос покажет ошибки валидации сущности при сохранении.
Выполняем запрос согласно правилам валидации:
```graphql
mutation{
  upsert_scr_Car(car:{
    regNumber:"aa999"
    manufacturer:"custom"
    carType: SEDAN
  })
  {
    id
    regNumber
    manufacturer
    carType
  }
}
```
### Запрет чтения, добавления сущностей
Делаем запрос с токеном пользователя Механик
```graphql
mutation{
  upsert_scr_Car(car:{
    regNumber:"aa999"
    manufacturer:"custom"
    carType: SEDAN
  })
  {
    id
    regNumber
    manufacturer
    carType
  }
}
```
Получаем сообщение о запрете изменения атрибута.
Проверяем права пользователя запросом:
```graphql
query{
  permissions{
    entities{
      target
      value
    }
    entityAttributes{
      target
      value
    }
  }
}
```
Все верно, к атрибуту `regNumber` у пользователя нет доступа 
### Загрузка специальных разрешений
Аналогично предыдущему запросу:
```graphql
query{
  permissions{
    specifics{
      target
      value
    }
  }
}
```

### Валидация композиций
```graphql
mutation{
  upsert_scr_DatatypesTestEntity(datatypesTestEntity:{
    name: "testing"
    compositionO2Mattr:{
      name: "testing deeply"
      quantity: -10
    }
  })
  {
    id
    name
    compositionO2Mattr{
      id
      name
      quantity
    }
  }
}
```
При создании сущности `DatatypesTestEntity` будет также выполнена валидация сущности `CompositionO2M`,
у которой атрибут `quantity` не может быть ниже нуля.
Замечание: валидация в глубину работает корректно только если на атрибуте
родительской сущности есть [аннотация `@Valid`
](https://github.com/Haulmont/jmix-graphql/issues/145#issuecomment-873882672)
