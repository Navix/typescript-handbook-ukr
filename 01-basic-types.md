Для зручної роботи з програмами, нам потрібно мати можливість працювати з найпростішими одиницями даних: числами, рядками, структурами, логічними значеннями і т.п. У TypeScript ми підтримуємо майже ті самі типи даних, які є у JavaScript.

## Boolean

Найпростіший тип даних - це `true`/`false` значення, котре JavaScript та TypeScript (а також інші мови) називають `boolean`.

```typescript
var isDone: boolean = false;
```

## Number

Так само, як в JavaScript, усі числа у TypeScript це значення з плаваючою комою. Ці значення мають тип `number`.

```typescript
var height: number = 6;
```

## String

Інша фундаментальна частина створення програм за допомогою JavaScript, для веб-сторінок, а також серверної сторони, це робота з текстовими даними. Як і в інших мовах, ми використовуємо тип `string`, щоб вказати на текстовий тип даних. Саме як JavaScript, TypeScript використовує подвійні кавички (") або одинарні кавички (') щоб огортати рядки.

```typescript
var name: string = "bob";
name = 'smith';
```

## Array

TypeScript, як JavaScript, дозволяє вам працювати з масивами значень. Тип масивів може бути вказаний двома шляхами. У першому випадку ви вказуєте тип елементів з `[]` у кінці, позначаючи масив єлементів такого типу:

```typescript
var list:number[] = [1, 2, 3];
```

Другий шлях — використання масиву з [узагальненим типом](/generics), `Array<elemType>`:

```typescript
var list:Array<number> = [1, 2, 3];
```

## Enum

Корисний додаток до стандартного набору типів, які ми знаємо по JavaScript, це `enum`. Подібно до мови C#, enum — це спосіб дати більш привітні назви наборам числових значень.

```typescript
enum Color {Red, Green, Blue};
var c: Color = Color.Green;
```

За замовчанням, enum починає нумерацію своїх членів з 0. Ви можете вручну виставити інші значення. Наприклад, в наступному прикладі ми почнемо нумерацію з 1, а не з 0:

```typescript
enum Color {Red = 1, Green, Blue};
var c: Color = Color.Green;
```

Або виставимо вручну значення усіх членів:

```typescript
enum Color {Red = 1, Green = 2, Blue = 4};
var c: Color = Color.Green;
```

Зручною фічею enum є можливість отримати його назву з числового значення. Наприклад, якщо ми маємо значення 2, але не впевнені с якою назвою воно пов’язано в `enum Color`, ми можемо отримати відповідне ім’я:

```typescript
enum Color {Red = 1, Green, Blue};
var colorName: string = Color[2];

alert(colorName);
```


## Any

Можливо нам потрібно оголосити змінну, але ми не знаємо її типу у процесі розробки. Ці значення нам можуть надходити динамічно (від юзера або зі сторонніх бібліотек). В цих випадках нам потрібно пропустити перевірку типів на етапі компіляції. Для цього використовується тип `any`:

```typescript
var notSure: any = 4;
notSure = "maybe a string instead";
notSure = false;
```

Використання типу `any` це потужній спосіб для роботи з існуючим JavaScript, дозволяючи вам поступово впроваджувати перевірку типів на етапі компіляції.

Також `any` зручний у випадку, коли ви знаєте тип тільки частини даних, але не всіх. Наприклад, ви можете мати масив змішаних типів:

```typescript
var list:any[] = [1, true, "free"];

list[1] = 100;
```

## Void

У якомусь сенсі протилежність до `any` - це `void`, відсутність будь якого із типів. Зазвичай ви можете побачити цей тип у функцій які не повертають жодного значення:

```typescript
function warnUser(): void {
    alert("This is my warning message");
}
```