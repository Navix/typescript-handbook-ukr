Цей запис описує різні способи організації коду з використанням модулів в TypeScript. Ми опишемо внутрішні та зовнішні модулі та роздивимось випадки у яких кожен з них підходить і як їх використовувати. Також ми роздивимось деякі просунуті теми з використання зовнішніх модулів, та вкажемо на поширені помилки при використанні модулів у TypeScript.

### Перші кроки

Почнемо з програми, яку ми будемо використовувати протягом цієї статті. Ми написали невеликий набір простих рядкових валідаторів, такі що ви могли б використовувати на формах або для перевірки даних від зовнішніх джерел.

#### Валідатору в одному файлі

```typescript
interface StringValidator {
    isAcceptable(s: string): boolean;
}

var lettersRegexp = /^[A-Za-z]+$/;
var numberRegexp = /^[0-9]+$/;

class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}

class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}

// Дані для прикладу
var strings = ['Hello', '98052', '101'];
// Валідатори
var validators: { [s: string]: StringValidator; } = {};
validators['ZIP code'] = new ZipCodeValidator();
validators['Letters only'] = new LettersOnlyValidator();
// Показуємо як кожен рядок пройшов валідацію
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

### Додаємо модульність

Ми будемо додавати більше валідаторів, отож нам потрібне щось на кшталт організаційної схеми, яка дозволить стежити за нашими типами та не турбуватися про збіг імен з іншими об’єктами. Замість того, щоб тримати велику кількість різноманітних імен у глобальному просторі імен, огорнемо наші об’єкти у модулі.

У цьому прикладі ми перемістили типи пов’язані з валідацією у модуль під назвою `Validation`. Ми хочемо, щоб інтерфейси та класи були доступні ззовні модулю, тому ми позначили їх ключовим словом `export`. З іншого боку змінні `lettersRegexp` та `numberRegexp` відносятся до внутрішньої реалізації, тому їх ми не експортуємо. У кінці файлу показано, як описувати виклики ззовні модулю, наприклад `Validation.LettersOnlyValidator`.

#### Валідатори в модулі

```typescript
module Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    var lettersRegexp = /^[A-Za-z]+$/;
    var numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}

// Дані для прикладу
var strings = ['Hello', '98052', '101'];
// Валідатори
var validators: { [s: string]: Validation.StringValidator; } = {};
validators['ZIP code'] = new Validation.ZipCodeValidator();
validators['Letters only'] = new Validation.LettersOnlyValidator();
// Показуємо як кожен рядок пройшов валідацію
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

## Розділення у різні файли

Коли наша програма росте, нам потрібно розділити код між файлами, щоб його було простіше підтримувати.

Тут ми розділили модуль `Validation` на багато файлів. Навіть при тому, що файли розділені, вони можуть бути частиною одного модулю і будуть працювати ніби визначенні в одному місці. Оскільки існують залежності між файлами, ми додали `reference`, щоб показати ці відносини компілятору. Функціональність нашого прикладу залишилась незмінною.

### Багатофайлові внутрішні модулі

#### Validation.ts

```typescript
module Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }
}
```

#### LettersOnlyValidator.ts

```typescript
/// <reference path="Validation.ts" />
module Validation {
    var lettersRegexp = /^[A-Za-z]+$/;
    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }
}
```

#### ZipCodeValidator.ts

```typescript
/// <reference path="Validation.ts" />
module Validation {
    var numberRegexp = /^[0-9]+$/;
    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}
```

#### Test.ts

```typescript
/// <reference path="Validation.ts" />
/// <reference path="LettersOnlyValidator.ts" />
/// <reference path="ZipCodeValidator.ts" />

// Дані для прикладу
var strings = ['Hello', '98052', '101'];
// Валідатори
var validators: { [s: string]: Validation.StringValidator; } = {};
validators['ZIP code'] = new Validation.ZipCodeValidator();
validators['Letters only'] = new Validation.LettersOnlyValidator();
// Показуємо як кожен рядок пройшов валідацію
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

Якщо ми використовуємо багато файлів, нам потрібно бути впевненими, що весь зкомпільований код був завантажений. Є два способи зробити це.

Перший, це використання модифікатора `--out` при компілюванні, щоб весь код був збережений в один JavaScript файл.

```
tsc --out sample.js Test.ts
```

Компілятор автоматично відсортує код в залежності від його залежностей у файлах. Також ви можете вручну вказати порядок підключення кожного файлу:

```
tsc --out sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts
```

Як альтернативу, ми можемо використовувати по-файлову компіляцію, щоб створити JavaScript файл на кожен TypeScript файл. У такому випадку нам потрібно використовувати `script` на нашій веб-сторінці для кожного потрібного файлу у правильному порядку, наприклад:

#### MyTestPage.html (витяг)

```html
    <script src="Validation.js" type="text/javascript" />
    <script src="LettersOnlyValidator.js" type="text/javascript" />
    <script src="ZipCodeValidator.js" type="text/javascript" />
    <script src="Test.js" type="text/javascript" />
```

## Зовнішні модулі

TypeScript також реалізує концепцію зовнішніх модулів. Зовнішні модулі використовуються у двох системах: node.js та require.js. Програмам які не використовують node.js або require.js не потрібні зовнішні модулі і вони можуть бути організовані з використанням внутрішніх модулів, які ми роздивилися раніше.

У зовнішніх модулях відносини між файлами засновані на імпорті та експорті на рівні файлів. У TypeScript кожен файл, який містить імпорт або експорт вищого рівня, вважається зовнішнім модулем.

Нижче ми переробимо попередній приклад на використання зовнішніх модулів. Зауважте, що ми більше не використовуємо ключове слово `module`, кожен файл модуль сам по-собі та ідентифікується по імені файлу.

Тег `reference` змінено на оголошення імпортів, які вказують на зв’язки між модулями. Оголошення імпорту має дві частини: ім’я по якому цей модуль буде відомим у поточному файлі та ключове слово `require` з вказанням шляху до файлу з модулем:

```typescript
import someMod = require('someModule');
````

Ми вказуємо на об’єкти, які видно ззовні модуля, за допомогою ключового слова `export`, відповідно до експорту у внутрішніх модулях.

При компілюванні потрібно вказати тип модульної системи: для node.js `--module commonjs`, для require.js `--module amd`:

```
tsc --module commonjs Test.ts
```

Кожен зовнішній модуль буде скомпільовано в окремий .js файл. Подібно до `reference` компілятор буде слідувати за оголошеннями імпорту, щоб скомпілювати пов’язані файли.

#### Validation.ts

```typescript
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

#### LettersOnlyValidator.ts

```typescript
import validation = require('./Validation');
var lettersRegexp = /^[A-Za-z]+$/;
export class LettersOnlyValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
```

#### ZipCodeValidator.ts

```typescript
import validation = require('./Validation');
var numberRegexp = /^[0-9]+$/;
export class ZipCodeValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

#### Test.ts

```typescript
import validation = require('./Validation');
import zip = require('./ZipCodeValidator');
import letters = require('./LettersOnlyValidator');

// Дані для прикладу
var strings = ['Hello', '98052', '101'];
// Валідатори
var validators: { [s: string]: validation.StringValidator; } = {};
validators['ZIP code'] = new zip.ZipCodeValidator();
validators['Letters only'] = new letters.LettersOnlyValidator();
// Показуємо як кожен рядок пройшов валідацію
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

### Генерація коду для зовнішніх модулів

В залежності від модульної системи, компілятор буде генерувати відповідний код для node.js (commonjs) або require.js (AMD). Щоб отримати більше інформації про те, що робить згенерований код, дізнайтесь в документації до кожної з модульних систем.

На простому прикладі показано як ключові слова конвертуються у код для модульних систем.

#### SimpleModule.ts

```typescript
import m = require('mod');
export var t = m.something + 1;
```

#### AMD / RequireJS SimpleModule.js:

```js
define(["require", "exports", 'mod'], function(require, exports, m) {
    exports.t = m.something + 1;
});
```

#### CommonJS / Node SimpleModule.js:

```js
var m = require('mod');
exports.t = m.something + 1;
```

## Export =

У попередньому прикладі, кожен валідатор повертав різні методи. У таких випадках не дуже зручно працювати з різними методами, коли достатньо буде одного ідентифікатора.

Конструкція `export =` вказує на один об’єкт який буде експортовано з модуля. Це може бути клас, інтерфейс, модуль, функція або enum. У цьому випадку при імпорті не потрібно додатково вказувати назву цього об’єкту.

Нижче ми спростимо реалізацію валідаторів, шляхом експорту одного об’єкту з кожного модулю з використанням конструкції `export =`. Це дозволить замість виклику `zip.ZipCodeValidator` просто викликати `zipValidator`.

#### Validation.ts

```typescript
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

#### LettersOnlyValidator.ts

```typescript
import validation = require('./Validation');
var lettersRegexp = /^[A-Za-z]+$/;
class LettersOnlyValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
export = LettersOnlyValidator;
```

#### ZipCodeValidator.ts

```typescript
import validation = require('./Validation');
var numberRegexp = /^[0-9]+$/;
class ZipCodeValidator implements validation.StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;
```

#### Test.ts

```typescript
import validation = require('./Validation');
import zipValidator = require('./ZipCodeValidator');
import lettersValidator = require('./LettersOnlyValidator');

// Дані для прикладу
var strings = ['Hello', '98052', '101'];
// Валідатори
var validators: { [s: string]: validation.StringValidator; } = {};
validators['ZIP code'] = new zipValidator();
validators['Letters only'] = new lettersValidator();
// Показуємо як кожен рядок пройшов валідацію
strings.forEach(s => {
    for (var name in validators) {
        console.log('"' + s + '" ' + (validators[name].isAcceptable(s) ? ' matches ' : ' does not match ') + name);
    }
});
```

## Синоніми

Інший шлях яким ви можете спросити роботу з модулями, це використовувати `import q = x.y.z`, щоб створити коротке ім’я для найчастіше використовуваних об’єктів. Не переплутайте це з конструкцією `import x = require('name')`, яку ми використовуємо для завантаження зовнішніх модулів. У даному випадку ми просто створюємо синонім на певний об’єкт. Ви можете використовувати імпорт цього роду для будь-якого типу ідентифікаторів, включаючи об’єкти створені зовнішніми модулями.

#### Базові синоніми

```typescript
module Shapes {
    export module Polygons {
        export class Triangle { }
        export class Square { }
    }
}

import polygons = Shapes.Polygons;
var sq = new polygons.Square(); // Те саме що 'new Shapes.Polygons.Square()'
```

Зауважте, що ми не використовували клюючове слово `require`, а напряму призначаємо ім’я, яке імпортуємо. Це рівнозначно до використання `var`, але також вказує на простір імен імпортованого об’єкту. Важливо, що для значень, імпорт — це відокремлене посилання на оригінал, тож зміни цих значень не будуть відбиватися на оригінальних змінних.

## Опціональне завантаження модулів та інші продвинуті сценарії

В деяких випадках вам може бути потрібне завантаження модулів тільки при виконанні певних умов. Нижче буде показано як це зробити в TypeScript та як реалізувати інші продвинуті сценарії прямого завантаження модулів без втрати безпеки типів.

Компілятор визначає, чи використовується кожен модуль в згенерованому JavaScript. Для модулів, які використовуються тільки в якості частини системи типу, завантаження не потрібне. Це вибракування невикористовуваних посилань є хорошою оптимізацією продуктивності, а також дозволяє опціональне завантаження цих модулів.

Головною ідеєю патерну є те, що конструкція `import id = require('...')` дає доступ до типів зовнішнього модуля. Завантажувач модулю буде викликано динамічно, як показано у блоках `if` нижче. У такому випадку модулі завантажуються тільки при потребі. Щоб це правильно працювало, важливо імопртовані модулі використовувати тільки в описах типів (в не там, де вони потраплять у сгенерований JavaScript).

Щоб підтримувати безпеку типів, ми можемо використовувати ключове слово `typeof`. Ключове слово `typeof`, коли ви використовуєте його при описі типу, вказує на тип значення, у цьому випадку на тип зовнішнього модулю.

#### Динамічне завантаження модулів у node.js

```js
declare var require;
import Zip = require('./ZipCodeValidator');
if (needZipValidation) {
    var x: typeof Zip = require('./ZipCodeValidator');
    if (x.isAcceptable('.....')) { /* ... */ }
}
```

#### Динамічне завантаження модулів у require.js

```js
declare var require;
import Zip = require('./ZipCodeValidator');
if (needZipValidation) {
    require(['./ZipCodeValidator'], (x: typeof Zip) => {
        if (x.isAcceptable('...')) { /* ... */ }
    });
}
```

## Робота з іншими JavaScript бібліотеками

Щоб описати інтерфейс бібліотек написаних не на TypeScript, нам потрібно описати API цієї бібліотеки. Зважаючи на те, що більшість JavaScript бібліотек створює лише кілька високорівневих об’єктів, модулі — це хороший спосіб представити їх. Ми називаємо опис, який не визначає реалізації "ambient". Зазвичай вони описані у .d.ts файлах. Якщо ви добре знайомі з C/C++, то це схоже на .h файли або `extern`. Подивимось на кілька прикладів.

### Внутрішні "ambient" модулі

Популярна бібліотека D3 створює глобальний об’єкт `d3`. Оскільки ця бібліотека завантажена за допомогою тегу `<script>` (а не за допомогою модулю), це буде внутрішнім описом. Щоб TypeScript знав інтерфейси ми використовуємо внутрішній опис. Наприклад:

#### D3.d.ts (спрощений уривок)

```typescript
declare module D3 {
    export interface Selectors {
        select: {
            (selector: string): Selection;
            (element: EventTarget): Selection;
        };
    }

    export interface Event {
        x: number;
        y: number;
    }

    export interface Base extends Selectors {
        event: Event;
    }
}

declare var d3: D3.Base;
```

### Зовнішні "ambient" модулі

У node.js у більшості випадків завантажує один або більше модулів. Нам потрібно описати кожен модуль у його власному .d.ts файлі. Але це краще ніж писати один великий .d.ts файл. Для цього ми використовуємо ім’я модуля у лапках, який у подальшому буде доступний для імпорту. Наприклад:

#### node.d.ts (спрощений уривок)

```typescript
declare module "url" {
    export interface Url {
        protocol?: string;
        hostname?: string;
        pathname?: string;
    }

    export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
}

declare module "path" {
    export function normalize(p: string): string;
    export function join(...paths: any[]): string;
    export var sep: string;
}
```

Тепер ми можемо написати `/// <reference> node.d.ts` і потім завантажувати модулі використовуючи конструкцію `import url = require('url');`.

```typescript
///<reference path="node.d.ts"/>
import url = require("url");
var myUrl = url.parse("http://www.typescriptlang.org");
```

## Поширені помилки

У цьому розділі ми роздивимось різноманітні пастки на шляху використання внутрішніх та зовнішніх модулів, та способи іх оминути.

### /// <reference> на зовнішній модуль

Поширеною помилкою є використання конструкції `/// <reference>` для завантаження зовнішнього модульного файлу, замість використання `import`. Щоб зрозуміти розбіжності, нам потрібно оглянути три способи, якими компілятор знаходить інформацію про зовнішні модулі.

По-перше, він намагається знайти .ts файл вказаний у `import x = require(...);`. Файл мусить містити реалізацію та високорівневі `export` або `import` описи.

По-друге, за допомогою пошуку .d.ts файлу, де замість реалізації буде лише високорівневий опис `export` або `import`.

І в останню чергу через пошук зовнішнього "ambient" опису, де є `declare` з ім’ям потрібного модулю у лапках.

#### myModules.d.ts

```typescript
// У .d.ts або .ts файлі це не є зовнішнім модулем:
declare module "SomeModule" {
    export function fn(): string;
}
```

#### myOtherModule.ts

```typescript
/// <reference path="myModules.d.ts" />
import m = require("SomeModule");
```

Тег `reference` дозволяє нам вказати на .d.ts файл який містить "ambient" опис зовнішнього модулю. Саме як файл `node.d.ts` у попередніх прикладах.

### Надлишковий простір імен

Якщо ви вирішили використовувати зовнішні модулі замість внутрішніх, дуже легко отримати файл, який виглядає ось так:

#### shapes.ts

```typescript
export module Shapes {
    export class Triangle { /* ... */ }
    export class Square { /* ... */ }
}
```

Модуль верхнього рівня `Shapes` огортає `Triangle` та `Square` без причини. Це збиває з пантелику і дратує користувачів вашого модулю:

#### shapeConsumer.ts

```typescript
import shapes = require('./shapes');
var t = new shapes.Shapes.Triangle(); // shapes.Shapes?
```

Ключова можливість зовнішніх модулів у TypeScript — два різні модулі ніколи не додадуть об’єкти в один простір видимості. Оскільки користувач зовнішнього модулю вирішує яке ім’я йому призначити, немає необхідності проактивно огортати експортовані об’єкти у свій простір імен.

Щоб упевнити вас, чому вам не потрібно створювати додатковий простір імен, нагадаємо, що основною ідеєю простору імен є логічне угруповання конструкцій та запобігання конфліктів імен. Оскільки зовнішній модуль сам по-собі логічне групування, а його високорівневе ім’я визначається у коді, який його імпортує, то відпадає необхідність у створенні додаткового прошарку для експортованих об’єктів.

Доопрацьований приклад:

#### shapes.ts

```typescript
export class Triangle { /* ... */ }
export class Square { /* ... */ }
```

#### shapeConsumer.ts

```typescript
import shapes = require('./shapes');
var t = new shapes.Triangle();
```

### Компроміси для зовнішніх модулів

Оскільки є взаємна відповідність одного JS-файлу до одного модулю, TypeScript має взаємну відповідніть одного зовнішнього модулю до одного скомпільованого JS-файлу. Одним з ефектів стає неможливість використання опції `--out` для об’єднання декількох зовнішніх модулів у один JavaScript файл.