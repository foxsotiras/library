# Глоссарий идентификаторов ИС «Интеллектуальная библиотека»

ID элемента указывается первым аргументом его объявления в PlantUML. Для общих элементов используется один и тот же глобальный ID в объявлениях и связях всех схем. За основу взяты сводные диаграммы [контекста](docs/src/context.puml) и [контейнеров](docs/src/containers.puml). Дополнения к ландшафту и межконтейнерные стыки сверены с соответствующими исходниками в `docs/src`.

## Система, роли и внешние интеграции

| Глобальный ID | Тип C4 | Название и назначение |
| --- | --- | --- |
| `library_system` | `System` / `System_Boundary` | ИС «Интеллектуальная библиотека». На контексте и ландшафте — программная система, на диаграмме контейнеров и в динамических схемах — её граница. |
| `reader` | `Person` | Читатель. Ищет и бронирует издания, пользуется личным кабинетом и электронным фондом. |
| `librarian` | `Person` | Библиотекарь. Регистрирует читателей, оформляет выдачи и возвраты. |
| `acquisitor` | `Person` | Специалист по комплектованию. Ведёт каталог и фонд, рассматривает заявки на закупку. |
| `moderator` | `Person` | Модератор сообщества. Обрабатывает жалобы и модерирует форум. |
| `admin` | `Person` | Администратор библиотеки. Управляет учётными записями, ролями и настройками, получает отчёты. |
| `payment_gateway` | `System_Ext` | Платёжный шлюз. Принимает оплату штрафов и уведомляет ядро о статусе платежа. |
| `trends_api` | `System_Ext` | Внешние книжные API. Предоставляют метаданные и рейтинги популярности. |
| `neb_catalog` | `System_Ext` | Сводный каталог НЭБ. Источник библиографических записей RUSMARC. |
| `mail_service` | `System_Ext` | Почтовый сервис. Доставляет уведомления читателям через SMTP. |
| `city_portal` | `System_Ext` | Городской портал. Даёт переход к услугам библиотеки на ландшафте. |
| `municipal_iam` | `System_Ext` | Муниципальная IAM. Проектная интеграция единого входа через OIDC. |

В диаграммах ландшафта `municipality` обозначает границу муниципальной цифровой среды, а `library_it` — границу ответственности команды внутри неё. Сканер штрихкодов `barcode_scanner` объявлен как `System_Ext` в [компонентной диаграмме панели](docs/src/components/mandryka-admin_web.puml). Это локальная периферия, подключаемая через `scanner_adapter`.

## Контейнеры

| Глобальный ID | Тип C4 | Название и назначение | Ответственный |
| --- | --- | --- | --- |
| `reader_portal` | `Container` | Портал читателя. Каталог, личный кабинет, сообщество и читалка. | Глимакова Е.А. |
| `admin_web` | `Container` | Панель сотрудника. Обслуживание, комплектование, модерация и отчётность. | Мандрыка М.М. |
| `api_gateway` | `Container` | Backend API & Core Service. REST API и бизнес-логика библиотеки. | Буглак Д.Г. |
| `database` | `ContainerDb` | PostgreSQL. Учётные записи, каталог, выдачи, брони и прочие данные системы. | Буглак Д.Г. |
| `analytics_service` | `Container` | Сервис аналитики востребованности. События, тренды, рейтинг и рекомендации. | Миронов Д.С. |
| `message_broker` | `ContainerQueue` | RabbitMQ. Асинхронная доставка доменных событий. | Миронов Д.С. |
| `media_service` | `Container` | Сервис электронного фонда. Загрузка и потоковая выдача файлов с проверкой доступа. | Зайниев А.А. |
| `object_storage` | `ContainerDb` | S3-совместимое хранилище RustFS. Электронные издания, обложки и вложения. | Зайниев А.А. |

## Стыковочные компоненты L3

В диаграмме контейнера-владельца стык объявляется как `Component` с указанным ID. В диаграмме другого контейнера ссылка на этот стык оформляется как `Component_Ext` с тем же ID.

| Глобальный ID | Контейнер-владелец | Назначение | Кто обращается по диаграммам |
| --- | --- | --- | --- |
| `auth_comp` | `api_gateway` | Вход, токены, роли и проверка доступа к изданию. | `reader_portal`, `admin_web`, `media_service` |
| `catalog_comp` | `api_gateway` | Каталог, экземпляры, импорт записей НЭБ, рекомендации и отчёты о спросе. | `reader_portal`, `admin_web` |
| `loan_comp` | `api_gateway` | Регистрация читателей, выдача, возврат, продление и бронирование. | `reader_portal`, `admin_web` |
| `payment_comp` | `api_gateway` | Штрафы, создание платежей и обработка подтверждений оплаты. | `reader_portal`, `payment_gateway` |
| `community_comp` | `api_gateway` | Форумы, жалобы, модерация, заявки на комплектование и голоса. | `reader_portal`, `admin_web` |
| `analytics_api` | `analytics_service` | gRPC API рейтинга востребованности, рекомендаций и отчётов. | `api_gateway` через `analytics_client` |
| `media_api` | `media_service` | Приём файлов, выдача обложек и чтение электронных изданий по HTTP Range. | `reader_portal`, `admin_web` |

## Локальные ID компонентов

Область действия этих ID — диаграмма контейнера-владельца. В первом столбце указан глобальный ID контейнера.

| Контейнер | Локальные ID и назначение |
| --- | --- |
| [`reader_portal`](docs/src/components/glimakova-reader_portal.pulm) | `router` — маршруты и доступ, `community` — сообщество, `catalog` — каталог, `cabinet` — личный кабинет, `ereader` — читалка, `store` — состояние портала, `media` — клиент электронного фонда, `api` — клиент ядра. |
| [`admin_web`](docs/src/components/mandryka-admin_web.puml) | `rbac_router` — навигация по ролям, `circulation_ui` — обслуживание, `acquisition_ui` — комплектование, `moderation_ui` — модерация, `reports_ui` — отчёты, `admin_ui` — администрирование, `scanner_adapter` — ввод со сканера, `state_store` — состояние панели, `api_client` — клиент ядра, `upload_client` — загрузка файлов. |
| [`api_gateway`](docs/src/components/buglak-backend.puml) | `notification_comp` — уведомления, `import_comp` — импорт НЭБ, `event_publisher` — публикация доменных событий, `analytics_client` — клиент gRPC, `data_access` — доступ к данным и транзакции. |
| [`analytics_service`](docs/src/components/mironov-analytics.puml) | `event_consumer` — обработка событий, `trends_collector` — сбор трендов, `demand_scoring` — расчёт востребованности, `recommendation_engine` — рекомендации, `report_builder` — отчёты, `analytics_repo` — хранение агрегатов и результатов расчёта. |
| [`media_service`](docs/src/components/zayniev-media_service.puml) | `access_guard` — проверка доступа, `file_processor` — обработка файлов, `stream_service` — потоковая выдача, `storage_adapter` — операции с объектным хранилищем. |

## Псевдонимы в индивидуальных диаграммах

В `docs/src/context/` и `docs/src/containers/` встречаются собственные имена элементов. Таблица показывает соответствия по роли или назначению. При переносе связи в общую диаграмму применяется глобальный ID справа.

| Псевдоним в индивидуальной схеме | Глобальный ID |
| --- | --- |
| `library`, `biblio`, `smart_library` | `library_system` |
| `chitatel` | `reader` |
| `bibliotekar` | `librarian` |
| `acquisition`, `acquisition_staff` | `acquisitor` |
| `payment`, `bank`, `payment_gw`¹ | `payment_gateway` |
| `trends`, `trend_service`, `open_library` | `trends_api` |
| `email`, `email_service`, `mail`, `mail_provider` | `mail_service` |
| `backend_api`, `api` (в контейнерной схеме Миронова), `core_api` | `api_gateway` |
| `ml`, `ml_service`, `analytics`, `demand_ml` | `analytics_service` |
| `db`, `registry_db` | `database` |
| `broker` | `message_broker` |
| `storage`, `file_storage`, `file_store` | `object_storage` |
| `portal`¹ | `reader_portal` |

¹ `payment_gw` и `portal` встречаются в компонентной схеме портала. В этой же схеме `api` обозначает клиентский компонент портала.

Объединённый веб-клиент (`web`, `web_app`, `web_portal`) в индивидуальных контейнерных схемах следует сопоставлять с `reader_portal` или `admin_web` по роли пользователя и назначению связи. В сводной модели нет прямого соответствия для отдельных элементов `guest`, `publisher`, `sso`, `captcha`, `proxy`, `gateway`, `worker`, `notifier`, `cache` и `search`.

## Локальные ID развертывания

| Диаграмма | ID и назначение |
| --- | --- |
| [Панель сотрудника](docs/src/deployment/mandryka-deployment.puml) | `client_node` — рабочее устройство, `dmz` — демилитаризованная зона, `ingress_a` и `ingress_b` — два узла Nginx. |
| [Электронный фонд](docs/src/deployment/zayniev-deployment.puml) | `ingress` — вход через два Nginx и VIP, `media_a` и `media_b` — узлы приложений, `media_service_a` и `media_service_b` — экземпляры `media_service`, `storage_node` — узел `object_storage`, `backup_node` и `backup` — узел и приёмник резервных копий. |
