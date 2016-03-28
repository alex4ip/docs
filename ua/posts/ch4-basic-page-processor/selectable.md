## 4.2 Вибіркова послідовність API

Послідовніть API `Selectable` актуальним є основною функцією WebMagic. Можливість вибору інтерфейсу для використання, ви можете завершити ланцюжок вилучення безпосередньо до елементів сторінки, і потрібно подбати про деталі відкликані.

Можна побачити в попередньому прикладі, page.getHtml () повертає об'єкт `Html`, який реалізує інтерфейс `Selectable`. Цей інтерфейс містить ряд важливих способів, я розділити його на дві категорії: часткове розтин екстракції і отримати результати.

#### Селектрори для екстракції v.4.2.1 API:

| метод | опис | приклади |
| ------------ | ------------- | ------------ |
| xpath(String xpath) | використання XPath селектора  | html.xpath("//div[@class='title']") |
| $(String selector) | використання CSS селектора  | html.$("div.title") |
| $(String selector,String attr) | використання CSS селектора  | html.$("div.title","text") |
| css(String selector) | функції з $(), використання CSS селектора  | html.css("div.title") |
| links() | вибраті усі лінки  | html.links() |
| regex(String regex) | використання регулярний вираз для екстракції  | html.regex("\<div\>(.\*?)\</div>") |
| regex(String regex,int group) | використання регулярний вираз для екстракції та вказати группу захоплення  | html.regex("\<div\>(.\*?)\</div>",1) |
| replace(String regex, String replacement) | Замінити вміст  | html.replace("\<script>.\*\</script>","")|

Ця частина витягується API повертає ключ `інтерфейси Selectable`, а це означає, що витяг підтримується прикутих викликів. Дозвольте мені використовувати приклад, щоб пояснити використання ланцюга API.

Наприклад, зараз я хочу, щоб захопити всі проекти Java на GitHub, ці елементи можуть бути [https://github.com/search?l=Java&p=1&q=stars%3A%3E1&s=stars&type=Repositories](https:// github.com/search?l=Java&p=1&q=stars%3A%3E1&s=stars&type=Repositories) побачити результати пошуку.

Щоб уникнути сканування занадто широкий, я вказати обмеження перевірки посилання з секції закладок. Правила обходу контенту є більш складними, я б, як писати?

![selectable-chain-ui](http://webmagic.qiniudn.com/oscimages/151454_2T01_190591.png)

По-перше, дивіться стор HTML структура виглядає наступним чином:

![selectable-chain](http://webmagic.qiniudn.com/oscimages/151632_88Oq_190591.png)

Тоді я можу використовувати CSS селектори, щоб витягти цей DIV, а потім взяти всі посилання. Для цілей страхування, я буду використовувати регулярні вирази, щоб визначити, що формат витягнутої URL, а потім остаточне формулювання виглядає так:

```java
List<String> urls = page.getHtml().css("div.pagination").links().regex(".*/search/\?l=java.*").all();
```

Тоді ми можемо поставити ці URL доданий до списку для сканування:

```java
List<String> urls = page.getHtml().css("div.pagination").links().regex(".*/search/\?l=java.*").all();
page.addTargetRequests(urls);
```

Це не просто? На додаток до пошуку по посиланню, ланцюги звертається за вибором також може зробити багато роботи. У розділі 9 ми знову вище приклади.

#### 4.2.2 API для отримання результатів :

Коли ланцюг виклику, ми звичайно хочемо отримати тип рядка результату. На цей раз ми повинні використовувати API для отримання результатів. Ми знаємо, що правила вилучення, або XPath, CSS селектори або регулярний вираз, завжди можна витягти безліч елементів. WebMagic вони були об'єднані, ви можете отримати один або кілька елементів за допомогою різних API.

| метод | опис | приклади |
| ------------ | ------------- | ------------ |
| get() | повертає результат типу String | String link= html.links().get()|
| toString() | функція get(), повертає результат типу String | String link= html.links().toString()|
| all() | повертає всі результати вилучення | List<String> links= html.links().all()|
| match() | це match result | if (html.links().match()){ xxx; }|

Наприклад, ми знаємо, що сторінка буде мати результат, ви можете використовувати selectable.get() або selectable.toString(), щоб отримати цей результат.

Тут selectable.toString() використовує ToString() цей інтерфейс, а також для виробництва і рамок в поєднанні, більш зручними. Тому що при нормальних обставинах, нам потрібно тільки вибрати один елемент!

selectable.all() отримає до всіх елементів.

Ну, до сих пір, озирнутися назад на 3.1 GithubRepoPageProcessor, ви можете відчувати себе більш ясно, чи не так? Визначає основний метод, результати вже можна побачити плазує в консолі виводу.