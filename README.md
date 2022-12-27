Корпоративная система

В папке ./build находится frontend-часть проекта, в остальных папках реализован backend на Express.js.
База данных - MongoDB ( ORM - [Mongoose](http://mongoosejs.com/)).
В server.js реализован http-сервер, который на любой get-запрос вернет index.html (маршрутизация выполняется на frontend'e средствами бибилиотеки react-router).
Также на бэкенде реализованы 12 различных запросов: - POST-запрос на `/api/registration` - создание нового пользователя (регистрация). Сигнатура запроса: `{ username, surName, firstName, middleName, password }`. Возвращает объект авторизовавшегося пользователя. - POST-запрос на `/api/login` - авторизация после пользователького ввода. Cигнатура запроса: `{ username, password }` Возвращает объект авторизовавшегося пользователя. - POST-запрос на `/api/refresh-token` - обновление access-токена. В headers['authorization'] - refresh-токен. Возвращает обьект с токенами - GET-запрос на `/api/profile` - авторизация при наличии токена. Возвращает объект пользователя. - PATCH-запрос на `/api/profile` - обновление информации о пользователе.
Сигнатура запроса:
`        {
            firstName: String,
            middleName: String,
            surName: String,
            oldPassword: String,
            newPassword: String,
            avatar: File
        }
       `
Возвращает объект обновленного пользователя. - DELETE-запрос на `/api/users/:id` - удаление пользователя. - GET-запрос на `/api/news` - получение списка новостей. - POST-запрос на `/api/news` - создание новой новости. Сигнатура запроса: `{ text, title }`. - PATCH-запрос на `/api/news/:id` - обновление существующей новости. Сигнатура запроса: `{ text, title }`. - DELETE-запрос на `/api/news/:id` - удаление существующей новости. - Автоматический GET-запрос на `/api/users` - получение списка пользователей. - PATCH-запрос на `/api/users/:id/permission` - обновление существующей записи о разрешениях конкретного пользователя. Сигнатура:
`    {
        permission: {
            chat: { C: Boolean, R: Boolean, U: Boolean, D: Boolean },
            news: { C: Boolean, R: Boolean, U: Boolean, D: Boolean },
            settings: { C: Boolean, R: Boolean, U: Boolean, D: Boolean }
        }
    }
   `

> Обьект пользователя:

```
{
    firstName: String,
    id: Primary key,
    image: String,
    middleName: String,
    permission: {
        chat: { C: Boolean, R: Boolean, U: Boolean, D: Boolean },
        news: { C: Boolean, R: Boolean, U: Boolean, D: Boolean },
        settings: { C: Boolean, R: Boolean, U: Boolean, D: Boolean }
    }
    surName: String,
    username: String
}
```

> Обьект авторизованного пользователя:

```
{
    firstName: String,
    id: Primary key,
    image: String,
    middleName: String,
    permission: {
        chat: { C: Boolean, R: Boolean, U: Boolean, D: Boolean },
        news: { C: Boolean, R: Boolean, U: Boolean, D: Boolean },
        settings: { C: Boolean, R: Boolean, U: Boolean, D: Boolean }
    }
    surName: String,
    username: String,

    accessToken: String,
        refreshToken: String,
        accessTokenExpiredAt: Date (ms),
        refreshTokenExpiredAt: Date (ms)
}
```

> Обьект новости:

```
{
    id: Primary key,
    created_at: Date,
    text: String,
    title: String,
    user: {
        firstName: String,
        id: Key,
        image: String,
        middleName: String,
        surName: String,
        username: String
    }
}
```

> Обьект с токенами:

```
{
    accessToken: String,
    refreshToken: String,
    accessTokenExpiredAt: Date (ms),
    refreshTokenExpiredAt: Date (ms)
}
```

> (Более подробную информацию о url, дополнительных параметрах и передаваемых данных запроса вы можете получить через средства разработчика при взаимодействии с интерфейсом).

- Взаимодействие frontend и backend частей между собой - с помощью socket для реализации чата. Есть хеш-объект, в который записываются все активные подключения в формате:

```
{ #socketId: {
  username: #username,
  socketId: #socketId,
  userId: #userId,
  activeRoom: null // По умолчанию
  },
  ...
}
```

socket-подключение обрабатывает следующие события:

- `users:connect`, инициируется при подключении пользователя. Создается объект пользователя и в нем сохраняется socketId сокета, userId пользователя и имя пользователя, как свойства, обновляется общий объект, и отправляется в виде массива, только что подключившемуся пользователю (с помощью события `users:list`) и разсылается всем подключенным сокетам(с помощью события `users:add`).
- `message:add`, инициируется при отправке одним из пользователей сообщения другому.
- `message:history`, инициируется при открытии пользователем чата.Возвращает пользователю список сообщений диалога с выбранным пользователем. Параметры: recipientId - id пользователя-получателя (чат с которым мы открыли), userId - id пользователя (свой). Список сообщений диалога отправляется с помощью события `message:history`.
- `disconnect`, инициируется при отключении пользователя и передает всем подключенным пользователям socketId отключившегося пользователя (с помощью события `users:leave`), и пользователь удаляется из объекта всех подключенных пользователей.
