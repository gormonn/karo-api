# Оглавление
* [Порядок запросов](#Порядок-запросов)
* [Обзор мест](#Обзор-мест)
    * [Расположение экранов](##Расположение-экранов)
    * [Категории области](##Категории-области)
    * [Распределение мест](##Распределение-мест)
    * [Стили / Типы сидений](##Стили-/-Типы-сидений)
    * [Правила выбора места](##Правила-выбора-места)
* [Данные сиденья](#Данные-сиденья)
    * [Данные сессии OData](##Данные-сессии-OData)
    * [Данные плана мест](##Данные-плана-мест)
    * [Структурированные данные о расположении сидений](##Структурированные-данные-о-расположении-сидений)
    * [GetSessionSeatPlan](##GetSessionSeatPlan)
    * [GetSessionSeatData](##GetSessionSeatData)
    * [AddTickets](##AddTickets)

# Порядок запросов
См. [Порядок запросов](http://testvistaweb.karofilm.ru/WSVistaWebClient/api-docs/creating-orders/order-overview) для заказа билетов.

# Обзор мест
[Ссылка на оригинал](http://testvistaweb.karofilm.ru/WSVistaWebClient/api-docs/seating/overview)

Размещение (или отображение и выбор отдельных мест) является ключом к процессу оформления билетов. Как таковое понимание концепций и API в этой области необходимо, прежде чем создавать клиентское приложение для создания билетов.

## Расположение экранов
* Каждый *кинотеатр* в Vista имеет несколько *экранов* (театров/зрительных залов)
* Каждый *экран* может иметь несколько различных *схем расположения сидений* (конфигураций)
* В результате каждый *сеанс* имеет уникальный экран отображения мест `As a result each session has a unique seating display and as such seating APIs are session related rather than screen-related`

## Категории области
* Каждый *макет экрана* состоит из нескольких различных областей, известных как *категории областей*
* Билеты связаны с одной *категорией зоны*, это означает, что только определенные билеты доступны для определенных мест в *сеансе*
* Это важно понимать при создании клиентского интерфейса, так как он должен ограничивать билеты и/или места на основе предыдущих выборов клиента `This is important to understand when creating client UI as it must restrict tickets and/or seats based on the customer's previous selections`

## Распределение мест
* Для отдельных *сессий* распределение мест может быть включено или отключено. При отключении клиенты не смогут выбирать места как часть бронирования. Это *значит* кто первый пришел, того и тапки. `Individual sessions can have seat allocation enabled or disabled. When disabled customers will not be able to select seats as part of the booking. It will be first-come first-served at the cinema.`
* Клиенты должны учитывать `IsAllocatedSeating` свойство *сеанса* в API и пропускать выбор мест, если это не применимо.
* Сеанс может или не может быть настроен для автоматического распределения мест.
    * Если автоматическое распределение не происходит, клиент ДОЛЖЕН выбрать места вручную.
    * Если автоматическое распределение происходит, но происходит сбой из-за ограничений в алгоритме распределения, клиент ДОЛЖЕН выбрать места вручную.
    * Если автоматическое распределение происходит без ошибок, то клиент МОЖЕТ пропустить выбор места, но обычно отображается пользовательский интерфейс, позволяющий ему просматривать / изменить назначенные места

## Стили / Типы сидений
Vista поддерживает несколько разных мест, которые ведут себя немного по-разному. Обычно компоненты пользовательского интерфейса выбора места обрабатывают правила вокруг них.

Тип | Описние
-|-
Нормальный | Обычное одно место
Диван | Сиденья дивана сгрупированны вместе, чтобы представить любое расположение сидений для нескольких человек. Зачастую покупатели покупают билеты на весь диван, а не на его часть
Места для инвалидов | Представляют из себя места, зарезервированные в театре для тех, кто находится в инвалидной коляске.
Сиденье-компаньон | Они представляют собой места, расположенные рядом с местами для инвалидов и зарезервированные для спутников инвалида-колясочника. Часто требуется, чтобы они могли быть выбраны только тогда, когда также выбрано связанное место для инвалидов

## Правила выбора места
Правила о том, как и когда можно зарезервировать места, легко настраиваются в продающих приложениях Vista, включая такие правила, как:
* Клиент не может оставить 1 место между местами
* Клиент не может оставить 1 место между сиденьем и проходом
* Клиент не может выбрать сопутствующее сиденье без кресла-коляски
* и т.п.

Важно отметить, что эта логика применяется только на уровне пользовательского интерфейса. Connect не будет применять какие-либо правила в отношении типов мест или мест.

# Данные сиденья
[Ссылка на оригинал](http://testvistaweb.karofilm.ru/WSVistaWebClient/api-docs/seating/seat-data)

Как правило, существует два типа информации о сидении, которая нужна клиенту:
* Фактическое расположение мест в сеансе, включая категории площадей и отдельные места в них. Это используется для визуализации карты мест в пользовательском интерфейсе.
* Текущий статус сессии с точки зрения наличия мест: от количества мест на высоком уровне, оставшихся в сеансе, до статуса проданного места на низком уровне для отдельных мест

API Connect предоставляет эту информацию через набор API, которые описаны ниже.

## Данные сессии OData

Сессия возвращает число сидячих связанных свойств.

**Важное примечание:**

Эти данные, особенно `SeatsAvailable` данные, НЕ являются данными реального времени и могут быть в разной степени устаревшими в зависимости от внутренней конфигурации и конфигурации кэширования Connect.
```
{
    "ID": "0000000001-51592",
    "CinemaId": "0000000001",
    "SessionId": "51592",
    "AreaCategoryCodes": [
        "0000000001",
        "0000000002"
    ],
    "Showtime": "2019-05-03T18:00:00",
    "IsAllocatedSeating": true,
    "SeatsAvailable": 251,
    "ScreenName": "ABC Screen 1",
    "ScreenNumber": 1,
    "SoldoutStatus": 0,
}
```

## Данные плана мест
Существует несколько способов получения данных о плане мест в зависимости от потребностей конкретного клиента. Каждый возвращает по существу одну и ту же информацию, но достоинства в их поведении заслуживают понимания.

**Важная заметка**

Сложность *плана места* была развита за эти годы, и эти API могут быть довольно детальными. Для современных клиентов важно следовать передовой практике при запросе планов мест:
* Если доступно, укажите *структурированный* `SetDataFormat` (примеры ниже). Альтернативой является устаревшая закодированная строка, и если не указать ни одну из них, будет возвращено и то и другое, что приведет к снижению производительности на сервере
* Если доступно, включите все `Include***` свойства. Они существуют по определенным причинам совместимости, и все современные клиенты должны иметь дело со всеми данными, которые могут быть конфигрурированны в схеме *расположения рабочих мест*.

## Структурированные данные о расположении сидений
Ниже приведен пример данных о плане / расположении кресел.

Он состоит из смеси:
* Физаческая планировка сидений
* Физические размеры, которые можно использовать для рисования карты мест
* Количество `SeatStyle` отдельных мест (например, нормальное, диван, кресло-коляска и т.д.)
* Количество `Status` отдельных мест (например, *продано*, *зарезервировано* и т.д.)
Пожалуйста, ознакомьтесь со [справочной документацией API](http://testvistaweb.karofilm.ru/WSVistaWebClient/api-docs/api-reference/v1#!/RESTData/RESTData_GetSessionSeatPlanByCinemaidAndSessionid) для `SeatLayoutData` уточнения.

```
{
    "SeatLayoutData": {
        "Areas": [
            {
                "Number": 1,
                "AreaCategoryCode": "0000000001",
                "Description": "Stalls",
                "DescriptionAlt": "",
                "NumberOfSeats": 2,
                "IsAllocatedSeating": true,
                "Left": 0.2,
                "Top": 29.33,
                "Height": 18.83,
                "Width": 99.8,
                "Rows": [
                    {
                        "PhysicalName": "A",
                        "Seats": [
                            {
                                "Position": {
                                    "AreaNumber": 1,
                                    "RowIndex": 0,
                                    "ColumnIndex": 1
                                },
                                "Priority": 2,
                                "Id": "1",
                                "Status": 0,
                                "SeatStyle": 1,
                                "OriginalStatus": 0
                            },
                            {
                                "Position": {
                                    "AreaNumber": 2,
                                    "RowIndex": 0,
                                    "ColumnIndex": 3
                                },
                                "Priority": 2,
                                "Id": "2",
                                "Status": 0,
                                "SeatStyle": 1,
                                "OriginalStatus": 0
                            }
                        ]
                    }
                ],
                "RowCount": 1,
                "ColumnCount": 2
            }
        ],
        "AreaCategories": [
            {
                "AreaCategoryCode": "0000000001",
                "SeatsToAllocate": 0,
                "SeatsAllocatedCount": 0,
                "SeatsNotAllocatedCount": 0,
                "SelectedSeats": [],
                "IsInSeatDeliveryEnabled": false
            }
        ],
        "BoundaryRight": 100,
        "BoundaryLeft": 0,
        "BoundaryTop": 29.33,
        "ScreenStart": 0,
        "ScreenWidth": 100
    },
    "ResponseCode": 0,
    "ErrorDescription": null
}
```

## GetSessionSeatPlan
Эта конечная точка не имеет никакого отношения к любому заказу и может быть использована для отображения текущего состояния сеанся в виде карты или чаще для *сидения-первых* стиле билетов UIs, где клиент должен сначала выбрать места в виде *карты мест*, прежде чем выбрать билеты.

**Важное примечание:**

Эта конечная точка (благодаря включению текущих статусов мест) извлекает свои данные из кинотеатра, что увеличивает нагрузку на приложения Connect и нисходящие потоки в кинотеатре. Это так же конечная точка, которая обычно находится под высокой нагрузкой. Таким образом, большинство экземпляров Connect будут иметь определенный уровень краткосрочного кэширования данных. Как таковой он не может рассматриваться в *режиме реального времени*, но должен быть *близок к реальному времене* и более актуален, чем данные *сеанса*.

>GET: /restdata.svc/cinemas/ndomcinemaId‹/sessions/ndomsessionIdcasts/seat-plan

Ответ:
```
{
    "SeatLayoutData": {},
    "ResponseCode": 0,
    "ErrorDescription": null
}
```

## GetSessionSeatData
Эта конечная точка возвращает тот же ответ, что и **GetSessionSeatPlan**, но разработана специально для сценариев размещения *билетов в первую очередь*, когда клиент уже выбрал билеты.
* Эта конечная точка НЕ имеет кеширования и МОЖЕТ рассматриваться в *режиме реального времени*, поскольку каждый раз получает данные непосредственно из кинотеатра.
* Эта конечная точка имеет ряд опций для исключения определенных свойств сидений, а также возможность возврата заказа вместе с планом сидений.
* Свойство `SeatDataFormat` является ключевым и всегдадолжно иметь значение 2 (*структурированное*)
* Эта конечная точка, поскольку связана с конкретным заказом, вернет места, назначенные текущему заказу, со статусом *зарезервированного*, тогда как они будут помечены как доступные в [`GetSessionSeatPlan`](##GetSessionSeatPlan)
* Эта конечная точка потерпит неудачу, если в заказ не было добавлено ни одного билета

> POST: /restticketing.svc/order/seats/get
Запрос:
```
{
  "CinemaId": "{{cinemaId}}",
  "SessionId": "{{sessionId}}",
  "UserSessionId": "{{userSessionId}}",
  "ReturnOrder": true,
  "ExcludeAreasWithoutTickets": false,
  "IncludeBrokenSeats": true,
  "IncludeHouseSpecialSeats": true,
  "IncludeGreyAndSofaSeats": true,
  "IncludeAllSeatPriorities": true,
  "IncludeSeatNumbers": true,
  "IncludeCompanionSeats": true,
  "SeatDataFormat": true
}
```
Ответ:
```
{
    "Result": 0,
    "Order": {},
    "SeatLayoutData": {},
    "SeatsNotAllocated": false,
    "ErrorDescription": null
}
```
## AddTickets
**AddTickets** конечная точка имеет свойство `ReturnSeatData`, а также связанные с ними свойства, аналогичные [`GetSessionSeatData`](##GetSessionSeatData). Указав это, вы получите те же данные о расположении кресел, что и у этой конечной точки.
