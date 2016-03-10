Традиційно JavaScript фокусується на функціях та прототипному успадкуванню в якості основного засобу створення багаторазових компонентів, але це трохи незграбний спосіб і більш комфортним є об’єктно-орієнтований підхід, де класи успадковують функціонал та об’єкти будуються на основі цих класів. Починаючи з ECMAScript 6, наступною версії JavaScript, програмісти будуть мати можливість будувати свої програми використовуючи ООП. У TypeScript ми дозволяємо розробникам використовувати ці техніки вже зараз та компілювати код у JavaScript, який буде працювати на всіх основних браузерах та платформах.

## Класи

Давайте подивимося на простий приклад використання класів:

```typescript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

var greeter = new Greeter("world");
```

Синтаксис повинен бути вам знайомим, якщо ви використовували C# або Java раніше. Ми описуємо новий клас `Greeter`. Цей клас складається з трьох членів: властивості `greeting`, конструктора та методу `greet`.

Ви можете помітити, що коли ми звертаємося до одного з членів класу, то пишемо перед його ім’ям `this`.

На останньому рядку ми створюємо екземпляр класу Greeter за допомогою `new`. Так ми викликаємо конструктор, який описали раніше, створюємо новий об’єкт відповідний до Greeter, та запускаємо конструктор, щоб ініціалізувати його.

## Успадкування

В TypeScript ми можемо використовувати поширені об’єктно-орієнтовні прийоми. Авжеж, одним із основних прийомів є можливість розширювати існуючі класи для створення нових.

Подивимося на приклад:

```typescript
class Animal {
    name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number = 0) {
        alert(this.name + " moved " + meters + "m.");
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(meters = 5) {
        alert("Slithering...");
        super.move(meters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(meters = 45) {
        alert("Galloping...");
        super.move(meters);
    }
}

var sam = new Snake("Sammy the Python");
var tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

Цей приклад показує основні можливості успадкування у TypeScript, які є загальними і у інших мовах. Тут ми бачимо використання ключового слова `extends` для створення підкласу. `Horse` та `Snake` — це нащадки базового класу `Animal`, які мають доступ до його можливостей.

Також цей приклад показує можливість перевизначення методів основного класу у підкласі. Так `Snake` та `Horse` мають власну реалізацію методу `move`, яка має свою специфічність у кожному випадку.

## Private/Public модифікатори

### Публічні за замовчуванням

Ви, можливо, помітили що у наведених вище прикладах ми не використовували ключове слово `public` ні до одного з членів, щоб вказати їх публічність. Такі мови як C# вимагають, щоб кожен член був явно позначений `public`, щоб його було "видно". У TypeScript усі члени публічні за замовчуванням.

Ви можете визначати приватні члени і так контролювати що є загальнодоступним за межами вашого класу. Ми могли б описати клас `Animal` так:

```typescript
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```

### Розуміння приватного

Коли ми порівнюємо два різні типи, незважаючи на їх походження, якщо типи кожного з членів збігаються, тоді ми можемо сказати що і типи є однаковими.

Коли ми порівнюємо два типи з приватними членами, ми ставимося до них по-різному. Для двох типів, які будуть вважатися сумісними, якщо один з них має приватний елемент, то інший повинен мати приватний елемент, який був оголошений у тому ж місці.

Подивимося на приклад, щоб краще зрозуміти яку роль це буде відігравати на практиці:

```typescript
class Animal {
    private name:string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
	constructor() { super("Rhino"); }
}

class Employee {
    private name:string;
    constructor(theName: string) { this.name = theName; }
}

var animal = new Animal("Goat");
var rhino = new Rhino();
var employee = new Employee("Bob");

animal = rhino;
animal = employee; // помилка: Animal та Employee не сумісні
```

У цьому прикладі ми маємо `Animal` та `Rhino`, де `Rhino` є підкласом `Animal`. Також ми маємо новий клас `Employee`, який з точки зору форми відповідає `Animal`. Ми створюємо екземпляри цих класів та намагаємося призначити один одному, щоб побачити що трапиться. Через те що `Animal` та `Rhino` мають спільну приватну частину їх оголошення (`private name: string` у `Animal`), то вони є сумісними. Проте ситуація з `Employee` інша, коли ми намагаємося призначити `Employee` до `Animal`, то отримуємо помилку несумісності типів. Незважаючи на те що `Employee` має приватний параметр `name`, це не той самий параметр, що був при створенні `Animimal`.

### Параметри контруктора

Ключові слова `public` та `private` також дають вам можливість швидкого створення та ініціалізації параметрів вашого класу, за допомогою параметрів конструктора. Ось подальший перегляд попереднього прикладу. Зауважте як `private name: string` у конструкторі дозволяє нам створити та ініціалізувати параметр `name`.

```typescript
class Animal {
    constructor(private name: string) { }
    move(meters: number) {
        alert(this.name + " moved " + meters + "m.");
    }
}
```

Використання `private` створює та ініціалізує приватний параметр класу, аналогічним чином буде працювати і `public`.

## Accessors

TypeScript підтримує геттери/сеттери шляхом перехоплення звертання до параметру об’єкту. Це дає вам можливість мати більше контролю над тим, як звертаються до параметрів у кожному об’єкті.

Переробимо простий клас на використання `get` та `set`. Почнемо з класу без геттерів та сеттерів.

```typescript
class Employee {
    fullName: string;
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

Доки ми дозволяємо випадково встановлювати fullName напряму, це може поставити нас у складну ситуацію.

У цій версії ми перевіряємо, що користувач знає секретний пароль, перед тим як дозволяємо модифікувати дані. Це робиться через підміну прямого доступу до `fullName` за допомогою `set`, в якому ми перевіряємо пароль. Також ми описуємо відповідний `get`, щоб попередній приклад працював без проблем.

```typescript
var passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            alert("Error: Unauthorized update of employee!");
        }
    }
}

var employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

Щоб довести самому собі, що наш модифікатор перевіряє код доступу, ми можемо змінити `passcode` і побачити попередження на відсутність прав доступу на зміну даних.

## Статичні атрибути

До цього моменту ми розглядали роботу з членами екземплярів, які з’являються коли ми створюємо об’єкт. Також ми можемо створювати статичні члени класу, які доступні у самому класі без створення єкзмепляру. У цьому прикладі ми робимо статичним `origin` і це буде основним значенням для всіх об’єктів. Кожен екземпляр звертається до цього значення через ім’я класу. Аналогічно до `this.` при звертанні до параметру об’єкту. Отож тут ми пишемо `Grid.` для статичного звернення.

```typescript
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        var xDist = (point.x - Grid.origin.x);
        var yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

var grid1 = new Grid(1.0);  // 1x scale
var grid2 = new Grid(5.0);  // 5x scale

alert(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
alert(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

## Просунуті техніки

### Функції конструктора

Коли ви описуєте клас у TypeScript, насправді створюється декілька декларацій одночасно.

```typescript
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

var greeter: Greeter;
greeter = new Greeter("world");
alert(greeter.greet());
```

Коли ми пишемо `var greeter: Greeter`, ми використовуємо `Greeter` як тип екземпляру з класом `Greeter`.

Також ми створюємо функцію конструктора, яка викликається, коли ми створюємо новий екземпляр за допомогою `new`. Щоб побачити, що це значить на практиці, подивимось на JavaScript код, який був створений з попереднього прикладу:

```typescript
var Greeter = (function () {
    function Greeter(message) {
        this.greeting = message;
    }
    Greeter.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    return Greeter;
})();

var greeter;
greeter = new Greeter("world");
alert(greeter.greet());
```

Тут `var Greeter` буде призначено функцію конструктора. Коли ми викликаємо `new` та запускаємо цю функцію, ми отримуємо екземпляр класу. Також функція конструктора має всі статичні члени класу. Іншими словами кожен клас має об’єктну сторону та статичну сторону.

Давай модифікуємо приклад, щоб побачити різницю:

```typescript
class Greeter {
    static standardGreeting = "Hello, there";
    greeting: string;
    greet() {
        if (this.greeting) {
            return "Hello, " + this.greeting;
        }
        else {
            return Greeter.standardGreeting;
        }
    }
}

var greeter1: Greeter;
greeter1 = new Greeter();
alert(greeter1.greet());

var greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";
var greeter2:Greeter = new greeterMaker();
alert(greeter2.greet());
```

У цьому прикладі `greeter1` працює як і раніше. Ми створюємо екземпляр класу `Greeter` та використовуємо цей об’єкт. Це ми бачили раніше.

Далі ми використовуємо клас напряму. Тут ми створюємо нову змінну `greeterMaker`. Ця змінна містить сам клас, або кажучи інакше функцію конструктора. За допомогою `typeof Greeter` ми вказуємо що це сам тип класу Greeter, аніж на тип об’єкту. Цей тип містить все статичні члени `Greeter` разом з функцією, яка створює екземпляр класу. Ми показали це, коли викликали `new greeterMaker()` та створили об’єект класу `Greeter`.

### Клас у ролі інтерфейсу

Як ми казали у попередньому розділі, оголошення класу створює дві речі: тип який представляє екземпляри класу та функцію конструктора. Оскільки класи створюють типи, ви можете використовувати їх у тих самих місцях де ви можете використовувати інтерфейси.

```typescript
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

var point3d: Point3d = {x: 1, y: 2, z: 3};
```