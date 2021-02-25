# Identity bridge to operate Realm functions in both ways

In Caridy's explainer of Isolated Realms, we found a challenge to make cross-realm bridge functions operate in both ways.

This document extends Caridy's API to transfer identity to allow the Realms to operate in both ways.

## API (in typescript notation)

```ts
declare class Realm {
    constructor();
    static from(id: symbol): Realm;
    get id(): Symbol;
    eval(sourceText: string): any;
    Function(...args: string[]): Function;
    AsyncFunction(...args: string[]): AsyncFunction;
    import(specifier: string): Promise<???>;
}
```

## Namings:

- Blue: the Incubator, Outter Realm
- Red: the inner Realm. A realm created by an incubator.

## Realm#id

When a new Realm instance is created, it saves a Symbol that is an id reference for that Realm. This symbol should be stored in an internal mapping of instanced Realms. This internal is set in the `Realm` constructor.

When the Realm constructor is called with a new target:

  - ...
  - Assert: `Realm.[[OutterRealm]]` is initially set to `~empty~`.
  - Assert: `this` is the new Realm instance
  - Let `this.[[RealmId]]` be a new Symbol.
  <!--
  - Assert: `Realm.[[InnerRealms]]` is a List of records `{{Key}, {Value}}`.
  - Let __mapped__ be the Record of `{ {Key}: this.[[RealmId]], {Value}: this }`. // Creates a Record with the new instance 
  - Add __mapped__ to `Realm.[[InnerRealms]]`.
  -->
  - Bridge: Add `this.[[RealmId]]` to `this.[[BridgeToGlobalThis]].Realm.[[OutterRealm]]`.
  - ...

When `get id()` is called:

  ...
  - // Asserts `this` has a `[[RealmId]]` internal.
  - Returns `this.[[RealmId]]`.


## `Realm.from`

`Realm.from` creates a new instance referencing to an existing Realm. When `Realm.from` is called with the argument __id__:

  ...
  - Asserts `this` is a valid `Realm` constructor.
  - If __id__ has not the same primitive value of `Realm.[[OutterRealm]]`, throws a TypeError
  - Constructs a new instance of Realm withoout creating a new Realm, but bridging to the identified OutterRealm.
  ...

## Examples

### Blue Realm

```js
var hello = false; // FWIW, this is a global
var something;

const red = new Realm();

const shareIdentity = red.Function('id', 'return globalThis.__blue__ = id');
shareIdentity(red.id);

console.log(hello); // false
console.log(something); // undefined

await red.import('./realmCode.js'); // or any available code injection

console.log(hello); // true, see Red Realm code bellow
consonle.log(something); // undefined

// The cross-realm bridge going back and forth
console.log(red.eval('doSomething()')); // yields 42;

console.log(something); // 42;
```

### Red Realm

```js
// realmCode.js
const blue = Realm.from(globalThis.__blue__);

blue.eval('hello = true;'); // Always an indirect eval

const doSomething = blue.Function('v', 'return globalThis.something = v');
```
