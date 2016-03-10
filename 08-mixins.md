Поряд з традиційними ОО ієрархіями, інший популярний спосіб будування класів з багаторазових компонентів — це комбінування простих часткових класів (partial classes). Ви можете бути знайомі з ідеєю домішків (mixins) або типажів (traits) у таких мовах, як Scala.

## Приклад домішків

У коді нижче буде показано, як ви можете використовувати домішки у TypeScript. Після цього ми пояснимо, як це працює.

```typescript
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }

}

// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}

class SmartObject implements Disposable, Activatable {
    constructor() {
        setInterval(() => console.log(this.isActive + " : " + this.isDisposed), 500);
    }

    interact() {
        this.activate();
    }

    // Disposable
    isDisposed: boolean = false;
    dispose: () => void;
    // Activatable
    isActive: boolean = false;
    activate: () => void;
    deactivate: () => void;
}
applyMixins(SmartObject, [Disposable, Activatable])

var smartObj = new SmartObject();
setTimeout(() => smartObj.interact(), 1000);

////////////////////////////////////////
// In your runtime library somewhere
////////////////////////////////////////

function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        })
    });
}
```

## Розбираємо приклад

Приклад починається з двох класів, які будуть нашими домішками. Ви можете побачити, як кожен з них орієнтований на певну діяльність або функції. пізніше ми створюємо змішаний клас, який включає їх можливості.

```typescript
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }

}

// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}
```

Тепер створимо клас, який буде обробляти комбінацію з двох домішків:

```typescript
class SmartObject implements Disposable, Activatable {
```

Першу річ, яку потрібно відзначити, це використання `implements` замість `extends`. Це працює як розширення інтерфейсів та додає до класу тільки типи `Disposable` та `Activatable` без її реалізації. Це означає, що нам потрібно буде забезпечити реалізацію у класі. Але, це саме те, чого ми зможемо уникнути за допомогою домішків.

Для виконання цієї вимоги, ми створюємо параметри з типами, які прийдуть з домішків. Це покаже компілятору, що ці члени будуть доступні при виконанні програми. Навіть ці "накладні" дії дозволяють отримати користь від домішків.

```typescript
// Disposable
isDisposed: boolean = false;
dispose: () => void;
// Activatable
isActive: boolean = false;
activate: () => void;
deactivate: () => void;
```

Нарешті, ми домішуємо реалізацію у клас:

```typescript
applyMixins(SmartObject, [Disposable, Activatable])
```

Нарешті, ми створюємо допоміжну функцію, яка реалізує домішування. Вона переносить всі параметри кожного з домішків у клас, який ми розширюємо:

```typescript
function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        })
    });
}
```