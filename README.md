# Прокси-сервер для проверки формата комментариев в хранилище конфигураций 1С

## Назначение

Простой инструмент для контроля формата комментариев, которые разработчики указывают при помещении изменений в хранилище. Это прокси-сервер nginx, который с помощью кода на Lua анализирует запросы и выдает `500 Internal Server Error` в случае, если комментарий к версии хранилища не указан или указан не по формату.

По умолчанию реализована одновременно строгая и наивная проверка комментария. Корректный комментарий:

- не пустой
- начинается на `#`, после чего должны следовать либо 5 цифр, либо строка `"нетзадачи"`
- содержит два перевода строки

Примеры корректных комментариев:

```md
#12345 Обработка заполнения ТЧ "Товары"

Добавлена обработка заполнения ТЧ "Товары" документа "Реализация товаров и услуг"
```

```md
#нетзадачи

Тех. долг
```

Код проверки комментария на соответствие формату можно исправить непосредственно в `conf.d/crserver-filter.conf`.

Параметры прокси проверены на конфигурациях размера ERP 2.4, в том числе в режиме подключения пустой конфигурации к хранилищу.
Пока нет достоверных данных о стабильности работы прокси-сервера при параллельной работе, поэтому используйте на свой страх и риск.

## Использование

- внести изменения в `conf.d/crserver-filter.conf`
  - указать адрес прокси-сервера и адрес сервера хранилища
  - уточнить формат комментария к версии хранилища
- внести изменения в docker-compose.yml
  - указать порт прокси-сервера (по умолчанию `3333`)
- выполнить `docker-compose up -d`
- открыть Конфигуратор, закрыть хранилище, открыть хранилище, вместо адреса реального хранилища указать `прокси:порт`

БЫЛО: `http://srv-app-01/CRServer/repository.1ccr/ERP_Master`

СТАЛО: `http://srv-ci-01:3333/CRServer/repository.1ccr/ERP_Master`

После проверки работоспособности необходимо изменить порт основного http-сервера хранилища, обновить его в `conf.d/crserver-filter.conf`, перезапустить контейнер.

## Ссылки

- [commitHook](https://github.com/asosnoviy/commitHook)
- [OpenResty](https://openresty.org/)
