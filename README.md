# Интеграция сервиса Salesbeat

## Инизиализируем checkout

0) Необходимо запросить `CLIENT_TOKEN`/`SECRET_TOKEN` для вашего магазина, написав на `hi@salesbeat.pro`, а также указать `endpoint` для приёма данных заказа (подробнее в п.5).

1) В html-код магазина необходимо добавить загрузку `sdk`:

```html
<head>
...
<script src="https://cdn.salesbeat.pro/checkout-sdk.js">
...
</head>
```

2) После добавления товара в корзину необходимо вызвать метод (вызов необходимо сделать на стороне сервера) `https://checkout.salesbeat.pro/api/create_cart?secret_token=<SECRET_TOKEN>` и передать в него данные корзины (опционально и данные клиента — `customer_info`):

```json
{
    "cart_info": {
        "shop_cart_id": "<cart-id-in-your-shop>",
        "products": [{
            "id": "011",
            "name": "Гусли",
            "quantity": 1,
            "price": 999,
            "old_price": 1999,
            "lot": 1,
            "max_quantity": 1000,
            "min_quantity": 1,
            "image_url": "https://muzmart.com/files/user/%D0%A1%D1%82%D0%B0%D1%82%D1%8C%D0%B8/%D0%93%D1%83%D1%81%D0%BB%D0%B8/%D0%B3%D1%83%D1%81%D0%BB%D0%B8%20%D0%B3%D0%BB%D0%B0%D0%B2%D0%BD%D0%B0%D1%8F.png",
            "dimensions": {
                "x": 10,
                "y": 9,
                "z": 8
            },
            "weight": 200,
            "features": [{
                "name": "размер",
                "value": "большущие"
            }]
        }]
    },
    "customer_info": {
        "shop_customer_id": "<customer-id-in-your-shop>",
        "first_name": "Вася",
        "last_name": "Пупкин",
        "middle_name": "Иваныч",
        "phone": "74959999999",
        "email": "vasya@pupkinhead.com"
    }
}
```

В ответ данный метод вернёт:

```json
{
  "cart_info": {
    "shop_cart_id": null,
    "products": [
      {
        "id": "123",
        "name": "Sandali",
        "quantity": 1,
        "price": 1231.12,
        "old_price": null,
        "lot": 1,
        "max_quantity": 10000,
        "min_quantity": 1,
        "image_url": "https://holower.ru/goods/sandali.jpg",
        "dimensions": {
          "x": 12,
          "y": 15,
          "z": 19
        },
        "weight": 120,
        "features": []
      }
    ],
    "cart_id": "CART_ID"
  },
  "customer_info": {
    "customer_id": null,
    "shop_customer_id": null,
    "first_name": null,
    "last_name": null,
    "middle_name": null,
    "phone": null,
    "email": null
  }
}
```

В данном ответе нам понадобится значение `CART_ID` — `cart_info.cart_id`


3) Когда мы получили `CART_ID` — можем приступать к инициализации checkout-а:

```javascript
const salesbeat = new window.Salesbeat('<CART_ID>', '<CLIENT_TOKEN>');
```

4) В момент, когда пользователь желает перейти к оформлению заказа остаётся вызвать лишь один метод и осчастливить его:

```javascript
salesbeat.openCheckout();
```

5) После оформления заказа — salesbeat пришлёт запрос на вебхук со всеми данными в магазин по необходимому адресу (указанному в п.0). Формат данных:

```json
{
  "first_name": "Вася",
  "last_name": "Пупкин",
  "middle_name": "Иваныч",
  "phone": "74959999999",
  "email": "vasya@pupkinhead.com",
  "shop_cart_id": "<cart-id-in-your-shop>",
  "products": [
    {
      "id": "011",
      "name": "Гусли 222",
      "quantity": 1,
      "price": 999,
      "old_price": 1999,
      "lot": 1,
      "max_quantity": 10000,
      "min_quantity": 1,
      "image_url": "https://muzmart.com/files/user/%D0%A1%D1%82%D0%B0%D1%82%D1%8C%D0%B8/%D0%93%D1%83%D1%81%D0%BB%D0%B8/%D0%B3%D1%83%D1%81%D0%BB%D0%B8%20%D0%B3%D0%BB%D0%B0%D0%B2%D0%BD%D0%B0%D1%8F.png",
      "dimensions": {
        "x": 10,
        "y": 9,
        "z": 8
      },
      "weight": 200,
      "features": [
        {
          "name": "цвет",
          "value": "красный"
        }
      ]
    }
  ],
  "delivery": {
    "city_code": "0c5b2444-70a0-4932-980c-b4dc0d3f02b5",
    "delivery_id": "boxberry_courier",
    "delivery_method_name": "Boxberry",
    "delivery_method_id": "boxberry_courier",
    "delivery_price": 270,
    "delivery_days": 1,
    "pvz_id": null,
    "pvz_address": null,
    "comment": ""
  },
  "payment": {
    "payment_id": "cash",
    "payment_price": 1269
  }
}
```

В ответе на данный вебхук ожидается формат данных:

```json
{
    "callback": "<uri-to-payment-or-thank-you-page>"
}
```

Где в поле `callback` — `URI`, куда направить пользователя после завершения оформления заказа (это может быть страница "спасибо за заказ" или страница с оплатой).


## Изменение состава корзины:

Если вдруг состав корзины изменился — необходимо:

1) Вызвать на стороне сервера `https://checkout.salesbeat.pro/api/update_cart/<CART_ID>?secret_token=<SECRET_TOKEN>`, передав туда данные корзины:

```json
{
    "cart_info": {
        "shop_cart_id": "<cart-id-in-your-shop>",
        "products": [{
            "id": "011",
            "name": "Гусли",
            "quantity": 1,
            "price": 999,
            "old_price": 1999,
            "lot": 1,
            "max_quantity": 1000,
            "min_quantity": 1,
            "image_url": "https://muzmart.com/files/user/%D0%A1%D1%82%D0%B0%D1%82%D1%8C%D0%B8/%D0%93%D1%83%D1%81%D0%BB%D0%B8/%D0%B3%D1%83%D1%81%D0%BB%D0%B8%20%D0%B3%D0%BB%D0%B0%D0%B2%D0%BD%D0%B0%D1%8F.png",
            "dimensions": {
                "x": 10,
                "y": 9,
                "z": 8
            },
            "weight": 200,
            "features": [{
                "name": "размер",
                "value": "большущие"
            }]
        }]
    },
    "customer_info": {
        "shop_customer_id": "<customer-id-in-your-shop>",
        "first_name": "Вася",
        "last_name": "Пупкин",
        "middle_name": "Иваныч",
        "phone": "74959999999",
        "email": "vasya@pupkinhead.com"
    }
}
```

2) После получения ответа HTTP-запроса из п.1 — нужно на клиенте вызвать метод `refreshCheckout` — чтобы обновить состав корзины:

```javascript
salesbeat.refreshCheckout();
```


## Если вдруг возникнут проблемы/вопросы — `hi@salesbeat.pro`
