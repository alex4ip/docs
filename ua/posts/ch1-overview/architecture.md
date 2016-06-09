### 1.2 Загальна архітектура

Чотири компоненти пошукача WebMagic реалізовані в `Downloader`,` PageProcessor`, `Scheduler`,` Pipeline` контактують поміж собою. Ці компоненти відповідають чотирьом складовим життєвого циклу пошукача: завантаження, обробка, управління і здатність зберігати результат. Архітектура WebMagic подібна до `Scapy`, але реалізована на `Java`.

Пошукач має кілька компонентів, організованих таким чином, щоб вони могли взаємодіяти один з одним. Пошукач можно розглядати, як великий контейнер з логікою у ядрі ​​WebMagic.

Загальна архітектура WebMagic виглядає наступним чином:

![image](http://code4craft.github.io/images/posts/webmagic.png)

### 1.2.1 чотири компонента WebMagic

#### 1.Downloader завантажувач

Downloader відповідає за завантаження інформації з Інтернет-сторінки для подальшої обробки. У WebMagic, за замовчуванням, інструмент завантаження - [Apache HttpClient](http://hc.apache.org/index.html).

#### 2.PageProcessor аналіз сторінки

PageProcessor відповідає за аналіз сторінки, отримання завданої інформації, а також за обробку чергових посилань. WebMagic дозволяє використовувати для аналізу сторінки різні інструменти. Одним з таких інструментів є XPath [Xsoup](https://github.com/code4craft/xsoup), якій використовує [Jsoup](http://jsoup.org/) в якості синтаксичного аналізу HTML.

З усіх чотирьох компонентів, тільки `PageProcessor` є унікальним для кожної сторінки, для кожного сайтую Саме його користувач повинен налаштувати самостійно.

#### 3.Scheduler планувальник

Менеджер планувальника "Scheduler" дозволяє планувати чергу сканування URL. WebMagic за замовчуванням від JDK використовує менеджер черги URL-ів у пам'яті `memory queue management`, та може встановлювати пріоритети. Також підтримує використання розподіленого управління через Redis.

У випадку, коли проект вимагає особливих рішень, які не були реалізовіні, кожен користувач зможете налаштувати свій власний планувальник Scheduler.

#### 4.Pipeline конвеєрне збереження даних

Конвеєрний обробник даних "Pipeline" відповідає за отримання результатів, включаючи перерахунки, зберігання їх у файли або в бази даних. Спосіб збереження результатів користувач може встановити через налаштуванн. За замовчення у WebMagic отримані результати виводяться "на консоль" (в пркладах є варіант збереження результатів у файл.

`Pipeline` визначає спосіб збереження результатів, якщо ви хочете зберегти у визначеній базі даних, тоді потрібно вказати відповідну обробку даних. Як правило потрібно тільки написати `Pipeline` та викликати один з статичних методів, що реалізує потрібний алгоритм збереження.

### 1.2.2 для об'єктів передачі даних

#### 1. Запит `Request`

`Request` - це пакетний рівень з URL-адресою, `Request` запит відповідний до URL-адреси.

Він є носієм для взаэмодії `PageProcessor` та `Downloader`. Для `Downloader` це єдиний спосіб впливати на `PageProcessor`.

На додаток до самого URL, він містить поля зі структурою ключ-значення `extra`. Ви можете зберегти деякі додаткові спеціальні `extra` атрибути, а потім прочитати в інших місцях для виконання різних функцій. Наприклад, деяка додаткова інформація на одній сторінці, що використана при роботі в іншій, і так далі.

#### 2. Сторінка Page

`Page` представленя сторінки `Downloader` для її завантаження - зміст може бути HTML, чи він може бути JSON або в інших текстових форматах.

Процес екстракції, який забезпечує способи отримання та зберегання результатів, сторінки `Page` прописаний у ядрі WebMagi. Наразі у [розділі 4](../../posts/ch4-basic-page-processor/selectable.md) ми будемо детально його розглядати.

#### 3. Результуюча одиниця ResultItems

`ResultItems` еквівалентно до колекції - `Map` у `Java`, яка проводить обробку результатів `PageProcessor` для подальшого використання у конвеерному збереженню результатів `Pipeline`.  API у нього та у мапи `Map` дуже схожі, окрім поля `skip` яке при встановлені `true` означає, що не повинен бути оброблений `Pipeline` конвеєром збереження результатів.

### 1.2.3 Керування запуском движка пошукача --Spider
Spider is the core WebMagic internal processes. A property Downloader, PageProcessor, Scheduler, Pipeline is the Spider, these properties can be freely set by setting this property can perform different functions. Spider WebMagic also operate the entrance, which encapsulates the creation of crawlers, start, stop, multi-threading capabilities. Here is a set of each component, and set an example of multi-threading and startup. See detailed Spider setting Chapter 4 - [crawler configuration, start and stop](../ch4-basic-page-processor/spider-config.html).

Павук Spider є внутрішнім процесом ядра WebMagic. Павук `Spider` має властивісті `Downloader`, `PageProcessor`, `Pipeline`, що можуть бути встановлені setter-ами у різні функції. Павук `Spider` WebMagic - це точка входу, яка інкапсулює створення пошукових роботів, запуск, зупинку, можливость роботи в багатопоточному рижимі. Ось набір кожного компонента і наведений приклад запуску у багатопоточності. Детальніше про налаштування павука у частині 4 - [Конфігурація пошукача, запуск і зупинка](../ch4-basic-page-processor/spider-config.html).

```java
public static void main(String[] args) {
    Spider.create(new GithubRepoPageProcessor())
            // From https://github.com/code4craft began to grasp    
            .addUrl("https://github.com/code4craft")
            // Set the Scheduler, use Redis to manage URL queue
            .setScheduler(new RedisScheduler("localhost"))
            // Set Pipeline, will result in json way to save a file
            .addPipeline(new JsonFilePipeline("D:\\data\\webmagic"))
            //Open 5 simultaneous execution threads
            .thread(5)
            //Start crawler
            .run();
}
```

### 1.2.4 Швидкий старт

Може скластися враження, що приведена велика кількість компонентів, але не турбуйтесь - бо для старту їх не потрібно багато конфігурувати, тому що значна частина модулів WebMagic вже встановлену реалізацію за замовчуванням.

Загалом, для отримання пошукача, необхідно написати класс з імплементацію `PageProcessor`, де `create`-том створюється `Spider` із початковими даними для сканеру. У розділі 4 ми розповімо як писати налаштовання шукача `PageProcessor` і `Spider` щоб розпочати - [Імплементація `PageProcessor` -а](../ch4-basic-page-processor/pageprocessor.md).
