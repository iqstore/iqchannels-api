IQChannels API
==============

Содержание
----------
* [API](#api)
    * [Отправка запросов](#Отправка-запросов)
    * [Авторизация клиентов](#Авторизация-клиентов)
    * [Работа с чатами](#Работа-с-чатами)
    * [Работа с операторами чата](#Работа-с-операторами-чата)
    * [Работа с сообщениями чата](#Работа-с-сообщениями-чата)
    * [Работа с входящими оператора](#Работа-с-входящими-оператора)
    * [Работа с событями чатов](#Работа-с-событями-чатов)
* [Схема данных](#Схема-данных)
    * [General types](#general-types)
    * [Response](#response)
    * [Actors](#actors)
    * [Channels](#channels)
    * [Chats](#chats)
    * [Chat events](#chat-events)
    * [Chat inboxes](#chat-inboxes)
    * [Chat messages](#chat-messages)
    * [Clients](#clients)
    * [Files](#files)
    * [Rels](#rels)
    * [Users](#users)


API
===
Внутренее API, предназначенное для создания панели управления, рабочего места оператора, ботов.

Отправка запросов
-----------------

### Обычные запросы

Все запросы к серверу IQChannels передаются, как HTTP POST запросы, в теле которых
JSON-строка. Пример:
```http
POST /api/v1/channels/query HTTP/1.1
Host: demo.iqchannels.ru
Connection: keep-alive
Content-Length: 27
Origin: http://demo.iqchannels.ru
Content-Type: application/json
Accept: */*
Referer: http://demo.iqchannels.ru/chats/inbox
Accept-Encoding: gzip, deflate

{"Limit": 25, "Offset": 50}
```

В ответ сервер всегда возвращает `HTTP 200 OK`. Если иной статус ответа, значит
произошла ошибка вне IQChannels, например, сетевая или любая другая.
В теле ответа содержится структура [Response](#response) в виде JSON-строки.

Пример ответа:
```http
{
  "OK": true,
  "Error": null,
  "Result": [
    {
      "Id": 2,
      "Name": "telegram",
      "Type": "telegram",
      "Title": "Телеграм",
      "Description": "Поддержка клиентов в Телеграме",
      "Deleted": false,
      "EventId": 127,
      "ChatEventId": 16450,
      "TotalMembers": 1,
      "TotalChats": 3,
      "TotalOpenChats": 0,
      "TelegramBotToken": "318929672:AAH1zVdZP9GIBevYeJrP2Tp2BXkeo1j0mpI",
      "ApnsId": null,
      "CreatedAt": 1485945188465,
      "UpdatedAt": 1486115201505
    }
  ],
  "Rels": {}
}
```


### Realtime-запросы

Для получения обновленных данных от сервера используются два тип подключений:
HTML5 Server Sent Events и Websockets. Первые просты в реализации и предназначены 
для мобильных телефонов, ботов, вторые для браузеров.

В обоих случаях сервер присылает последовательность объектов [Response](#response) 
либо как HTML5 SSE-события, либо как Websocket-сообщения. Пример сообщения/события:

```json
{
  "OK": true,
  "Error": null,
  "Result": [
    {
      "Id": 16509,
      "Type": "chat_opened",
      "ChatId": 1,
      "Public": false,
      "Transitive": false,
      "SessionId": 40,
      "MessageId": null,
      "MemberId": null,
      "Actor": "user",
      "ClientId": null,
      "UserId": 1,
      "CreatedAt": 1488797464637
    }
  ],
  "Rels": {
    "Channels": [
      {
        "Id": 1,
        "Name": "support",
        "Type": "general",
        "Title": "Физические лица",
        "Description": "Техническая поддержка физических лиц",
        "Deleted": false,
        "EventId": 133,
        "ChatEventId": 16509,
        "TotalMembers": 7,
        "TotalChats": 10,
        "TotalOpenChats": 1,
        "TelegramBotToken": null,
        "ApnsId": 1,
        "CreatedAt": 1482148439205,
        "UpdatedAt": 1486113042329
      }
    ],
    "Clients": [
      {
        "Id": 1,
        "Type": "integrated",
        "Name": "Мартин Лютер Кинг",
        "Online": false,
        "IntegrationId": "1",
        "TelegramId": null,
        "CreatedAt": 1484233255366,
        "UpdatedAt": 1485175749359,
        "LastSeenAt": 1488366344160
      }
    ],
    "Chats": [
      {
        "Id": 1,
        "ChannelId": 1,
        "ClientId": 1,
        "Open": true,
        "EventId": 16513,
        "MessageId": 5402,
        "SessionId": 40,
        "AssigneeId": null,
        "ClientUnread": 5,
        "UserUnread": 0,
        "TotalMembers": -5,
        "CreatedAt": 1484233263296,
        "ChangedAt": 1488797464635,
        "OpenedAt": 1488797464635,
        "ClosedAt": 1488456651514,
        "CurrentMemberId": 1
      }
    ]
  }
}
```

Авторизация клиентов
--------------------
В первый раз клиенту требуется авторизоваться через внешнюю для IQChannels систему,
например, залогиниться в интернет-банке или в мобильном банке. После этого клиенту
нужно передать какой-либо авторизационный токен в IQChannels. Последний может быть
токеном сессии или любой другой сторокой. 

Этот токен IQChannels передаст во внешнюю систему авторизации клиента и получит
информацию о клиенте.

| Запрос | Авторизация клиента с токеном внешней системы
| --------- | ------- 
| Путь    | `/api/v1/clients/integration_auth`
| Запрос  | [ClientIntegrationAuthRequest](#clientintegrationauthrequest)
| Результат | [ClientAuth](#clientauth)


| Запрос | Авторизация клиента с токеном сессии IQChannels
| --------- | ------- 
| Путь    | `/api/v1/clients/auth`
| Запрос  | [ClientAuthRequest](#clientauthrequest)
| Результат | [ClientAuth](#clientauth)


Работа с чатами
---------------
| Запрос | Получение чата по айди
| --------- | ------- 
| Путь    | `/api/v1/chats/get/:chatId`
| Запрос  | Пустой
| Результат | [Chat](#chat)


| Запрос | Получение чатов в канале
| --------- | ------- 
| Путь    | `/api/v1/chats/query_by_channel/:channelId`
| Запрос  | [ChatQuery](#ChatQuery)
| Результат | list<[Chat](#chat)>


| Запрос | Открытие чата
| --------- | ------- 
| Путь    | `/api/v1/chats/open/:chatId`
| Запрос  | Пустой
| Результат | [Chat](#chat)


| Запрос | Закрытие чата
| --------- | ------- 
| Путь    | `/api/v1/chats/get/:chatId`
| Запрос  | Пустой
| Результат | [Chat](#chat)


Работа с операторами чата
-------------------------

| Запрос | Войти в чат
| --------- | ------- 
| Путь    | `/api/v1/chats/join/:chatId`
| Запрос  | Пустой
| Результат | [ChatMember](#chatmember)

| Запрос | Выйти из чата
| --------- | ------- 
| Путь    | `/api/v1/chats/leave/:chatId`
| Запрос  | Пустой
| Результат | Пустой

| Запрос | Приглашение оператора в чат
| --------- | ------- 
| Путь    | `/api/v1/chats/invite/:chatId`
| Запрос  | `{UserId: userId}`
| Результат | [ChatMember](#chatmember)

| Запрос | Назначение чата на оператора
| --------- | ------- 
| Путь    | `/api/v1/chats/assign/:chatId`
| Запрос  | `{UserId: userId}`
| Результат | [Chat](#chat)

| Запрос | Передача чата другому оператору
| --------- | ------- 
| Путь    | `/api/v1/chats/transfer/:chatId`
| Запрос  | `{UserId: userId}`
| Результат | [Chat](#chat)

| Запрос | Получение участников чата
| --------- | ------- 
| Путь    | `/api/v1/chats/members/query_by_chat/:chatId`
| Запрос  | [ChatMemberQuery](#chatmemberquery)
| Результат | list<[ChatMember](#chatmember)>


Работа с сообщениями чата
-------------------------

| Запрос | Отправка состояние, что пользователь печатает в чате
| --------- | ------- 
| Путь    | `/api/v1/chats/typing/:chatId`
| Запрос  | Пустой
| Результат | Пустой

| Запрос | Отправка сообщения в чат
| --------- | ------- 
| Путь    | `/api/v1/chats/send/:chatId`
| Запрос  | [ChatMessageForm](#chatmessageform)
| Результат | Айди сообщения, информация об отправленном сообщении может быть получена с помощью realtime-api.

| Запрос | Получение сообщений в чате
| --------- | ------- 
| Путь    | `/api/v1/chats/messages/query_by_chat/:chatId`
| Запрос  | [ChatMessageQuery](#chatmessagequery)
| Результат | [Chat](#chat)

| Запрос | Отметить сообщения как доставленные
| --------- | ------- 
| Путь    | `/api/v1/chats/messages/received`
| Запрос  | `[messageId0, messageId1, messageIdN]`
| Результат | Пустой

| Запрос | Отметить сообщения как прочитанные
| --------- | ------- 
| Путь    | `/chats/messages/read`
| Запрос  | `[messageId0, messageId1, messageIdN]`
| Результат | Пустой


Работа с входящими оператора
----------------------------

| Запрос | Получение входящих оператора (объекта входящие)
| --------- | ------- 
| Путь    | `/api/v1/chats/inboxes/get_by_user/:userId`
| Запрос  | Пустой
| Результат | [ChatInbox](#ChatInbox)


| Запрос | Получение тредов во входящих оператора.
| --------- | ------- 
| Путь    | `/api/v1/chats/inbox_threads/query_by_user/:userId`
| Запрос  | [ChatInboxThreadQuery](#chatinboxthreadquery)
| Результат | list<[ChatInboxThread](#ChatInboxThread)>


Работа с событями чатов
-----------------------

### Websocket
| Запрос | Подписка на события в чате
| --------- | ------- 
| Тип     | Websocket
| Путь    | `/api/v1/ws/chats/events/listen_to_chat/:chatId?FromId=:lastEventId`
| Аргументы | `FromId` — последнее полученное айди события, если есть.
| Результат | Websocket-сообщения с list<[ChatEvent](#chatinbox)>

| Запрос | Подписка на события чатов в канале
| --------- | ------- 
| Тип     | Websocket
| Путь    | `/api/v1/ws/chats/events/listen_to_channel/:channelId?FromId=:lastEventId`
| Аргументы | `FromId` — последнее полученное айди события, если есть.
| Результат | Websocket-сообщения с list<[ChatEvent](#chatinbox)>

| Запрос | Подписка на события чатов для оператора
| --------- | ------- 
| Тип     | Websocket
| Путь    | `/api/v1/ws/chats/events/listen_to_user/:userId?FromId=:lastEventId`
| Аргументы | `FromId` — последнее полученное айди события, если есть.
| Результат | Websocket-сообщения с list<[ChatEvent](#chatinbox)>

### HTML5 SSE
| Запрос | Подписка на события в чате
| --------- | ------- 
| Тип     | HTML5 SSE
| Путь    | `/api/v1/sse/chats/events/listen_to_chat/:chatId?FromId=:lastEventId`
| Аргументы | `FromId` — последнее полученное айди события, если есть.
| Результат | Websocket-сообщения с list<[ChatEvent](#chatinbox)>

| Запрос | Подписка на события чатов в канале
| --------- | ------- 
| Тип     | HTML5 SSE
| Путь    | `/api/v1/sse/chats/events/listen_to_channel/:channelId?FromId=:lastEventId`
| Аргументы | `FromId` — последнее полученное айди события, если есть.
| Результат | Websocket-сообщения с list<[ChatEvent](#chatinbox)>

| Запрос | Подписка на события чатов для оператора
| --------- | ------- 
| Тип     | HTML5 SSE
| Путь    | `/api/v1/sse/chats/events/listen_to_user/:userId?FromId=:lastEventId`
| Аргументы | `FromId` — последнее полученное айди события, если есть.
| Результат | Websocket-сообщения с list<[ChatEvent](#chatinbox)>


Схема данных
============
Схема данных IQChannels.

General types
-------------
Общие типы данных.

| Тип       | Значение по умолчанию  | Комментарий
| --------- | ------- | -------
| `bool`    | `false` | Булевое значение.
| `int32`   | `0`     | 32-битное число.
| `int64`   | `0`     | 64-битное число.
| `string`  | `""`    | UTF-8 строка.
| `Timestamp` | `0`   | UTC время в миллисекундах, размер соответствует типу int64.
| `map<K, V>` | `null` | Типизированный словарь ключ-значение.
| `list<E>` | `null`  | Типизированный список.

По умолчанию все примитивные типы данные non-nullable, т.е. если значения нет, 
то используется значение по умолчанию. 

Nullable типы данных отмечаются знаком вопроса в конце, например `int32?`.
По умолчанию эти поля проставляются в null.


Response
--------
Ответ от сервера. 

Если ответ успешен, тогда данные содержат результат `Result` и граф объектов `Rels`, 
на которые ссылаются объекты из результата. Например, если в результате есть
объект с полем `UserId`, то в `Rels.Users` будет пользователь с этим айди. 
Объекты по айди собираются рекурсивно, т.е. для любого поля с айди будет такой же объект в `Rels`.

Если ответ не успешен, тогда данные содержат ошибку `Error`.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `OK`        | `bool`
| `Error`     | [Error](#error)? | Описание ошибки, если запрос не успешен.
| `Result`    | any | Результат запроса.
| `Rels`      | [Rels](#rels)? | Граф зависимых объектов, на которые ссылаются объекты из результата.

Пример:
```json
{
  "OK": true,
  "Error": null,
  "Result": [
    {
      "Id": 55,
      "UserId": 1,
      "ChatId": 1,
      "ChangeId": 556,
      "Open": false,
      "MessageId": 5385,
      "TotalUnread": 0,
      "CreatedAt": 1484233263325,
      "ChangedAt": 1488456651541
    }
  ],
  "Rels": {
    "Channels": [
      {
        "Id": 1,
        "Name": "support",
        "Type": "general",
        "Title": "Физические лица",
        "Description": "Техническая поддержка физических лиц",
        "Deleted": false,
        "EventId": 132,
        "ChatEventId": 16504,
        "TotalMembers": 7,
        "TotalChats": 10,
        "TotalOpenChats": 0,
        "ApnsId": 1,
        "CreatedAt": 1482148439205,
        "UpdatedAt": 1486113042329
      }
    ],
    "Clients": [
      {
        "Id": 1,
        "Type": "integrated",
        "Name": "Мартин Лютер Кинг",
        "Online": false,
        "IntegrationId": "1",
        "TelegramId": null,
        "CreatedAt": 1484233255366,
        "UpdatedAt": 1485175749359,
        "LastSeenAt": 1488366344160
      }
    ],
    "Chats": [
      {
        "Id": 1,
        "ChannelId": 1,
        "ClientId": 1,
        "Open": false,
        "EventId": 16508,
        "MessageId": 5385,
        "SessionId": 38,
        "AssigneeId": null,
        "ClientUnread": 4,
        "UserUnread": 0,
        "TotalMembers": -6,
        "CreatedAt": 1484233263296,
        "ChangedAt": 1488456651514,
        "OpenedAt": 1488442564321,
        "ClosedAt": 1488456651514,
        "CurrentMemberId": 1
      }
    ],
    "Users": [
      {
        "Id": 1,
        "Name": "Иван Коробков",
        "DisplayName": "Иван Коробков",
        "Email": "i.korobkov@iqstore.ru",
        "Online": true,
        "Deleted": false,
        "AvatarId": "942e7550-fe67-11e6-96c2-cf22d6de8c96.jpg",
        "CreatedAt": 1481528996861,
        "LoggedInAt": null,
        "LastSeenAt": 1488789960417
      }
    ]
  }
}
```


### Error
Ошибка приложения.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Code`      | [ErrorCode](#errorcode) | Код ошибки.
| `Text      | `string` | Человеческое описание ошибки.


### ErrorCode
Enum, общие коды ошибок приложения.

| Название  | Значение | Комментарий
| --------- | -------- | -----------
| `ErrorCodeInternalError` | `internal_server_error` | Внутренняя ошибка сервера.
| `ErrorCodeBadRequest` | `bad_request` | Неправильные аргументы запроса.
| `ErrorCodeNotFound` | `not_found` | Объект не найден.
| `ErrorCodeForbidden` | `forbidden` | Доступ запрещен.
| `ErrorCodeUnauthorized` | `unauthorized` | Для доступа требуется авторизация.
| `ErrorCodeInvalid` | `invalid` | Ошибка валидации данных, пользователь ввел неправильные данные.


Actors
------

### ActorType
Enum, тип актора, т.е. субъекта, который совершил действие.

| Название  | Значение
| --------- | --------
| `ActorAnonymous` | Пустая строка
| `ActorClient` | `client`
| `ActorUser`   | `user`
| `ActorSystem` | `system`


Channels
--------

### Channel
Канал поддержки. В рамках организации может быть несколько каналов, например,
для общения с разными типами клиентов: юридическими лицами и физическими лицами.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id`      | `int64` |
| `Name`    | `string` | Название канала на английском языке без пробелов.
| `Type`    | [ChannelType](#channeltype)
| `Title`   | `string` | Название канала на русском языке.
| `Description` | `string` | Описание канала.
| `Deleted`   | `bool` | Флаг, что канал удален.
| `EventId`   | `int64?` | Айди последнего события в канале.
| `ChatEventId` | `int64?` | Айди последнего события в чатах канала.
| | |
| `TotalMembers` | `int32` | Количество операторов в канале.
| `TotalChats` | `int32` | Количество чатов в канале.
| `TotalOpenChats` | `int32` | Количество активных чатов в канале.
| | |
| `CreatedAt` | `Timestamp` | Дата создания.
| `UpdatedAt` | `Timestamp` | Дата обновления.


### ChannelType
Enum, тип канала.

| Название  | Значение
| --------- | --------
| `ChannelTypeGeneral` | Пустая строка
| `ChannelTypeTelegram` | `telegram`
| `ChannelTypeFacebook` | `facebook`
| `ChannelTypeVK` | `vk`
| `ChannelTypeViber` | `viber`


### ChannelMember
Оператор в канале.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id`      | `int64` |
| `ChannelId` | `int64` |
| `UserId` | `int64` | Айди пользователя.
| `Deleted` | `boolean` | Флаг, который показывает, что оператор удален из канала.
| `CreatedAt` | `Timestamp`
| `DeletedAt` | `Timestamp`


Chats
-----

### Chat
Чат поддержки между пользователем и оператором или несколькими операторами.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id`      | `int64`
| `ChannelId` | `int64` | Айди канала.
| `ClientId` | `int64` | Айди клиента.
| `Open`     | `boolean` | Флаг, что чат активен, что у него есть открытая текущая сессия общения.
| | |
| `EventId`   | `int64?` | Айди последнего события в чате.
| `MessageId` | `int64?` | Айди последнего сообщения в чате.
| `SessionId` | `int64?` | Айди последней сессии.
| `AssigneeId` | `int64?` | Айди оператора, на которого назначен чат.
| | |
| `ClientUnread` | `int32` | Количество сообщений, не прочтенных клиентов.
| `UserUnread` | `int32` | Количество сообщений, не прочтенных операторами.
| `TotalMembers` | `int32` | Количество операторов в чате.
| | |
| `CreatedAt` | `Timestamp`
| `ChangedAt` | `Timestamp`
| `OpenedAt` | `Timestamp`
| `ClosedAt` | `Timestamp`


### ChatQuery
Запрос чатов в канале.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Limit`   | `int32?` | Опциональное поле, по умолчанию 50.
| `Offset`  | `int32?` | Опциональное поле для постраничной выдачи.
| `Open`    | `boolean?` | Опциональное поле, флаг для выдачи всех, только открытых, только закрытых чатов.


### ChatMember
Оператор в чате.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id`      | `int64`
| `ChatId`  | `int64`
| `UserId`  | `int64`
| | |
| `Present` | `boolean` | Флаг, что оператор сейчас присутствует в чате.
| `Assigned` | `boolean` | Флаг, что на оператора назначен текущий чат.
| | |
| `CreatedAt` | `Timestamp`
| `AddedAt` | `Timestamp` | Время последнего входа в чат.
| `RemovedAt`| `Timestamp` | Время последнего выхода из чата.
| `AssignedAt` | `Timestamp` | Время, когда на оператора последний раз назначался текущий чат.
| | |
| `Me` | `boolean` | Вычисляемый флаг, указывает, что текущий оператор — это я, т.е. текущий пользователь, который совершает запросы.


Chats Events
------------

### ChatEvent
Событие в чате. Служит для получения обновлений в реальном времени, в т.ч. и обновлений чатов в канале,
обновлений во входящих пользователи, обновлений чатов клиента и отдельного чата.

Событие всегда происходит с одним объектом. Если действите привело к изменению нескольких объектов,
тогда будет несколько событий с разными типами с этими объектами.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id` | `int64`
| `Type` | [ChatEventType](#chateventtype)
| `Transitive` | `boolean` | Показывает, что событие не хранится в базе данных и у него нет айди. Например, `typing`.
| | |
| `ChatId`  | `int64?` 
| `MessageId` | `int64?`
| `MessageId` | `int64?`
| `InboxId` | `int64?`
| `ThreadId` | `int64?`
| | |
| `Actor`   | [ActorType](#actortype)
| `ClientId` | `int64?` | Клиент, который совершил действие.
| `UserId`  | `int64?` | Пользователь, который совершил действие.
| `CreatedAt` | `Timestamp`


### ChatEventType
Enum, тип события в чате.

| Название  | Значение | Комментарий
| --------- | -------- | -----------
| `ChatEventChatCreated` | `chat_created` | Чат создан, первое событие в чате.
| `ChatEventChatChanged` | `chat_changed` | Чат изменился, например, изменилось кол-во непрочтенных сообщений.
| `ChatEventChatOpened` | `chat_opened` | Чат открыт.
| `ChatEventChatClosed` | `chat_closed` | Чат закрыт.
| `ChatEventAssigned` | `chat_assigned` | В чате назначен ответственный оператор.
| `ChatEventUnassigned` | `chat_unassigned` | В чате больше нет ответственного оператора.
| | |
| `ChatEventMemberCreated` | `member_created` | В чате создан оператора, т.е. сам объект [ChatMember](#ChatMember).
| `ChatEventMemberAdded` | `member_added` | В чат вошел оператор.
| `ChatEventMemberRemoved` | `member_removed` | Из чата вышел оператор.
| | |
| `ChatEventTyping` | `typing` | Клиент или пользователь печатает в чате.
| `ChatEventMessageCreated` | `message_created` | Новое сообщение в чате.
| `ChatEventSystemMessageCreated` | `system_message_created` | Новое системное сообщение в чате.
| `ChatEventMessageReceived` | `message_received` | Сообщение доставлено получателю (клиенту или оператору).
| `ChatEventMessageRead` | `message_read` | Сообщение прочтено получателем.
| | |
| `ChatEventUserInboxChanged` | `inbox_changed` | Входящие оператора изменились.
| `ChatEventUserThreadCreated` | `thread_created` | Новый тред во входящих оператора.
| `ChatEventThreadOpened` | `thread_opened` | Тред открыт, т.е. чат треда открыт и в чат добавлен текущий оператора.
| `ChatEventThreadClosed` | `thread_closed` | Тред закрыт, либо текущий оператор вышел из чата, либо чат завершен.
| `ChatEventThreadChanged` | `thread_changed` | Тред изменился.


### ChatEventQuery
Запрос событий чата.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `FromId` | `int64?` | Опциональное поле, айди события, с которого нужно возвращать новые события.
| `Limit` | `int64?` | Опциональное поле.


Chat Inboxes
------------

### ChatInbox
Входящие оператора, состоят из тредов [ChatThread](#chatthread).

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id` | `int64` | Совпадает с айди пользвателя.
| `UserId` | `int64` |
| `EventId` | `int64?` | Последнее событие во входящих оператора.
| | |
| `TotalThreads` | `int32` | Общее количество тредов у оператора.
| `TotalOpenThreads` | `int32` | Количество активных тредов (активных чатов), в которых участвует оператор.
| `TotalAssignedThreads` | `int32` | Количество чатов, которые назначены на оператора.
| | |
| `CreatedAt` | `Timestamp` |
| `ChangedAt` | `Timestamp` |
| `AssignedAt` | `Timestamp` |


### ChatInboxThread
Тред оператора в чате. Тред обеспечивает связь один-ко-многим чата и операторов.
Треды одного оператора объединяются во входящие [ChatInbox](#chatinbox).

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id` | `int64` | 
| `UserId` | `int64` |
| `ChatId` | `int64` |
| `EventId` | `int64?` | Последнее событие треда.
| `Open` | `boolean` | Флаг, что тред открыт, т.е. чат открыт и оператор присутсвует в этот чат.
| | |
| `CreatedAt` | `Timestamp` |
| `ChangedAt` | `Timestamp` |


### ChatInboxThreadQuery
Запрос тредов во входящих оператора.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Limit` | `int32?` | Опциональное поле, по умолчанию 50.
| `Offset` | `int32?` | Опциональное поле для постраничной навигации.
| `Open` | `boolean?` | Флаг, который возвращает все/открытые/закрытые треды.


Chat Messages
-------------

### ChatMessage
Одно сообщение в чате.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id` | `int64` |
| `UID` | `string` | Уникальное айди сообщения в виде 40-char lowercase hex string.
| `ChatId` | `int64` |
| `SessionId` | `int64` |
| `LocalId` | `int64` | Значение, уникальное на стороне отправителя, для предотвращения дубликатов сообщений при повторной отправки.
| `EventId` | `int64?`
| `Public ` | `boolean` | Флаг, что сообщение публичное, т.е. видно клиенту, а не только операторам.
| | |
| `Author` | [ActorType](#actortype) | Тип автора сообщения.
| `ClientId | `int64?` | Клиент, который отправил сообщение.
| `UserId` | `int64?` | Оператор, который отправил сообщение.
| | |
| `Payload` | [ChatPayloadType](#chatpayloadtype) | Тип содержания сообщения.
| `Text` | `string` | Текст сообщение, либо пустая строка, если сообщение не текстовое.
| `FileId` | `string?` | Айди файла, отправленного в чат (файлы умеют текстовые айди).
| `NoticeId` | `int64?` | Айди системного уведомления, отправленного в чат.
| | |
| `Received` | `boolean` | Сообщение доставлено получателю, доставлено клиент на его мобильный, доставлено оператора в браузер.
| `Read` | `boolean` | Сообщение прочитано получателем.
| `Pushed` | `boolean` | Сообщение отправлено через мобильное пуш-уведомление (iOS/Android).
| | |
| `CreatedAt` | `Timestamp` |
| `ReceivedAt` | `Timestamp` |
| `ReadAt` | `Timestamp` |
| | |
| `My` | `boolean` | Вычисляемый флаг, который показывает что это сообщение отправил пользователь/клиент, который сейчас совершает запрос.


### ChatMessageForm
Форма создания нового сообщения.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `LocalId` | `int64` | Уникальное значение на стороне отправителя, рекомендуется использовать время в UTC в миллисекундах. Позволяет в случае ошибки повторно отправлять одно и то же сообщение без дубликатов.
| `Payload` | [ChatPayloadType](#chatpayloadtype) | Тип содержания сообщения.
| `Text` | `string` | Текст сообщения, если содержание — текст.
| `FileId` | `string?` | Айди загруженного файла, если содержание файл или фото. 


### ChatPayloadType
Enum, тип содержания сообщения.

| Название  | Значение | Комментарий
| --------- | -------- | -----------
| `ChatPayloadText` | `text` | Текствое сообщение.
| `ChatPayloadFile` | `file` | Файл или фото.
| `ChatPayloadNotice` | `notice` | Системное уведомление (не присылается клиентам). 


### ChatNotice
Системное уведомление, которое присылается только операторам, но не клиентам.
Например, чат открыт, оператор вошел в чат и т.д.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id` | `int64` | 
| `Type` | [ChatNoticeType](#chatnoticetype) | Тип уведомления.
| `ActorId` | `int64?` | Айди пользователя, который совершил действие в чате.
| `MemberId` | `int64?` | Айди оператора в чате.
| `CreatedAt` | `Timestamp` |


### ChatNoticeType
Тип системного уведомления в чате.

| Название  | Значение  | Комментарий
| --------- | ---- | -------
| `Id` | `int64` |
| `ChatNoticeChatOpened` | `chat_opened` | Чат отрыт.
| `ChatNoticeChatClosed` | `chat_closed` | Оператор завершил чат.
| `ChatNoticeChatAssigned` | `chat_assigned` | Чат назначен на оператора.
| | |
| `ChatNoticeMemberLeft` | `member_left` | Оператор вышел из чата.
| `ChatNoticeMemberJoined` | `member_joined` | Оператор вошел в чат.
| `ChatNoticeMemberInvited` | `member_invited` | Оператор пригласил другого оператора в чат.


### ChatMessageQuery
Запрос сообщений в чате.

| Название  | Значение  | Комментарий
| --------- | ---- | -------
| `MaxId` | `int64?` | Опциональное поле, айди последнего сообщения.
| `Limit` | `int32?` | Опциональное поле, по умолчанию 50.


Clients
-------

### Client
Клиент.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id`      | `int64`
| `Type`    | [ClientType](#clienttype) | Тип клиента. 
| `Name` | `string`
| `Online` | `boolean` | Флаг, что клиент сейчас находится в сети.
| | |
| `IntegrationId` | `string?` | Айди клиента во внешней интеграционной системе.
| `TelegramId` | `int64?` | Айди клиента в Телеграме.
| | |
| `CreatedAt` | `Timestamp` |
| `UpdatedAt` | `Timestamp` |
| `LastSeenAt` | `Timestamp?` | Дата, когда клиент последний раз был в сети.


### ClientAuth
Данные успешной авторизации клиента.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Client`  | [Client](#client) | Информация о клиенте.
| `Session` | [ClientSession](#clientsession) | Сессия авторизация с токеном авторизации.


### ClientSession
Внутренняя сессия авторизации клиента IQChannels. Содержит подписанный зашифрованный токен, который можно использовать
для авторизации клиентских запросов.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id`  | `int64`
| `ClientId` | `int64`
| `Token` | `string` | Подписанный зашифрованный токен авторизации.
| `Integration` | `boolean` | Клиент был авторизован во внешней системе.
| `CreatedAt` | `Timestamp`


### ClientAuthRequest
Запрос авторизации клиента.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Token`  | `string` | Авторизационный токен.


### ClientIntegrationAuthRequest
Запрос авторизации клиента через внешнюю систему.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Credentials`  | `string` | Любая строка, передается as-is во внешнюю систему.


Files
-----

### File
Файл, загруженный клиентом или пользователем на сервер. Единая структура данных
для всех типов файлов, включая документы, картинки и видео.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id`      | `string` | UUID строка с расширением, например, `5722e415-dbc7-11e6-859b-975704fae91d.jpg`.
| `Type`    | [FileType](#filetype)
| `Owner`   | [FileOwnerType](#fileownertype) | Тип владельца файла.
| `OwnerClientId` | `int64?` | Айди клиента, который владеет файлом. 
| | |
| `Actor` | [ActorType](#actortype) | Тип актора, который загрузил файл (пользователь/клиент).
| `ActorClientId` | `int64?` | Айди клиента, который загрузил файл.
| `ActorUserId` | `int64?` | Айди пользователя, который загрузил файл.
| | |
| `Name` | `string` | Оригинальное название файла.
| `Path` | `string` | Относительный путь к файлу на севрере.
| `Size` | `int64` | Размер файла в байтах.
| | |
| `ImageWidth` | `int32?` | Ширина картинки в пикселях, если файл — картинка.
| `ImageHeight` | `int32?` | Высота картинки в пикселя, если файл — картинка.
| | |
| `ContentType` | `string` | Стандартный MIME тип файла (`image/png`, `text/plain` и т.д.)
| `CreatedAt` | `Timestamp` | Дата загрузки файла.


### FileType
Тип файла.

| Название  | Значение | Комментарий
| --------- | -------- | -----------
| `FileTypeFile` | `file` |
| `FileTypeImage` | `image` |


### FileOwnerType
Тип владельца файла.

| Название  | Значение | Комментарий
| --------- | -------- | -----------
| `FileOwnerPublic` | `public` | Публичный файл, например, аватарка оператора.
| `FileOwnerClient` | `client` | Файл, который загружен в чат с клиентом, доступ к таким файлам жестко ограничивается.


### FileImageSize
Размер, который нужно использовать для уменьшения изображения.

| Название  | Значение | Комментарий
| --------- | -------- | -----------
| `FileImageSizeAvatar` | `avatar` | Квадрат 168x168 пикселей.
| `FileImageSizeThumbnail` | `thumbnail` | 128x128 пикселей с сохранением пропорций.
| `FileImageSizePreview` | `preview` | 470x470 пикселей с сохранением пропорций.


Rels
----

### Rels
Объект, который позволяет передавать граф связей других объектов без дублирования информации.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Channels` | list<[Channel](#channel)>
| `Clients` | list<[Client](#client)>
| `Chats` | list<[Chat](#chat)>
| `ChatMembers` | list<[ChatMember](#chatmember)>
| `ChatMessages` | list<[ChatMessage](#chatmessage)>
| `ChatNotices` | list<[ChatNotice](#chatnotice)>
| `ChatInboxes` | list<[ChatInbox](#chatinbox)>
| `ChatInboxThreads` | list<[ChatInboxThread](#chatinboxthread)>
| `Files` | list<[File](#file)>
| `Users` | list<[User](#user)>


Users
-----

### User
Пользователь: администратор, оператор и т.д.

| Поле      | Тип  | Комментарий
| --------- | ---- | -------
| `Id`      | `int64`
| `Name`    | `string` 
| `DisplayName` | `string` | Имя, которое видят клиенты.
| `Email`   | `string`
| `Online`  | `boolean` | Пользватель сейчас в сети.
| `Deleted` | `boolean` | Пользователь удален.
| `AvatarId` | `string?` | Айди файла с аватаркой пользователя.
| | |
| `CreatedAt` | `Timestamp` |
| `LoggedInAt` | `Timestamp` | Дата последнего входа в систему.
| `LastSeenAt` | `Timestamp` | Дата, когда пользователь последний раз был в сети.
