Функції є основним будівельним блоком програм у JavaScript, вони є складовими прошарків абстракцій, імітують класи, приховують інформацію, виступають модулями. У TypeScript поряд з класами та модулями функції відіграють ключову роль у тому, як і що робити. Також TypeScript додає деякі нові можливості до стандартних JavaScript функцій, щоб зробити роботу з ними зручнішою.

## Функції

По-перше, так само, як і JavaScript, функції TypeScript можуть бути іменованими або анонімними. Це дозволяє вам вибрати найбільш зручний підхід для вашої програми.

Ось як ці два підходи виглядаюсь у JavaScript:

```typescript
// Іменована функція
function add(x, y) {
    return x+y;
}

// Анонімна функція
var myAdd = function(x, y) { return x+y; };
```

Також, як у JavaScript, функції можуть отримувати доступ до змінних, які є ззовні функції. Розуміння того, як це працює та коли варто використовувати такий прийом, виходить за межі цього розділу, але розуміння цієї механіки дуже важлива для роботи з JavaScript та TypeScript.

```typescript
var z = 100;

function addToZ(x, y) {
    return x+y+z;
}
```

## Типи функцій

### Типізуємо функцію

Давайте додамо типи до базового прикладу:

```typescript
function add(x: number, y: number): number {
    return x+y;
}

var myAdd = function(x: number, y: number): number { return x+y; };
```

Ми можемо додати тип до кожного з параметрів та до повертаємого функцією значення. TypeScript з’ясовує тип повертаємого типу по `return`, тому ви можете не вказувати його у багатьох випадках.

### Описуємо тип функції

Тепер, коли ми додали типи до функції, опишемо повний тип функції.

```typescript
var myAdd: (x:number, y:number)=>number =
    function(x: number, y: number): number { return x+y; };
```

Тип функції має дві частини: типи аргументів, та поветраємий тип. Коли ви описуєте тип функції обидві частини обов’язкові. Ми описуємо типи параметрів, просто як їх список, надаючі кожному ім’я та тип. Ім’я вказується для кращої читабельності.

```typescript
var myAdd: (baseValue:number, increment:number)=>number =
    function(x: number, y: number): number { return x+y; };
```

Для перевірки типів неважливо які імена ви дасте параметрам, необхідно лише дотримуватися порядку.

У другій частині ми вказуємо тип повертаємо значення. Як ми казали раніше, це обов’язкова частина для типу функції, ото ж якщо функція нічого не повертає, вам потрібно вказувати `void`.

Зауважимо, що тільки параметри та повертаємий тип складають тип функції. Зовнішні змінні, до яких функція може отримати доступ, не відображаються у цьому типи, вони є частиною прихованого стану функції та не можуть бути складовою API.

### Визначення типів

Ви могли помітити, що компілятор TypeScript визначає тип, навіть якщо ви його явно не вказали:

```typescript
// myAdd має повний опис типу
var myAdd = function(x: number, y: number): number { return x+y; };

// Параметри 'x' та 'y' мають тип number
var myAdd: (baseValue:number, increment:number)=>number =
    function(x, y) { return x+y; };
```

Це називається "контекстне типізування", одна з форм визначення типів. Це допомагає зменшити кількість дій для підтримки всіх даних типізованими.

## Необов’язкові та параметри за замовчуванням

На відміну від JavaScript, у TypeScript кожен оголошений параметр повинен бути при її виклику. Це не значить, що вона не може бути `null`, але коли функція викликаеться, компілятор перевіряє, що користувач передав усі параметри. Також компілятор перевіряє, що тільки потрібні параметри передані. Якщо коротше, потрібно передати точну кількість параметрів, які функція очікує.

```typescript
function buildName(firstName: string, lastName: string) {
    return firstName + " " + lastName;
}

var result1 = buildName("Bob");  // помилка, параметрів замало
var result2 = buildName("Bob", "Adams", "Sr.");  // помилка, параметрів забагато
var result3 = buildName("Bob", "Adams");  // ось так правильно
```

У JavaScript кожен параметр є необов’язковим і користувач може їх не передавати. У цьому випадку вони будуть `undefined`. Ми можемо отримати цю функціональність у TypeScript за допомогою `?` за ім’ям параметру, який ми хочемо зробити необов’язковим. Наприклад, зробимо `lastName` необов’язковим:

```typescript
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

var result1 = buildName("Bob");  // тепер цей виклик також працює
var result2 = buildName("Bob", "Adams", "Sr.");  // помилка, параметрів забагато
var result3 = buildName("Bob", "Adams");  // ось так правильно
```

Необов’язкові параметри повинні йти за обов’язковими. Якщо ми хочемо зробити `firstName` необов’язковим, на відміну від `lastName`, тоді нам потрібно поміняти порядок параметрів у функції, та поставити `firstName` у кінець.

У TypeScript ми також можемо виставляти значення у необов’язкових параметрів, на випадок коли вони не передані. Це називається значення за замовчуванням. Візьмемо попередній приклад, та вкажемо для `lastName` значення за замовчунням `Smith`.

```typescript
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

var result1 = buildName("Bob");  // тепер цей виклик також працює
var result2 = buildName("Bob", "Adams", "Sr.");  // помилка, параметрів забагато
var result3 = buildName("Bob", "Adams");  // ось так правильно
```

Також як і з необов’язковими параметрами, параметри за замовчуванням повинні ідти у кінці списку.

## Залишкові параметрі

Обов’язкові, необов’язкові та параметри за замовчуванням мають одну спільну рису: за один раз вони описують один параметр. Іноді вам потрібно працювати з набором або групою параметрів, або ви можете навіть не знати скільки параметрів передадуть у функцію. У JavaScript ви можете напряму працювати зі всіма аргументу через змінну `agruments`.

В TypeScript ви можете зібрати ці аргументи в одну змінну:

```typescript
function buildName(firstName: string, ...restOfName: string[]) {
	return firstName + " " + restOfName.join(" ");
}

var employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

Залишкові параметрі розглядаються як необмежена кількість необов’язкових параметрів. Ви можете їх не передати або передавати будь-яку їх кількість. Компілятор створить масив цих аргументів з ім’ям яке ви зазначите після трикрапки (ellipsis).

Ellipsis також необхідно вказувати у типи функції, якщо там необхідні залишкові параметри:

```typescript
function buildName(firstName: string, ...restOfName: string[]) {
	return firstName + " " + restOfName.join(" ");
}

var buildNameFun: (fname: string, ...rest: string[])=>string = buildName;
```

## Лямбди та використання 'this'

Дуже поширеним питанням для початківців у JavaScript — використання `this`. TypeScript є надбудовою над JavaScript, тому потрібно розуміти як працює `this` і що потрібно робити, коли він використовується невірно. Ціла стаття могла бути написана про використання `this` у JavaScript, але зараз зосередимося на основах.

В JavaScript `this` це змінна, яка виставляється у момент виклику функції. Це робить її дуже потужною та гнучкою, але ціною цього виступає необхідність знати у якому оточенні функція викликаєтся. Це може збивати з пантелику, наприклад, коли функція використовується як колбек.

Подивимося на приклад:

```typescript
var deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        return function() {
            var pickedCard = Math.floor(Math.random() * 52);
            var pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

var cardPicker = deck.createCardPicker();
var pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

Якщо ви спробуєте запустити цей приклад, то маєте отримати помилку, замість очікуваного віконця повідомлення. Причина у тому, що `this` використовується функцією створеною у `createCardPicker`, буде належати до `window`, а не об’єкту `deck`. Це стається у результаті виклику `cardPicker()`, тут немає динамічного зв’язування для `this` ні з ким, крім `window`. (Примітка: у строгому режимі замість `window` буде `undefined`).

Ми можемо виправити це, переконавшись, що функція пов'язана з правильним `this`, перш ніж ми повертаємо функцію, яка буде використовуватися в подальшому. Таким чином, незалежно від того, як ми його пізніше використовуємо, він все одно матиме можливість побачити оригінальний об'єкт `deck`.

Щоб виправити це, ми оголошуємо функцію за допомогою лямбда синтаксису (`()=>{}`). Це автоматично захопить `this` для функції із того місця, де вона створена, а не викликана.

```typescript
var deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        // Примітка: рядок нижче — це лямбда, яка дозволяє нам використовувати 'this'
        return () => {
            var pickedCard = Math.floor(Math.random() * 52);
            var pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

var cardPicker = deck.createCardPicker();
var pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

## Перезавантаження

JavaScript по-суті дуже динамічна мова. Не є незвичним, коли одна функція у JavaScript повертає різні типи об’єктів в залежності від параметрів, які були передані.

```typescript
var suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x): any {
    // Перевірити що ми працюємо з об’єктом або масивом
    // якщо так, тоді вибрати та повернути карту
    if (typeof x == "object") {
        var pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Інашке дамо вибрати карту
    else if (typeof x == "number") {
        var pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

var myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
var pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

var pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

Тут функція `pickCard` повертає різні речі в залежності від того, що користувач передав. Якщо користувач передасть об’єкт який представляє стіл, функція вибере карту. Якщо користувач вибере карту, ми скажемо йому, яку саме карту він вибрав. Але як це описати у системі типів?

Відповідь лежить у описі декількох типів для однієї функції через список перезавантажень. Цей список вказує компілятору, що йому потрібно використовувати для перевірки виклику функцій. Давайте створимо список перезавантажень, який опише, що приймає функція `pickCard` і що вона повертає у цих випадках.

```typescript
var suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // Перевірити що ми працюємо з об’єктом обо масивом
        // якщо так, тоді вибрати та повернути карту
    if (typeof x == "object") {
        var pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Інашке дамо вибрати карту
    else if (typeof x == "number") {
        var pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

var myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
var pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

var pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

Тепер перезавантаження дають нам перевірку типів для всіх викликів функції `pickCard`.

Для того, щоб компілятор міг вибрати правильну перевірку типів, він використовує процес подібний до JavaScript. Він перевіряє список перезавантажень та вибирає перше перезавантаження, яке відповідає переданим параметрам. Якщо він знаходить відповідність, він вибирає це перезавантаження, як правильне. По цій причині краще описувати перезавантаження від більш спеціфичних, до менш специфічних.

Зауважте, що `function pickCard(x): any` не є частиною списку перезавантажень, тому вона має тільки два перезавантаження:  одне бере об’єкт, друге бере число. Виклик `pickCard` з іншими параметрами призведе до помилки.
