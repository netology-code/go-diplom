# go-diplomn
Дипломная работа к профессии golang-разработчик 

Дипломный проект представляет собой каталог интернет магазина. 

Ваша задача заключается в создании работающего бэкэнд-приложения,
всеми основными функциями которого можно пользоваться.

## Описание

* В каталоге хранятся продукты, категории продуктов, цены на продукты, 
  магазины (под магазинами подразумеваются физические магазины, где забираем заказ)


* Каталог должен отдавать продукты с ценами, по магазину, по категории, давать возможность пиоска по названию товара 


* Необходимо реализовать бизнес-логику получения данных по заданным маршрутам
  (например можно закешировать ответ поисковый запрос или запрос магазинов,
  кеширование цен возможно только, если внешние сервисы отдают ответы с разной периодичностью
  

* так же необходимо реализовать аутентификацию входящих запросов
  рекомендуется делать это через использование jwt токена и middleware


* Необходимо предусмотреть, что пользователь может быть администратором, то есть иметь возможность добавлять и изменять данные
  (можно указать в настройках токен администратора и в middleware проверять тип запроса)
  

* Товары без цен возможны, в таком случае в реальном магазине рисуется кнопка ``` Узнать цену ```
  

* в каталог может обращаться два типа пользователей: пользователь и администратор

#### пользователь может:
  * получать продукты
  * получать список категорий
  * получать список магазинов
  * получать продукты по категории или магазину
  * выполнять поиск по наименованию продукта

#### администратор может:
  * то же, что и пользователь, а так же
  * добавлять продукт
  * добавлять цену
  * изменять продукт
  * изменять цену
  * добавлять категорию
  * добавлять магазин
  * изменять магазин и категорию

* Рекомендуется писать модульные тесты на обработчики

## Этапы разработки

Разработку Backend-сервиса рекомендуется разделить на следующие этапы:

1. Этап 1 - Создание модели данных 
2. Этап 2 - Создание бизнес логики и обработчиков 

Также настоятельно рекомендуется сдавать данную работу на этих промежуточных этапах вашему дипломному руководителю. Старайтесь делать это как можно чаще для того, чтобы избежать лишнего переписывания кода в процессе хождения не в ту сторону.

Разберём подробно каждый этап.

### Этап 1. Создание модели данных

На данном этапе мы реализуем модели в storage и физически в БД

Данные объявлений должны храниться в PostgreSql.


Необходимо реализовать модель для продукта, категорий, цен и магазинов

#### Модель для категории продукта
  Категории нужны, для правильного отображения товара в интернет магазине, для простоты считаем, что все категории активны
  
  | название |    тип     | обязательное | уникальное |
  | -------- | :--------: | :----------: | :--------: |
  |   id     |   `int`    |      да      |     да     |
  | name   | `string` |      да      |    да     |
  | createAt   |   `dateTime`   |      да      |    да     |
  | updatedAt     |  `dateTime`  |      да      |    да     |
  | UriName   |   `string`   |     да      |    нет     |

  Связка с товаром
  
  | название |    тип     | обязательное | уникальное |
  | -------- | :--------: | :----------: | :--------: |
  |   id     |   `int`    |      да      |     да     |
  | categoryId   | `int` |      да      |    да     |
  | productId   |   `int`   |      да      |    да     |

#### Модель для магазинов

  | название |    тип     | обязательное | уникальное |
  | -------- | :--------: | :----------: | :--------: |
  |   id     |   `int`    |      да      |     да     |
  | name   | `string` |      да      |    нет     |
  | createAt   |   `dateTime`   |      да      |    нет     |
  | updatedAt     |  `dateTime`  |      да      |    нет     |
  | Address   |   `string`   |     да      |    нет     |
  | Lon   |   `string`   |     нет      |    нет     |
  | Lat   |   `string`   |     нет      |    нет     |
  | WorkingHours   |   `string`   |     нет      |    нет     | 
  | ShopGuid   |   `uuid`   |     да      |    да     |

  Связка с товаром

  | название |    тип     | обязательное | уникальное |
  | -------- | :--------: | :----------: | :--------: |
  |   id     |   `int`    |      да      |     да     |
  | shopId   | `int` |      да      |    да     |
  | productId   |   `int`   |      да      |    да     |


#### Модель для цен

 В нашем случае цена, как правило, будет одна, но мы предусмотрим модель на случай расширение функционала, 
 (например если цена будет зависеть от города или если мы захотим проследить динамику цен)

  | название |    тип     | обязательное | уникальное |
  | -------- | :--------: | :----------: | :--------: |
  |   id     |   `int`    |      да      |     да     |
  | SalePrice   | `string` |      да      |    нет     |
  | FactoryPrice   |   `float`   |      да      |    нет     |
  | DiscountPrice     |  `float`  |      да      |    нет     |
  | createAt   |   `dateTime`   |     нет      |    нет     |
  | updatedAt   |   `dateTime`   |     нет      |    нет     |
  | IsActive   |   `bool`   |     да      |    нет     |
  | PriceGuid   |   `uuid`   |     да      |    да     | 
  | FkProduct   |   `int`   |     да      |    нет     | 


#### Модель для товара

  | название |    тип     | обязательное | уникальное |
  | -------- | :--------: | :----------: | :--------: |
  |   id     |   `int`    |      да      |     да     |
  | sku   | `string` |      да      |    нет     |
  | name   |   `string`   |      да      |    нет     |
  | type     |  `string`  |      да      |    нет     |
  | uri   |   `string`   |     да      |    нет     |
  | description   |   `string`   |     да      |    нет     |  
  | IsActive   |   `bool`   |     да      |    нет     |  
  | createAt   |   `dateTime`   |     да      |    нет     |
  | updatedAt   |   `dateTime`   |     да      |    нет     |

    
### Этап 2. Создание бизнес логики и обработчиков

#### Необходимо реализовать следующие маршруты:

* GET /api/v1/products
* GET /api/v1/search
* GET /api/v1/categories
* GET /api/v1/shops
* GET /api/v1/products_by_category
* GET /api/v1/products_by_shop
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
{ "Total" : 1,
  "Items": [{
    "sku": "2074",
    "name": "Кирпич",
    "uri": "/product/brick-2074",
    "description": "Кирпич",
    "prices" : [{ "sale": 10.00,
                  "factory": 2.00,
                  "discount": 8.00
    }]
  }]
}
```

если массива цен нет, то в json должен возвращаться пустой массив 

#### Поиск продуктов
  `GET /api/v1/search` — поиск продуктов по наименованию

  Параметры
  
  | название |    тип     | обязательное |
    | -------- | :--------: | :----------: |
  | product_name   | `string` |      да      |

  В ответ приходит  `json-объект` с данными:

```json
{ "Total" : 1,
  "Items": [{
    "sku": "2074",
    "name": "Кирпич",
    "uri": "/product/brick-2074",
    "description": "Кирпич",
    "prices" : [{ "sale": 10.00,
      "factory": 2.00,
      "discount": 8.00
  }]
}
```

#### получение списка категорий

  `GET /api/v1/categories` — все получить продукты из каталога
  В ответ приходит  `json-объект` с данными:
  ```json
  
  { "Total" : 2,  
    "Items": [
    {
        "id":"2476",
        "parent_id":"603",
        "uri_name":"linoleum-2476",
        "name":"Линолеум",


    },
    {
        "id":"2480",
        "parent_id":"603",
        "uri_name":"laminat-2480",
        "name":"Ламинат",
    }] 
  }
  ```

#### получение списка магазинов
  `GET /api/v1/shops` — все получить продукты из каталога
  В ответ приходит  `json-объект` с данными:
  ```json
  {   "Total" : 2,  
      "Items":[
    {
        "name":"Магазин №1",
        "address":" Ленина 1 стр 1",
        "working_hours":"8:00-20:00",
        "lon":"12.554764",
        "lat":"12.554564",
        "guid": "b5616a82-4637-4175-8d47-f27572a08c8e"


    },
    {
        "name":"Магазин №2",
        "address":" Ленина 2 стр 2",
        "working_hours":"8:00-20:00",
        "lon":"39.643151",
        "lat":"-0.351560",
        "guid": "7c294f0f-ea56-4928-8c53-68930384128a"
    }] 
  }
  ```

#### получение всех продуктов по категории
  выдает все активные продукты (у которых IsActive == true) 
`GET /api/v1/products_by_category` — все получить продуктыпо категории
  Параметры
  
  | название |    тип     | обязательное |
  | -------- | :--------: | :----------: |
  | category_id   | `string` |      да      |

  Ответ аналогичен ответу `GET /api/v1/products`


#### получение всех продуктов по магазину
  выдает все активные продукты (у которых IsActive == true)
  `GET /api/v1/products_by_shop` — все получить продукты по guid магазина
  Параметры

  | название |    тип     | обязательное |
    | -------- | :--------: | :----------: |
  | shop_guid   | `string` |      да      |
  
  Ответ аналогичен ответу `GET /api/v1/products`

#### Добавление продукта
   
   Внимание! Категория и Магазин уже должны быть созданы
   
    `POST /api/v1/product` - Добавляет продукт
    ```json
    {
    "sku": "2075",
    "name": "Кирпич 2",
    "uri": "/product/brick-2075",
    "description": "Кирпич 2", 
    "shop_guid": "7c294f0f-ea56-4928-8c53-68930384128a",
    "category_id": "2480"
    }
    ```
    
    В ответ приходит либо сообщение об ошибке, либо JSON-объект с данными:
    
    ```json
    { 
      "Total" : 1,
      "Items": [{
      "sku": "2075",
      "name": "Кирпич 2",
      "uri": "/product/brick-2075",
      "description": "Кирпич 2"
      }],
      "status": "ok"
    }
    ```

    ```json
    {
      "error": "что то пошло не так",
      "status": "error"
    }
    ```

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
      "Total" : 1,
      "Items": [{
      "sku": "2075",
      "name": "Кирпич 2",
      "uri": "/product/brick-2075",
      "description": "Кирпич 2",
      "is_active": "False"
      }],
      "status": "ok"
    }
    ```

    ```json
    {
      "error": "что то пошло не так",
      "status": "error"
    }
    ```
    Аналогично делаем добавление и изменение магазинов, категорий и цен 

### Этап 3. Возможные внешние зависимости

* Необходимо предусмотреть возможность нашего сервиса обращаться в другие сервисы (например сервис доступности или актуальных цен)
  для этого необходимо использовать механизм gRPS (внешние сервисы необходимо замокать)
  
# Запуск приложения

Для запуска приложения в корне проекта должны находиться следующие файлы:

- `Dockerfile` для сборки образа приложения;
- `docker-compose.yaml` со сервисом приложения и сервисом postgres;
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

* Попробовать найти ответ сначала самому в интернете. Ведь, именно это скилл поиска ответов пригодится тебе на первой работе. И только после этого спрашивать дипломного руководителя
* В одном вопросе должна быть заложена одна проблема
* По возможности, прикреплять к вопросу скриншоты и стрелочкой показывать где не получается. Программу для этого можно скачать здесь https://app.prntscr.com/ru/
* По возможности, задавать вопросы в комментариях к коду.
* Начинать работу над дипломом как можно раньше! Чтобы было больше времени на правки.
* Делать диплом по-частям, а не все сразу. Иначе, есть шанс, что нужно будет все переделывать :)

**Что следует делать, чтобы ничего не получилось:**

* Писать вопросы вида “Ничего не работает. Не запускается. Всё сломалось.”
* Откладывать диплом на потом.
* Ждать ответ на свой вопрос моментально. Дипломные руководители - работающие разработчики, которые занимаются, кроме преподавания, своими проектами. Их время ограничено, поэтому постарайтесь задавать правильные вопросы, чтобы получать быстрые ответы! 
