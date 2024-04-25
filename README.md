# Проектирование высоконагруженного пространства для командной работы

Курсовая работа в рамках 3-го семестра программы по Веб-разработке ОЦ VK x МГТУ им. Н.Э. Баумана (ex. "Технопарк") по дисциплине "Проектирование высоконагруженных сервисов"

#### Автор - [Андрей Мышляев](https://park.vk.company/profile/a.myshliaev/ "Страница на портале VK x МГТУ")
#### Задание - [Методические указания](https://github.com/init/highload/blob/main/homework_architecture.md)

#### Содержание:
1. [Тема, функционал и аудитория](#1)
2. [Расчет нагрузки](#2)
3. [Расчет глобальной нагрузки](#3)
4. [Расчет локальной нагрузки](#4)
5. [Логическая схема БД](#5)
6. [Физическая схема БД](#6)
7. [Алгоритмы](#7)
8. [Технологии](#8)
9. [Обеспечение надёжности](#9)
10. [Схема проекта](#10)

## Часть 1. Тема и целевая аудитория <a name="1"></a>

### Тема курсовой работы - **"Проектирование сервиса для командной работы"**
Confluence — это удобное рабочее пространство для удаленных команд, в котором участники могут хранить знания и сотрудничать друг с другом. В приложении Confluence Cloud можно легко записывать возникшие идеи, создавать и редактировать страницы и совместно работать с командой практически на любом устройстве.


### Ключевой функционал сервиса
- Поиск и навигация по документам.
- Просмотр документов.
- Создание и редактирование документов.

### Целевая аудитория
- Весь мир
- 25 млн пользователей в месяц (MAU) [^1]
- 0.84 млн пользователей в день (DAU) [^1]

## Часть 2. Расчет нагрузки <a name="2"></a>

### Продуктовые метрики

### Среднее количество действий пользователя по типам в день.
Допустим, в день пользователь совершает 10 посещений. Таким образом при ежедневном посещении 0.84 млн пользователей количество просмотров страниц будет равно 0.84 * 10 = 8.4 млн. На основе личного опыта предположим, что около 1 страницы пользователь находит по поиску, а навигация по сайту 10 - 1 = 9.
Тогда получается, что на поиск уходит 0.84 млн просмотров страниц, а на переход по ссылкам и навигацию внутри сайта уходит 7.56 млн просмотров.

#### Создание и редактирование страниц.
Допустим, в день пользователь делает 2 вправки. Тогда при ежедневном посещении 0.84 млн пользователей количество вправок будет равно:
```
0.84 * 2 = 1.68 млн.
```
Допустим, что количество созданий документов равняется 5% от числа вправок. Тогда новых документов в день равняется:
```
1.68 * 0.05 = 0.084 млн.
```

| Действие пользователя  | Количество в день |
|------------------------|-------------------|
| Поиск                  | 840 000           |
| Навигация              | 7 560 000         |
| Просмотр документов    | 8 400 000         |
| Создание документов    | 84 000            |
| Редактирование         | 1 680 000         |

В среднем одна картинка равняется 500 КБ.
В среднем на одного пользователя приходится 2 ГБ за 5 лет. Тогда за один день он занимает новые 1.1 МБ. Откуда получаем объем памяти, затрачиваемой каждый день: 
```
0.84 млн * 1.1 МБ = 924 ГБ.
```
Confluence существует уже 18 лет. Тогда всего памяти было затрачено: 
```
18 * 365 * 924 = 5.78 ПБ. 
```
Найдём общее число документов: 
```
18 * 365 * 84 000 = 551 880 000.
```
Найдём средний вес документа: 
```
5.78 ПБ / 551 880 000 = 11.2 МБ.
```
В среднем на статью приходится [620 слов](https://thequickadvisor.com/how-many-links-does-wikipedia-have/), а средняя длина слов равняется 5 буквам. 1 символ в Unicode кодируется 2 байтами. Статья также имеет ID - 16б (UUID).
В среднем каждая вправка занимает [25 МБ](https://stats.wikimedia.org/#/en.wikipedia.org/content/net-bytes-difference/normal|bar|2-year|~total|daily).

Средний объём памяти текстом:
```
(620 * 5 * 2 + 16) / 8 = 777 байт.
```

#### Сводная таблица продуктовых метрик
| Характеристика                                           |    Метрика    |
|:---------------------------------------------------------|:-------------:|
| MAU                                                      |      25M      |
| DAU                                                      |     0.84М     |
| Общее число документов                                   |   551.88 М    |
| Среднее время просмотра страницы                         |    5.26 м     |
| Средний размер документа                                 |    11.2 МБ    |
| Средний размер вправки документа                         |     25 МБ     |
| Среднее количество действий по просмотру документов      |  8.4 M/сутки  |
| Среднее количество поисков по документам                 | 0.84 M/сутки  |
| Среднее количество действий по созданию документов       | 0.084 М/сутки |
| Среднее количество действий по редактированию документов | 1.68 М/сутки  |

### Технические метрики:

Найдём среднее число картинок на документ. Из общего среднего веса картинки вычтем вес текста и делим на средний размер картинки.
```
(11.2 МБ - 777 байт) / 500 КБ = (11468 КБ - 0.75 КБ) / 500 КБ = 23
```

Размер хранения текста: 
```
0.77 КБ * 551 880 000 = 0.39 ТБ
```

Размер хранения картинок: 
```
23 * 500 КБ * 551 880 000 = 5.77 ПТ 
```

RPS по просмотру документов: 
```
8 400 000 / (60 * 60 * 24) = 97.2
```

RPS по созданию документов: 
```
84 000 / (60 * 60 * 24) = 0.97
```

RPS по редактированию документов: 
```
1 680 000  / (60 * 60 * 24) = 19.4
```

RPS по поиску документов:
```
840 000 / ( 60 * 60 * 24 ) = 9.7 RPS
```

Возьмем небольшой коэффициент k=1,5 отличия от среднего трафика для получения пикового трафика по просмотру документов: 
```
97.2 * 11.2 МБ * 1.5 = 1.59 ГБ/с
```

Возьмем небольшой коэффициент k=1,5 отличия от среднего трафика для получения пикового трафика по созданию документов: 
```
0.97 * 11.2 МБ * 1.5 = 0.016 ГБ/с
```

Возьмем небольшой коэффициент k=1,5 отличия от среднего трафика для получения пикового трафика по редактированию документов: 
```
19.4 * 25 МБ * 1.5 = 0.7 ГБ/с
```

В среднем одно слово занимает 5 букв. Тогда при поиске будет отправлять 5 байт.
Возьмем небольшой коэффициент k=1,5 отличия от среднего трафика для получения пикового трафика по поиску документов:
```
9.7 * 5  * 1.5 = 73 Байт/с
```

Пиковое потребление в течение суток:
```
1.59 + 0.016 + 0.7 = 2.31 ГБ/с
```

Cуммарный суточного трафик: 
```
(97.2 * 11.2 + 0.97 * 11.2 + 19.4 * 25) * (24 * 60 * 60) = 136 901 145 МБ/сутки = 0.127 ПТ/сутки
```

#### Сводная таблица технических метрик
| Характеристика                      |    Метрика     |
|:------------------------------------|:--------------:|
| Размер хранения текста              |    0.39 ТБ     |
| Размер хранения картинок            |    5.77 ПТ     |
| Пиковое потребление в течении суток |  12.31 ГБ/сек  |
| Суммарный суточный                  | 0.127 ПТ/сутки |
| RPS по просмотру документов         |      97.2      |
| RPS по созданию документов          |      0.97      |
| RPS по редактированию документов    |      19.4      |
| RPS по поиску документов            |      9.7       |


## Часть 3. Расчет глобальной нагрузки <a name="3"></a>

### Расстановка серверов компании atlassian.
![img_5.png](/images/img_5.png)

Расположение ДЦ
Основная аудитория находится в Северной и Южной Америке, Европе и Юго-восток Азии.
Расположим ДЦ в соответствии с географическим положением.

Тогда:

1. Расположим ДЦ в Нью-Йорке и Сан-Франциско (крупнейшие центры, отвечают за трафик восточного и западного побережья).

2. Сан-Паулу (крупнейший город и центр трафика Бразилии, отвечает за Бразилию).

3. Франкфурт (одна из крупнейших развязок в Европе и мире, отвечает за Европу).

4. Лондон (крупнейший узел Великобритании, отвечает за Британию).

5. Сингап (столица Сингапура, отвечает за Индию, юг Азии и Океанию).

6. Сидней (крупнейший узел в Австралии, отвечает за Австралию).

### Выбор способа глобальной балансировки

Предлагаю балансировку клиентов между регионами (Америка, Европа, Азия) осуществлять через привязку IP-адресов ДЦ к третьестепенным доменам компаний, указанным в URL-запросе.
Так, при обращении к domain-name.atlassian.net, DNS вернет IP того ДЦ, где зарегистрирована компания.
Такая схема позволит пользователям со всего мира подключаться к тому центру, через который обслуживается их компания.

## Часть 4. Расчет локальной нагрузки <a name="4"></a>
### L7 балансировка

Будем использовать сервер Nginx. Использовать его будем для следующих процессов:

* Отдача статики 
* Сжатие контента (gzip)
* Отслеживание обработки медленных клиентов (с помощью использования асинхронной обработки)
* SSL терминация.

Также Nginx осуществляет балансировку на ноды Kubernetes.
Kubernetes выполняет auto-scaling, перезапускает сервисы при падении. Это обеспечивает отказоустойчивость.

## Часть 5. Логическая схема БД <a name="5"></a>
![img_8.png](images/img_8.png)

|     Таблица      |                                                                                                                       Описание                                                                                                                        |
|:----------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
|     Sessions     |                                                                                                             Хранение сессий пользователей                                                                                                             |
|      Users       |                                                                                                                    Таблица юзеров                                                                                                                     |
|       Acl        |                                                                                                             Таблица прав юзеров к папкам                                                                                                              |
|    Documents     |                                                                                                                  Таблица документов                                                                                                                   |
|     Versions     |                                                                                                           Таблица прошлых версий документов                                                                                                           |
|    Companies     |                                                                                                           Таблица с информацией о компаниях                                                                                                           |
| Documents_search | Таблица предназначена для поиска документов. Эта таблица является облегченной копией таблицы documents для ускорения поиска. Она содержит только название и текст documents, чтобы была возможность поиска не только по названию, но и по содержанию. |

#### Sessions
Сессионная кука весит 32 байта, дата 8 байт, id пользователя 16 байт, откуда одна строка равняется: 32 + 8 + 16 = 56 байт.

Ежемесячно на сайт заходит 25 млн пользователей, таким образом необходимая память:
```
25 000 000 * 56 = 1.3 ГБ
```
Обращение к данной таблице будет происходить при просмотре, редактировании и создании документах. Также и при полнотекстовом поиске будет обращение к данной таблице.
```
97.2 + 19.4 + 0.97 + 9.7 = 127.3 RPS
```

#### Users
Имя пользователя 16 байт, id пользователя 16 байт, email 30 байт, дата 8 байт, название компании 32 байт. Необходимая память:
```
25 000 000 * ( 16 + 16 + 30 + 8 + 32 ) = 2.37 ГБ
```
При авторизации необходимо пройтись по таблице 1 раз, в месяц это будет 25 млн. Найдём RPS:
```
25 000 000 / ( 30 * 24 * 60 * 60) = 9.7 RPS
```

#### Acl
Два id по 16 байт, права 16 байт. Общее число документов 551 880 000. Найдём необходимую память:
```
551 880 000  * ( 16 * 3 ) =  24.67 ГБ
```
Обращение к этой таблице будет всегда, когда мы редактируем и просматриваем документы. Тогда общая нагрузка будет равняться сумме RPS просмотра документов и редактированию:
```
97.2 + 19.4 = 116.6 RPS
```

#### Documents

Всего документов 551 880 000, среднее число картинок 23, средний вес текста 777 байт, название 16, две даты по 8, три id по 16:
```
551 880 000 * (23 * 16 + 16 * 3 + 16 + 777 + 8 * 2) = 629.22 ГБ
```
Обращение к этой таблице будет равняться RPS просмотра документов, изменению и созданию. Также и поиск.
```
97.2 + 19.4 + 0.97 + 9.7 = 127.3 RPS
```

#### Versions
На создание 84 тыс документов приходится 1 680 000 редактирований. В среднем на один созданный документ приходится 20 изменений.

```
551 880 000 * 20 * ( 23 * 16 + 777 + 16 * 3 ) = 11.97 ТБ
```
Обращение к этой таблице всегда будет при изменении документов. Просмотр таблицы не учитывается, так как просмотр старых версий происходит крайне редко. Тогда общее RPS равняется 19.4

#### Companies
Id на 16 байт, название компании 32 байта, в среднем на компанию 30 человек.
```
(25 млн / 30 ) * ( 30 * 16 + 32 + 16) =  0.4 Гб
```
Обращение к этой таблице будет при авторизации пользователей для просмотра и редактирования документа. Откуда получаем 116.6 RPS.


| Таблица   | Количество строк | Размер строки | Размер всех записей |  RPS  |
|:----------|:----------------:|:-------------:|:-------------------:|:-----:|
| Sessions  |      25 млн      |    56 байт    |       1.3 ГБ        | 127.3 |
| Users     |      25 млн      |   102 байт    |       2.37 ГБ       |  9.7  |
| Acl       |    551.81 млн    |    48 байт    |      24.67 ГБ       | 116.6 |
| Documents |    551.81 млн    |    1.2 КБ     |      629.22 ГБ      | 127.3 |
| Versions  |     11 млрд      |    1.16 КБ    |      11.97 ТБ       | 19.4  |
| Companies |     0.83 млн     |   528 байт    |       0.4 Гб        | 116.6 |


## Часть 6. Физическая схема БД <a name="6"></a>
### Выбор СУБД
![img_6.png](images/img_6.png)

|     Таблица      |  База данных  |                                                                                                                       Описание                                                                                                                        |
|:----------------:|:-------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
|     Sessions     |     Redis     |                                                                                                             Хранение сессий пользователей                                                                                                             |
|      Users       |  PostgreSQL   |                                                                                                                    Таблица юзеров                                                                                                                     |
|       Acl        |  PostgreSQL   |                                                                                                             Таблица прав юзеров к папкам                                                                                                              |
|    Documents     |  PostgreSQL   |                                                                                                                  Таблица документов                                                                                                                   |
|     Versions     |  ClickHouse   |                                                                                                           Таблица прошлых версий документов                                                                                                           |
|    Companies     |  PostgreSQL   |                                                                                                           Таблица с информацией о компаниях                                                                                                           |
| Documents_search | ElasticSearch | Таблица предназначена для поиска документов. Эта таблица является облегченной копией таблицы documents для ускорения поиска. Она содержит только название и текст documents, чтобы была возможность поиска не только по названию, но и по содержанию. |
|      Фотки       |     CEPH      |                                                                                                                   Хранение картинок                                                                                                                   |

Так как в таблице Documents много данных и поиск полнотекстовый по нему в Postgresql будет долгим,
то для ускорения поиска необходимо выделить новую таблицу Documents_search, которая будет расположена в ElasticSearch. 
В этой таблице будет находиться название документа и текст документа, чтобы совершать полнотекстовый поиск по названию и тексту документа.

### Индексы
Индексировать необходимо следующие столбцы для ускорения запросов:
* Таблица Users: id
* Таблица Acl: (user_id, doc_ud)
* Таблица Documents: id
* Таблица Versions: doc_id
* Таблица Documents_search: id
* Таблица Companies: id

### Нормализация
В таблице Companies присутствует массив id работников, чтобы отсутствовали лишние join между таблицами.
В таблицах Documents и Versions присутствуют массивы путей картинок, вместо созданий 3х таблиц с лишними join.

### Клиентские библиотеки
Так как мы используем язык Go, то для подключения к СУБД будем использовать следующие библиотеки:

* Для подключения ElasticSearch: olivere/elastic
* Для подключения PostgreSQL: jackc/pgx
* Для подключения Redis: go-redis
* Для подключения CEPH: go-ceph
* Для подключения ClickHouse: clickhouse-go

### Шардирование
* для таблицы Users организуем шардинг по id.
* для таблицы Documents организуем шардинг по id.
* для таблицы Versions организуем шардинг по doc_id.
* для таблицы Companies организуем шардинг по id.
* для таблицы Acl организуем шардинг по doc_id.
* для таблицы Documents_search организуем шардинг по id.

[//]: # (### Репликация)

[//]: # (В качестве схемы репликации будем использовать стандартную схему с физической репликацией. )

[//]: # (Одна база данных на запись &#40;master&#41; и одна на чтение &#40;slave&#41;.)

[//]: # (Реплику делаем ассинхронным, так как мы можем позволить себе определенное отставание от мастера на какое-то время.)

## Часть 7. Алгоритм <a name="7"></a>

### Поиск
Пользователям требуется быстро находить нужные документы по ключевым словам. Поскольку полнотекстовый поиск в PostgreSQL медленный, 
мы будем использовать ElasticSearch, который обеспечивает более быстрый полнотекстовый поиск.

Создадим копию таблицы documents в ElasticSearch, которую назовем documents_search.

В качестве анализатора будем использовать стандартный анализатор ElasticSearch, 
который разбивает текст на токены согласно алгоритму Unicode Text Segmentation и приводит полученные токены к нижнему регистру.

Выгрузим данные из PostgreSQL в ElasticSearch и создадим обратные индексы для ускорения поиска. 
В процессе индексации будет создан список всех слов, используемых в документах, и для каждого слова будет "запомнено", 
в каких документах оно встречается. Хотя процесс индексации может занять некоторое время, наша основная цель - обеспечить высокую скорость поиска.

После выполнения поиска необходима сортировка по степени их релевантности. Это называется ранжирование. 
Ранжирование выполняется с использованием модели TF/IDF (Частота Термина/Обратная Частота Документа).

Суть модели TF/IDF заключается в следующем:
* Частота термина (TF): Это мера того, насколько часто конкретное слово или фраза встречаются в документе. Если слово встречается часто, его важность в контексте документа растет.
* Обратная частота документа (IDF): Это мера того, насколько часто слово встречается во всех документах. Если слово встречается в большинстве документов, его значение понижается, так как оно становится менее дифференцирующим.

Сочетание TF и IDF позволит ранжировать документы по степени их релевантности для конкретного поискового запроса.
Также используем необходима функция скоринга для настройки ранжирования, чтобы установить, что заголовок документа важнее его содержимого, и Elasticsearch будет учитывать это при ранжировании результатов.

После результаты будут отображаться в порядке убывания релевантности.







## Часть 8. Технологии <a name="8"></a>

| Технология    |              Область применения              |                                                                                           Мотивация                                                                                           |
|:--------------|:--------------------------------------------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| Golang        |                   Backend                    |                              Простота, своя модель параллелизма, сборщик мусора, cистема модулей, большое количество технологий из коробки, производительность.                               |
| React         |                   Frontend                   |                                                    Cбалансирован как с точки зрения сложности, так и с точки зрения богатства функционала.                                                    |
| Nginx         |                Load balancer                 |                                              Надежный и функциональный балансировщик, выступающий в том числе в качестве сервера раздачи статики                                              |
| Redis         |           Пользовательских сессий            |                                                                    Легко масштабируется, большое комьюнити, очень быстрый                                                                     |
| Kubernetes    |           Развертывание приложения           |                                                       автоматизирует процессы развертывания, масштабирования и управления контейнерами                                                        |
| CEPH          |              Хранилище картинок              |                                                                                       Хранение картинок                                                                                       |
| Api GateWay   |                Маршрутизация                 |                     Выполняет роль центральной точки входа для внешних запросов к микросервисной архитектуре, обеспечивая безопасность, мониторинг и управление трафиком.                     |
| ClickHouse    |                   Database                   |                                                Высокая производительность при выполнении аналитических запросов над большими объемами данных.                                                 |
| ElasticSearch |                    Поиск                     | Хранилище данных для поиска, анализа и хранения в реальном времени. Предоставляет расширенные возможности поиска, включая полнотекстовый поиск, фасетный поиск, ранжирование результатов и др |
| Prometheus    |            Сбор и хранение метрик            |                                                                         Возможность создания гибких запросов к данным                                                                         |
| Grafana       | Визуализация метрик для мониторинга сервисов |                                                                                   Гибкая настройка графиков                                                                                   |

## Часть 9. Обеспечение надёжности <a name="9"></a>

### Отказоустойчисвость БД
1. Резервирование БД:.
    * PostgreSQL Master-Slave репликация с помощью встроенной Streaming replication по 1 реплике таблиц
    * ClickHouse по 1 реплике с ReplicatedMergeTree
    * CEPH ??
2. Резервирование физических компонентов (сервера, диски и т.д.)

### Надежность сервисов
* re-try запросы с дедлайном.
* Graceful shutdown: Будем использовать Graceful Node Shutdown в Kubernetes, чтобы при остановке сервиса, не терялись обрабатываемые в этот момент запросы

### Observalvability
* Сбор метрик, логов с сервисов.

## Часть 10. Схема проекта <a name="10"></a>
![img_7.png](/images/img_7.png)

| Сервис                   |                       Используемая таблица/хранилище                        |
|:-------------------------|:---------------------------------------------------------------------------:|
| User and Session Service |                  Таблица user в PostgreSQL, сессии в Redis                  |
| Documents service        | Таблицы Documents, Acl в PostgreSQL, Versions в ClickHouse, картинки в CEPH |
| Companies service        |                       Таблица Companies в PostgreSQL                        |
| Search service           |                  Таблица Documents_search в ElasticSearch                   |

## Список литературы.

1. [Карта ДЦ www.atlassian.com](https://www.atlassian.com/trust/reliability/infrastructure)
2. [Глобальная карта сетевых магистралей](https://global-internet-map-2022.telegeography.com/)
3. [Распределение пользователей по странам.](https://pro.similarweb.com/#/digitalsuite/websiteanalysis/audience-geography/*/999/3m?key=atlassian.com&webSource=Total)
4. [Принцип работы ElasticSearch](https://ai-news.ru/2023/09/osnovy_polnotekstovogo_poiska_v_elasticsearch_chast_vtoraya.html)
