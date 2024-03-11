# Расчётно-пояснительная записка к курсовой работе по проектированию архитектуры высоконагруженного приложения

#### Методические указания: https://github.com/init/highload/blob/main/homework_architecture.md
## Содержание:
1. [Тема работы, основной функционал и аудитория](#1)
2. [Расчет нагрузки](#2)
3. [Глобальная балансировка нагрузки](#3)
3. [Источники](#Источники)

## 1. Тема работы, основной функционал и аудитория <a name="1"></a>
Требуется разработать архитектуру высоконагруженного приложения для социальной сети Instagram

### Функционал MVP:
- Регистрация и авторизация пользователей
- Поиск пользователей
- Подписка на контент других пользователей
- Загрузка контента (фото/видео)
- Оценка контента (лайки, комментарии)
- Просмотр контента
- Просмотр комментариев

### Целевая аудитория:
- 2 млрд. MAU (Monthly Active Users)
- 100 млн. фото и видео выкладывается на площадке каждый день
- 30 мин. в среднем проводят люди в Instagram

#### Распределение аудитории по странам:
| Страна         | Аудитория (млн. чел.) |
|----------------|-----------------------|
| Индия          | 230                   |
| США            | 160                   |
| Бразилия       | 120                   |
| Индонезия      | 100                   |
| Россия         | 63                    |
| Турция         | 52                    |
| Япония         | 46                    |
| Мексика        | 38                    |
| Великобритания | 32                    |
| Германия       | 30                    |

## 2. Расчет нагрузки <a name="2"></a>
Необходимо пересчитать аудиторные метрики в ключевые продуктовые и технические метрики.

### Продуктовые метрики
- 2 млрд. чел. MAU (Monthly Active Users)
- 500 млн. чел. DAU (Daily Active Users)
- 150 млн. новых пользователей зарегистрировались в 2023 году
- Предположительное число авторизаций одного пользователя в год - 3 раза
- 71.9% контента - фотографии, остальное видео
- 95 млн. новых постов каждый день (68.4 фото и 26.6 видео)
- 4.2 млрд. лайков и 1.2 млрд. комментариев каждый день
- 30 мин. проводит средний пользователь Instagram в день
- Предположительно пользователь просматривает 50 фото и 20 видео в день
- Предположительно пользователь ищет 1 профиль в день, а подписывается на каждого второго (0.5)

| Метрика                                                   | Значение | 
|-----------------------------------------------------------|----------|
| MAU (чел.)                                                | 2 млрд.  |
| DAU (чел.)                                                | 500 млн. |
| Прирост аудитории за 2023г.                               | 150 млн. |
| Число авторизаций пользователя в год одним пользователем  | 3        |
| Поиск других людей в день одним пользователем             | 1        |
| Подписок на профили в день одним пользователем            | 0.5      |
| Публикаций единиц контента в день одним пользователем     | 0.19     |
| Число лайков одним пользователем в день                   | 8.4      |
| Число комментариев одним пользователем в день             | 2.4      |
| Число просмотренных фотографий одним пользователем в день | 50       |
| Число просмотренных видео одним пользователем в день      | 20       |
| Получение комментариев к записи                           | 7        |

#### Средний размер харанилища пользователя по типам:
- Данные профиля (имя, описание профиля, подписки и т.д.) - 100 Кб
- Аватар профиля - 150 Кб
- Посты (фото, видео) - 500 Мб/год (Предположительно)

#### Среднее количество действий пользователя по типам в день
Берём в расчёт метрику DAU, а также приблизительные значения описанных выше метрик

| Действие пользователя  | Количество в день |
|------------------------|-------------------|
| Регистрация            | 410 тыс.          |
| Авторизация            | 16.6 млн.         |
| Поиск профиля          | 500 млн.          |
| Подписка на профиль    | 250 млн.          |
| Публикация фото        | 68.4 млн          |
| Публикация видео       | 26.6 млн.         |
| Лайк                   | 4.2 млрд.         |
| Комментарий            | 1.2 млрд.         |
| Просмотр фото          | 25 млрд.          |
| Просмотр видео         | 10 млрд.          |
| Получение комментариев | 3.5 млрд.         |


- Число регистраций (в день) 
```azure
Прирост за год / 365 = 150 млн. / 365 = 410 тыс.
```
- Число авторизаций (в день)
```azure
MAU * 3 / 12 / 30 = 16.6 млн.
```

### Технические метрики

#### Размер хранения в разбивке по типам данных
Учитывается размер хранения новых данных за год, так как данных об общем числе тех или иных 
данных за всю историю приложения нет. Например, считается размер хранения данных профилей
пользователей, зарегистрированных в 2023г. 

| Тип               | Количество (ед.) | Размер 1 ед. | Общий размер |
|-------------------|------------------|--------------|--------------|
| Фотографии        | 25 млрд.         | 2 Мб         | 50 Пб        |
| Видео             | 10 млрд          | 20 Мб        | 200 Пб       |
| Комментарии       | 430 млрд.        | 1 Кб         | 430 Тб       |
| Данные профиля    | 150 млн.         | 100 Кб       | 15 Тб        |
| Аватарки профилей | 150 млн.         | 150 Кб       | 22.5 Тб      |

#### RPS
Для приложений с выраженными пиками активности, такие как социальных сети, например, 
которые испытывают существенные всплески трафика в определенные моменты (время новостей, 
спортивные события, специальные онлайн-акции), могут иметь значительно выше соотношение между 
среднесуточным и пиковым RPS, например, 1:10 или даже 1:20. Возьмём среднее значение 1:10.

| Тип запроса            | RPS    | RPS пиковый |
|------------------------|--------|-------------|
| Регистрация            | 4.75   | 47.5        |
| Авторизация            | 192    | 1920        |
| Поиск профиля          | 5800   | 58000       |
| Подписка на профиль    | 2900   | 29000       |
| Публикация фото        | 790    | 7900        |
| Публикация видео       | 308    | 3080        |
| Лайк                   | 48611  | 486110      |
| Комментарий            | 13889  | 138890      |
| Просмотр фото          | 289350 | 2893500     |
| Просмотр видео         | 115740 | 115740      |
| Получение комментариев | 40509  | 40509       |
|                        |        |             |
| Общее                  | 518000 | 5180000     |

#### Сетевой трафик
Возьмём среднюю скорость сети 10 Мбит/с

| Пиковое потребление | Суммарный суточный трафик |
|---------------------|---------------------------|
| 51.8 Тбит/с         | 44.8 Пб                   |



## 3. Глобальная балансировка нагрузки <a name="3"></a>
### Расположение ДЦ
Выделим возможные расположения дата-центров с учётом географического распределения аудитории:
1. Северная Америка (США)
- Ашберн, Вирджиния (восточное побережье) (из-за его роли в качестве одного 
из крупнейших телекоммуникационных узлов в США)
- Сан Хосе, Калифорния (западное побережье)
2. Южная Америка
- Бразилия. Сан-Паулу (крупнейший город в Бразилии с развитой инфраструктурой, 
способный обслуживать пользователей по всей Южной Америке)
3. Азия 
- Сингапур (выступает в качестве ключевого телекоммуникационного и технологического хаба для Юго-Восточной Азии)
- Бангалор, Индия (один из крупнейших ДЦ)
4. Европа 
- Амстердам (один из крупнейших интернет-хабов в Европе с высокоразвитой инфраструктурой).
- Франкфурт (другой крупный интернет-хаб в Европе, имеющий один из самых больших интернет-обменных пунктов в мире).
5. Африка
- Йоханнесбург (ЮАР) (один из наиболее развитых городов Африки с хорошей технологической инфраструктурой, 
который может служить точкой доступа к интернету для южноафриканского региона).

### Глобальная балансировка
Для глобальной балансировки запросов и нагрузки будем использовать:
- Для определения региона - GeoDNS, т.к. ДЦ сильно разнесены по миру, что позволит достаточно точно 
определять в ДЦ какого региона стоит роутить запрос.
- Для роутинга запросов в пределах региона будем использовать BGP Anycast.

## Источники: <a name="Источники"></a>
1. [Instagram statistics 2023](https://www.zippia.com/advice/instagram-statistics/#:~:text=As%20of%202022%2C%20Instagram%20has,posted%20on%20Instagram%20each%20day.)
2. [75+ Instagram Statistics](https://www.socialpilot.co/instagram-marketing/instagram-stats)
