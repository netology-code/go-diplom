# Дипломная работа к профессии golang-разработчик 

Дипломный проект представляет собой каталог интернет-магазина. Ваша задача заключается в создании работающего бэкэнд-приложения,всеми основными функциями которого можно пользоваться.

## Описание

* В каталоге хранятся продукты, категории продуктов, цены на продукты, 
  магазины (под магазинами подразумеваются физические магазины, где забираем заказ)


* Каталог должен отдавать продукты с ценами, по магазину, по категории, давать возможность поиска по названию товара 


* Необходимо реализовать бизнес-логику получения данных по заданным маршрутам, 
  также необходимо реализовать кеширование
  (например, можно закешировать ответ поисковый запрос или запрос магазинов,
  кеширование цен возможно только, если внешние сервисы отдают ответы с разной периодичностью)
  

* Реализовать аутентификацию входящих запросов
  (рекомендуется делать это через использование jwt токена и middleware)


* Необходимо предусмотреть, что пользователь может быть администратором, то есть иметь возможность добавлять и изменять данные
  (можно указать в настройках токен администратора и в middleware проверять тип запроса)
  

* Товары без цен возможны, в таком случае в реальном магазине рисуется кнопка ``` Узнать цену ```
  

* в каталог может обращаться два типа пользователей: пользователь и администратор

#### Пользователь может:
  * получать продукты
  * получать список категорий
  * получать список магазинов
  * получать продукты по категории или магазину
  * выполнять поиск по наименованию продукта

#### Администратор может:
  * то же, что и пользователь, а также
  * добавлять продукт
  * добавлять цену
  * изменять продукт
  * изменять цену
  * добавлять категорию
  * добавлять магазин
  * изменять магазин и категорию

### Рекомендуется писать модульные тесты на обработчики

## Этапы разработки

Разработку Backend-сервиса рекомендуется разделить на следующие этапы:

1. Этап 1 - Создание модели данных 
2. Этап 2 - Создание бизнес логики и обработчиков 

Настоятельно рекомендуется сдавать данную работу на этих промежуточных этапах вашему дипломному руководителю. Старайтесь делать это как можно чаще для того, чтобы избежать лишнего переписывания кода в процессе хождения не в ту сторону.

Разберём подробно каждый этап.

### Этап 1. Создание модели данных

На данном этапе мы реализуем модели в storage и физически в БД.

Данные объявлений должны храниться в PostgreSql.

В моделях заданы поля для даты и времени создания записи и даты и времени обновления, нужно, чтобы они заполнялись автоматически.

Необходимо реализовать модель для продукта, категорий, цен и магазинов.

#### Модель для категории продукта
  Категории нужны для правильного отображения товара в интернет-магазине. Для простоты считаем, что все категории активны:
  
  | название |    тип     | обязательное | уникальное |
  | -------- | :--------: | :----------: | :--------: |
  |   id     |   `int`    |      да      |     да     |
  | name   | `varchar` |      да      |    да     |
  | create_at   |   `timestamp`   |      да      |    да     |
  | updated_at     |  `timestamp`  |      да      |    да     |
  | uri_name   |   `varchar`   |     да      |    нет     |

  Связка с товаром:
  
  | название |    тип     | обязательное | уникальное |
  | -------- | :--------: | :----------: | :--------: |
  | category_id  | `int` |      да      |    да     |
  | product_id   |   `int`   |      да      |    да     |

  В данной таблице уникальность создается составным первичным ключом ил 2 полей

#### Модель для магазинов:

  | название |    тип     | обязательное | уникальное |
  | -------- | :--------: | :----------: | :--------: |
  |   id     |   `int`    |      да      |     да     |
  | name   | `varchar` |      да      |    нет     |
  | create_at  |   `timestamp`   |      да      |    нет     |
  | updated_at     |  `timestamp`  |      да      |    нет     |
  | address   |   `varchar`   |     да      |    нет     |
  | lon   |   `varchar`   |     нет      |    нет     |
  | lat   |   `varchar`   |     нет      |    нет     |
  | working_hours   |   `varchar`   |     нет      |    нет     |

  Связка с товаром:
  
  | название |    тип     | обязательное | уникальное |
  | -------- | :--------: | :----------: | :--------: |
  | shop_id   | `int` |      да      |    да     |
  | product_id   |   `int`   |      да      |    да     |

В данной таблице уникальность создается составным первичным ключом ил 2 полей

#### Модель для цен

 В нашем случае цена, как правило, будет одна, но мы предусмотрим модель на случай расширение функционала, 
 (например, если цена будет зависеть от города или если мы захотим проследить динамику цен)

  | название |    тип     | обязательное | уникальное |
  | -------- | :--------: | :----------: | :--------: |
  |   id     |   `int`    |      да      |     да     |
  | sale_price   | `int` |      да      |    нет     |
  | factory_price   |   `int`   |      да      |    нет     |
  | discount_price     |  `int`  |      да      |    нет     |
  | create_at   |   `timestamp`   |     нет      |    нет     |
  | updated_at   |   `timestamp`   |     нет      |    нет     |
  | is_active   |   `bool`   |     да      |    нет     |
  | product_id   |   `int`   |     да      |    нет     | 


#### Модель для товара

  | название |    тип     | обязательное | уникальное |
  | -------- | :--------: | :----------: | :--------: |
  |   id     |   `int`    |      да      |     да     |
  | sku   | `varchar` |      да      |    нет     |
  | name   |   `varchar`   |      да      |    нет     |
  | type     |  `varchar`  |      да      |    нет     |
  | uri   |   `varchar`   |     да      |    нет     |
  | description   |   `varchar`   |     да      |    нет     |  
  | is_active   |   `bool`   |     да      |    нет     |  
  | create_at   |   `timestamp`   |     да      |    нет     |
  | updated_at   |   `timestamp`   |     да      |    нет     |

    
### Этап 2. Создание бизнес логики и обработчиков

#### Необходимо реализовать следующие маршруты:

* GET /api/v1/products
* GET /api/v1/search
* GET /api/v1/categories
* GET /api/v1/shops
* GET /api/v1/categories/:category_id/products
* GET /api/v1/shops/:shop/products
* POST /api/v1/product
* PUT /api/v1/product
* POST /api/v1/category
* PUT /api/v1/category
* POST /api/v1/shop
* PUT /api/v1/shop
* POST /api/v1/price
* PUT /api/v1/price

#### Получение продуктов
`GET /api/v1/products` — все получить продукты из каталога
В ответ приходит  `json-объект` с данными:

```json
{
  "total": 1,
  "items": [
    {
      "sku": "2074",
      "name": "Кирпич",
      "uri": "/product/brick-2074",
      "description": "Кирпич",
      "prices": [
        {
          "sale": 10.00,
          "factory": 2.00,
          "discount": 8.00
        }
      ]
    }
  ],
  "status": "ok"
}
```
поле uri - содержит slug для продукта, состоящий из типа продукта и sku продукта, используется во фронт-энд сервисе
формируется по формуле `/products/{тип продукта}-{sku}`
если массива цен нет, то в json должен возвращаться пустой массив 

#### Поиск продуктов
  `GET /api/v1/search` — поиск продуктов по наименованию

  Параметры
  
  | название |    тип     | обязательное |
  | -------- | :--------: | :----------: |
  | product_name   | `string` |      да      |

  В ответ приходит  `json-объект` с данными:

```json
{
  "total": 1,
  "items": [
    {
      "sku": "2074",
      "name": "Кирпич",
      "uri": "/product/brick-2074",
      "description": "Кирпич",
      "prices": [
        {
          "sale": 10.00,
          "factory": 2.00,
          "discount": 8.00
        }
      ]
    }
  ],
  "status": "ok"
}
```

#### получение списка категорий

  `GET /api/v1/categories` — все получить продукты из каталога
  В ответ приходит  `json-объект` с данными:
  ```json

{
  "total": 2,
  "items": [
    {
      "id": "2476",
      "parent_id": "603",
      "uri_name": "linoleum-2476",
      "name": "Линолеум"
    },
    {
      "id": "2480",
      "parent_id": "603",
      "uri_name": "laminat-2480",
      "name": "Ламинат"
    }
  ],
  "status": "ok"
}
  ```

поле uri - содержит slug для категории, состоящий из названия категории и id категории, используется во фронт-энд сервисе
формируется по формуле `{названия категории}-{id}`

#### получение списка магазинов
  `GET /api/v1/shops` — все получить продукты из каталога
  В ответ приходит  `json-объект` с данными:
  ```json
  {
  "total": 2,
  "items": [
    {
      "id": "1",
      "name": "Магазин №1",
      "address": " Ленина 1 стр 1",
      "working_hours": "8:00-20:00",
      "lon": "12.554764",
      "lat": "12.554564",
    },
    {
      "id": "2",
      "name": "Магазин №2",
      "address": " Ленина 2 стр 2",
      "working_hours": "8:00-20:00",
      "lon": "39.643151",
      "lat": "-0.351560",
    }
  ],
  "status": "ok"
}
  ```

#### получение всех продуктов по категории
  выдает все активные продукты (у которых is_active == true) 
`GET /api/v1/categories/:category_id/products` — получить все продукты по категории
  Параметры
  
  | название |    тип     | обязательное |
  | -------- | :--------: | :----------: |
  | category_id   | `string` |      да      |

  Ответ аналогичен ответу `GET /api/v1/products`


#### получение всех продуктов по магазину
  выдает все активные продукты (у которых is_active == true)
  `GET /api/v1/shops/:shop_id/products` — получить все продукты по id магазина
  Параметры

  | название |    тип     | обязательное |
  | -------- | :--------: | :----------: |
  | shop_id   | `string` |      да      |
  
  Ответ аналогичен ответу `GET /api/v1/products`

#### Добавление продукта
   
   **Внимание! Категория и Магазин уже должны быть созданы**
   
`POST /api/v1/product` - Добавляет продукт
```json
    {
    "sku": "2075",
    "name": "Кирпич 2",
    "uri": "/product/brick-2075",
    "description": "Кирпич 2", 
    "shop_id": "2",
    "category_id": "2480"
    }
```
    
В ответ приходит либо сообщение об ошибке, либо JSON-объект с данными:
    
```json
    {
  "total": 1,
  "items": [
    {
      "sku": "2075",
      "name": "Кирпич 2",
      "uri": "/product/brick-2075",
      "description": "Кирпич 2"
    }
  ],
  "status": "ok"
}
```

```json
    {
      "error": "что то пошло не так",
      "status": "error"
    }
```

#### Обновление продукта
`PUT /api/v1/product` - Обновляем продукт
   
Обязательное поле sku
```json
    {
    "sku": "2075",
    "is_active": "false" 
    }
```

В ответ приходит либо сообщение об ошибке, либо JSON-объект с данными:

```json
    {
  "total": 1,
  "items": [
    {
      "sku": "2075",
      "name": "Кирпич 2",
      "uri": "/product/brick-2075",
      "description": "Кирпич 2",
      "is_active": "False"
    }
  ],
  "status": "ok"
}
```

```json
    {
      "error": "что то пошло не так",
      "status": "error"
    }
```

Аналогично делаем добавление и изменение магазинов, категорий и цен.

### Этап 3. Возможные внешние зависимости

* Необходимо предусмотреть возможность нашего сервиса обращаться в другие сервисы (например, сервис доступности или актуальных цен).
  Для этого необходимо использовать механизм gRPS (внешние сервисы необходимо замокать).
  
# Запуск приложения

Для запуска приложения в корне проекта должны находиться следующие файлы:

- `Dockerfile` для сборки образа приложения;
- `docker-compose.yaml` с сервисом приложения и сервисом postgres;
- `README.md` с описанием проекта и вариантами его запуска.

Настройка параметров приложения должна производиться через переменные окружения. Это требование как для запуска в окружении хоста, так и при работе с docker.

Список переменных окружения должен быть описан в файле `.env-example`. Этот файл не должен содержать значений. Пример файла:

```bash
HTTP_HOST=
HTTP_PORT=
POSTGRES_URL=
```

### Как правильно задавать вопросы дипломному руководителю?

**Что следует делать, чтобы все получилось:**

* Попробовать найти ответ сначала самому в интернете. Именно этот скилл поиска ответов пригодится вам на первой работе. И только после этого спрашивать дипломного руководителя.
* В одном вопросе должна быть заложена одна проблема.
* По возможности прикрепляйте к вопросу скриншоты и стрелочкой показывать, где не получается. Программу для этого можно скачать здесь https://app.prntscr.com/ru/
* По возможности задавать вопросы в комментариях к коду.
* Начинайте работу над дипломом как можно раньше! Чтобы было больше времени на правки.
* Делать диплом по частям, а не все сразу. Иначе есть шанс, что нужно будет все переделывать :)

**Что следует делать, чтобы ничего не получилось:**

* Писать вопросы вида “Ничего не работает. Не запускается. Всё сломалось.”
* Откладывать диплом на потом.
* Ждать ответ на свой вопрос моментально. Дипломные руководители - работающие разработчики, которые занимаются, кроме преподавания, своими проектами. Их время ограничено, поэтому постарайтесь задавать правильные вопросы, чтобы получать быстрые ответы! 
