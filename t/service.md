# Заказ и оформление услуг

Для заказа услуг используется метод ```POST /services/orders```, куда в body передается:
| Поле | Описание | Варианты | Стандартное значение |
|------------------------------------------------------|--------------------------------| --- | --- |
| count | количество услуг к заказу | integer | 1 |
| term | срок продления услуги | hour, month и т.д. |  |
| name | имя новой услуги | string |  |
| productId | ID тарифа (можно получить в ```GET /products``` | integer | |
| parameters | параметры услуги | индивидуально для каждого типа услуг. Подробнее, ниже. |  |
| autoProlong | Автопродление | boolean | true |
| method | Метод оплаты (```GET /payment/methods```) или ```balance``` для оплаты с баланса | string |  |

Пример заказа EPs-1 на час без автопродления с ОС Ubuntu 20.04:
```
// Откройте консоль браузера в ЛК и введите
this.api.query('POST', 'services/orders', {
  count: 1,
  term: 'hour',
  name: 'i love api',
  productId: 3,
  parameters: {
    os: 25
  },
  autoProlong: false,
  method: 'balance'
});
```

Параметры услуги можно получить в поле ```ServiceType.computedParameters``` (```GET /service/types```)

### Более подробно про типы услуг и их параметры
* [Домены](/t/types/domain.md)
* [WAF-Защита](/t/types/waf.md)

## Управление услугой
Для управления услугой используется ```POST /services/:ID/ctl```:
| Поле | Описание | Варианты | Стандартное значение |
|------------------------------------------------------|--------------------------------| --- | --- |
| action | действие | suspend, resume, reboot |  |

После этого вернется объект услуги.
Полное описание всех полей (поля, говорящие за себя не описаны):
| Поле | Описание |
|------------------------------------------------------|--------------------------------|
| payload | Полезная нагрузка услуги и её обработчика. Используется для хранения значений ради индивидуальных случаев. |
| secureParameters | Конфиденциальные данные (могут быть зашифрованы с помощью aes-256-ctr (алгоритм дешифрации ниже)
| status | статичное состояние услуги |
| action | переходное состояние услуги (включение/выключение...) |
| currentStatus | вернет action, либо status при отсутствии action. Полезно для отображения состояния услуги в текущий момент. |
| specialPrices | индивидуальные цены (выставляются администратором) |
| relativePrices | индивидуальные относительные цены (выставляются администратором) |
| lastStatus | последний статус перед выключением услуги (для продления) |
| timestamps | временные метки жизненного цикла |

Дешифрация данных:
```
const pincode = '1337'; // Пинкод
const key = await scrypt.scrypt(Buffer.from(pincode), Buffer.from('rabbit-billing'), 16, 8, 1, 32);
const decipher = crypto.createDecipheriv('aes-256-ctr', key, Buffer.from(service.secureParameters.iv, 'hex'));
try {
  const decrpyted = Buffer.concat([
    decipher.update(Buffer.from(this.service.secureParameters.content, 'hex')),
    decipher.final(),
  ]);
  
  const data = JSON.parse(decrpyted.toString());
  // Используем data как хотим
} catch (err) {}
```

Вы можете изменить имя услуги: ```PUT /services/:ID { "name": "i ochen' love api" }```
Или удалить услугу без возврата средств: ```DELETE /services/:ID```
