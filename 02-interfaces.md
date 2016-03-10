Одним з основних принціпів TypeScript є зосередження на "формі" значення при перевірці типів. Це інколи називають "duck typing" або "structural subtyping". В TypeScript інтерфейси відіграють роль задання цих типів та є потужним способом визначення контрактів в вашому коді, а також за його межами.

## Наш перший інтерфейс

Найпростішим способом побачити як працюють інтерфеси буде почати з легкого прикладу:

```typescript
function printLabel(labelledObj: {label: string}) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

Компілятор перевіряє виклик до `printLabel`. `printLabel` має один параметр, котрий вимагає об’єкт з параметром `label` типу `string`. Прошу зауважити, що переданий об’єкт має більше параметрів, ніж описано, але компілятор перевіряє лише необхідні.

Ми можемо написати цей приклад ще раз, але цього разу використаємо інтерфейс, щоб вказати необхідність у параметрі `label` з типом `string`:

```typescript
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

Тепер інтерфейс `LabelledValue` ми можемо використовувати для зазначення вимог описаних у попередньому прикладі. Він все ще вказує на наявність одного параметру `label` з типом `string`. Зверніть увагу на те, що ми не вказуємо, що переданний об’єкт реалізує інтерфейс, як ми це робимо у інших мовах. Тут тільки форма має значення: якщо переданий об’єкт задовольняє нашим умовам — він дозволяєтся.

Варто відзначити, що компілятор не перевіряє послідовність властивостей, важлива лише наявність та типова відповідність.

## Необов’язкові параметри

Не всі параметри інтерфейсу обов’язкові. При певних умовах параметри можуть бути відсутні. Такі параметри популярні при використанні таких шаблонів проектування, як "option bags", коли користувач передає до функції об’єкт в якому заповнені лише пара властивостей.

Ось приклад цього шаблону:

```typescript
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});
```

Інтерфейси з необов’язковими параметрами схожі на інші інтерфейси, але кожен такий параметр позначений символом '?' при оголошенні.

Перевага необов’язкових параметрів у тому, що ви можете описати всі можливі параметри, навіть ті що не завжди передаються. Наприклад, якщо ми зробимо опечатку у написанні назви параметру, що передаємо до функції `createSquare`, то ми отримаємо сповіщення про цю помилку:

```typescript
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  var newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.collor;  // Тайпчекер виявив помилку
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

var mySquare = createSquare({color: "black"});
```

## Типи функцій

Інтерфейси здатні описати широкий набір форм, яких набувають об’єкти в JavaScript. У додаток до опису об’єктів з їх параметрами, інтерфейси здатні описувати типи функцій.

Щоб описати функцію за допомогою інтерфейсу, ми вказуємо що він може бути викликаним. Це як опис функції але лише зі списком параметрів та типом повертаємого значення.

```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```

Після визначення ми можемо використовувати цей інтерфейс як і інші. Ось показано, як ви можете створити змінну з вказанням типу функції та призначити їй функцію з відповіднім описом.

```typescript
var mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  var result = source.search(subString);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```

Для типів функцій не обов’язково щоб назви параметрів збігались. Ми можемо написати попередній приклад ось так:

```typescript
var mySearch: SearchFunc;
mySearch = function(src: string, sub: string) {
  var result = src.search(sub);
  if (result == -1) {
    return false;
  }
  else {
    return true;
  }
}
```

Також автоматично визначаєтся, що повертає наша функція та перевіряється відповідність цього значення до інтрефейсу. У нашому випадку це true/false, але якщо ми спробуємо повернути рядок або цифри, компілятор попередить нас про невідповідність типів.

## Типи масивів

Аналогічно до того, як ми описуємо типи функцій, ми можемо описати типи масивів. Тип масивів має 'index' тип, що описує типи дозволенні для ключів об’єкту, поряд вказано відповідний тип повертаємих данних при запиту по такому ключу.

```typescript
interface StringArray {
  [index: number]: string;
}

var myArray: StringArray;
myArray = ["Bob", "Fred"];
```

There are two types of supported index types: string and number. It is possible to support both types of index, with the restriction that the type returned from the numeric index must be a subtype of the type returned from the string index.

While index signatures are a powerful way to describe the array and 'dictionary' pattern, they also enforce that all properties match their return type. In this example, the property does not match the more general index, and the type-checker gives an error:

```typescript
interface Dictionary {
  [index: string]: string;
  length: number;    // error, the type of 'length' is not a subtype of the indexer
}
```

## Типи класів

### Реалізація інтерфейсу

Одним з найбільш поширеніших застосувань інтерфейсів у таких мовах як C# та Java є явне зазначення, що клас відповідає конкретному контракту. Це також можливо у TypeScript.

```typescript
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

Також ви можете описати в інтерфейсі методи, які потрібно реалізувати для класу, так як ми це зробили з 'setTime' у наступному прикладі:

```typescript
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface  {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

Інтерфейси описують публічну сторону класу, а не публічну разом з приватною. Це забороняє вам використовувати інтерфейси для перевірки типів приватних методів та властивостей.

### Різниця між static/instance використанням класу

Коли ви працюєте з класами та інтерфейсами, допомагає держати у голові, що клас дві частини: статичну та об’єктну. Ви можете помітити, що при описі інтерфейсу з методом конструктора та спробі створити клас, який реалізує цей інтерфейс, ви отримаєте помилку:

```typescript
interface ClockInterface {
    new (hour: number, minute: number);
}

class Clock implements ClockInterface  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

Причина у тому, що коли клас реалізує інтерфейс, перевіряється тільки екземплярна частина. У той час як конструктор перебуває в статичній частині та не потрапляє під перевірку.

Замість цього слід було звертатися до статичної частини класу напряму. В цьому прикладі ми працюємо з класом напряму:

```typescript
interface ClockStatic {
    new (hour: number, minute: number);
}

class Clock  {
    currentTime: Date;
    constructor(h: number, m: number) { }
}

var cs: ClockStatic = Clock;
var newClock = new cs(7, 30);
```

## Розширення інтерфейсів

Як класи, інтерфейси можуть розширяти один одного. Це вирішує завдання з копіюванням членів одного інтерфейсу в інший, що дозволяє вам відчути більше свободи у відокремленні ваших інтерфейсів в багаторазові компоненти.

```typescript
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

var square = <Square>{};
square.color = "blue";
square.sideLength = 10;
```

Інтерфейс может розширити декілька інших інтерфейсів, створюючи при цьому комбінацію з усіх цих інтерфейсів в одному.

```typescript
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

var square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

## Гібридні типи

Як ми вже згадували раніше, інтерфейси можуть описати різноманітність типів, присутніх у JavaScript. Через динамічний і гнучкий характер JavaScript, ви можете мати справу з об’єктом, який працює як поєднання деяких з описаних вище типів.

Одним з таких прикладів є об’єкт, який діє як функція і як об’єкт з додатковими властивостями:

```typescript
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

var c: Counter;
c(10);
c.reset();
c.interval = 5.0;
```

При взаємодії з зовнішнім JavaScript кодом, вам може знадобитися використання шаблонів, як вище, щоб повністю описати форму типу.