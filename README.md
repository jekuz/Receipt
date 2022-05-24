Console app implements functionality of a store receipt:

![изображение](https://user-images.githubusercontent.com/59832216/169829469-1f73f7cb-dd0f-4197-8ae6-d22d664264b5.png)

- run: java CheckRunner pl-productList.csv cl-cardList.csv 12-2 44-7 75-11 89-25 94-3 cn-21
- single argument 'help' shows using examples: java CheckRunner help
- arguments format: ID-QTY (product identifier - product quantity)
- additional arguments: pl-filename, cl-filename cn-number (product list, discount card list, discount card number)
- run example: java CheckRunner pl-productList.csv cl-cardList.csv 3-15 16-8 34-1 cn-1025
- positions limiter included

The check form implements:

- scaling in width
- displaying an auto-centered header with store data
- counter of limited receipts (increasing with each subsequent call)
- current date and time
- lines of commodity items
- position quantity limiter
- promotional positions indicating the percentage of the discount and the amount minus it
- the amount total, discount card percentage and amount, result total
- trimming fields if the length is exceeded
- save to file
- HTTP output

Check calculation implements:

- calculation of all positions based on the transmitted data in the parameters
- calculation of promotional positions based on the provided data of the product list
- calculation of the total discount on the provided discount card with exception of promotional positions
- calculation of the total positions amount, discount card's amount and the final result amount
- rounding the position amounts and totals to the cent for buyer's interest

Additionally:

- test files generation + saving of product lists and discount cards
- check for the existence of arguments files specified
- checking product's and discount card existence
- full check of arguments for correctness
- main methods and calculation's test coverage

-------------------------------------------------------------------------------------------------------------------

Консольное приложение реализующее функционал формирования магазинного чека.

Приведённый пример запуска:
java CheckRunner pl-productList.csv cl-cardList.csv 12-2 44-7 75-11 89-25 94-3 cn-21

Цифровые аргументы - аргументы позиций и их количества.
Необязательные аргументы:
pl - аргумент .csv файла позиций (при отсутствии будет сгенерирован тестовый объект ста позиций)
сl - аргумент .csv файла скидкарт (при отсутствии будет сгенерирован тестовый объект ста скидкарт)
cn - аргумент номера скидкарты

Аргументы принимаются в произвольном порядке.
Единственный аргумент 'help' выводит в консоль примеры запуска.

Принцип работы:

1. Статическим методом 'parser' класса 'Args' формируется объект разобранных аргументов класса 'Data':

   - productsFileName - имя файла списка продуктов формата .csv (аргумент 'pl', null если не указан)
   - cardsFileName - имя файла списка скидкарт формата .csv (аргумент 'cl', null если не указан)
   - cardNumber - номер скидкарты (аргумент 'cn', null если не указан)
   - products - хэшмап аргументов позиций (id : qty)
   
   Существует ограничение на количество:
    - позиций в чеке (по умолчанию 5)
    - товара в позиции (по умолчанию 999)
    - чеков (по умолчанию 9999)
   
   При некорректных аргументах выбрасываются соответствующие исключения уведомляющие о причине.
   Единственный аргумент 'help' выводит в консоль примеры использования.

2. Формируются объекты данных позиций (pList) и скидкарт (cList):

   - Объекты данных позиций и скидкарт представляют собой ассоциативные массивы (хэшмапы) обобщённых типов
     (дженериков) зависящих от класса реализации имплементирующего интерфейс 'DataMap' имеющего методы
     проверки ключа на существование 'contains', получения значения по ключу ('getValue') и метод сохранения
     данных в файл 'save'.
   - Объект данных позиций - (id : product), где объект 'product' состоит из наименования, цены, процента скидки
     на позицию и количества позиции от которого скидка действует. Фактически, каждая позиция может быть
     акционной и иметь индивидуальные параметры скидки.
   - Объект данных скидкарт: (номер карты : процент скидки).
   - При отсутствии аргументов чтения из файлов объекты данных генерируются.
   - Генератор позиций формирует объект из 100 товаров со случайными данными.
   - Генератор скидкарт формирует объект из 100 скидкарт от 0 до 100% скидки по карте.
   - Объекты данных имеют методы проверки на содержание, получение значения и метод сохранения.
   - Данные объектов хранятся в файлах в формате .csv

3. Осуществляется проверка аргументов объекта класса 'Data' на присутствие указанных позиций 
   и номера скидкарты в сформированных объектах данных 'pList' и 'cList' соответственно.
   Проверку реализует статический метод 'check' класса 'Args'.

   - выбрасываются исключения если 'id' позиции или номер скидкарты (если она была указана)
     не существуют в соответствующих объектах данных.

4. Формируется чек исходя из результатов калькуляции на основе объекта аргументов класса 'Data':

   - Вызывается статический метод 'result' класса 'Calc', возвращающий объект итогов класса 'Result',
     состоящего из номера чека, массива объектов позиций класса 'Position' и всех итоговых значений.
   - Во избежание погрешностей, суммы расчётов хранятся в центах в целочисленных типах.
   - Во всех итоговых суммах происходит округление до цента в сторону покупателя.
   - Объект класса 'FormBuilder' имплементирует интерфейс 'Form', представляющий собой массив
     строк чека и имеющего реализованные методы печати и сохранения в файл.
     Исходные данные берутся из объекта итогов класса 'Result', переданного в 'FormBuilder'.
   - Ширина строк чека масштабируема, поля раздвигаются и обрезаются при необходимости.
   - Включаются дополнительные строки на акционные позиции с указанием процента скидки и
     цены позиции после применения этой скидки.
   - Скидка по скидкарте не распространяется на расчитанные акционные позиции.

5. Запускается сервер HTTP, отдающий чек на localhost:8888/check

Разбор аргументов и все расчёты подтверждаются произвольным количеством динамических тестов.   
Динамические тесты иммитируют корректные и некорректные, а так же неправильные аргументы.
Корректные аргументы расчитываются и сравниваются с результатами статического метода 
'result' класса 'Calc'.

-------------------------------------------------------------------------------------------------------------------

- запуск: java CheckRunner <параметры>
- единственный параметр help выводит примеры использования
- формат цифровых параметров: ID-Quantity, где ID идентификатор товара, а Quantity его количество
- дополнительные параметры: pl-имя файла списка товаров, cl-имя файла списка скидкарт, cn-номер скидкарты
- пример запуска: java CheckRunner pl-productList.csv cl-cardList.csv cn-1025 3-15 16-8 34-1
- ограничитель позиций

Форма чека реализует:

- масштабирование по ширине
- вывод автоцентрированной шапки с данными магазина
- вывод счётчика чеков с ограничением (увеличивающийся с каждым последующим вызовом)
- вывод текущей даты и времени
- ограничитель количества товара в позиции
- вывод строк товарных позиций
- вывод строк акционных позиций с указанием процента скидки и суммы за её вычетом
- вывод строк итогов с указанием суммы и процента скидки по скидкарте
- обрезку полей при условии превышения длины
- сохранение в файл
- HTTP вывод

Калькуляция чека реализует:

- расчёт всех позиций исходя из переданных данных в параметрах
- расчёт акционных позиций исходя из предоставляемых данных списка товаров
- расчёт общей скидки по предоставленной скидкарте за исключением акционных позиций
- расчёт общей суммы позиций, суммы скидки по скидкарте и конечной суммы оплаты
- округление сумм позиций и итогов до цента в сторону покупателя

Дополнительно:

- генерирование и сохранение тестовых файлов списков товаров и скидкарт
- проверка существования файлов указанных в параметрах
- проверка существования товара и скидкарты
- полная проверка параметров на корректность и повторы
- покрытие тестами ключевых методов
