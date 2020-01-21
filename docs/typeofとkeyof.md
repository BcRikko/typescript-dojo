typeofとkeyof
====


typeofとは
---

変数を型化できる。

```ts
const　obj = {
  foo: 'foo',
  bar: 'bar'
}

type ObjType = typeof obj
// {
//   foo: string
//   bar: string
// }
```


keyofとは
----

プロパティ名を型にする。イメージ的には`Object.keys()`に近い。

```ts
interface MyObj {
  foo: 'hoge',
  bar: 'fuga'
}

type ObjKeyType = keyof MyObj
// "foo" | "bar"
```

オブジェクトリテラルからプロパティ名の型を作るときは、`typeof`と併用するとできる。

```ts
const obj = {
  foo: 'hoge',
  bar: 'fuga'
}

type ObjType = typeof obj
type ObjKeyType = keyof ObjType
// "foo" | "bar"
```


keyofの使いみち
----

オブジェクトを作成するときにプロパティ名を制限したいときに使える。

```ts
const obj = {
  foo: 'hey',
  bar: 123
}

type ObjType = typeof obj
// {
//   foo: string
//   bar: number
// }

type ObjKeyType = keyof ObjType
// "foo" | "bar"

type MyObj = {
  [key in ObjKeyType]: ObjType[key]
}

const myObj: MyObj = {
  foo: 'string',
  // foo: 100,      // ← ERROR:Type 'number' is not assignable to type 'string'.
  bar: 100,
  // bar: 'string'  // ← ERROR:Type 'string' is not assignable to type 'number'.
  // baz: 'error'   // ← ERROR:'baz' does not exist in type 'MyObj'
}
```

* `[key in ObjKeyType]`はObjKeyTypeに含まれる型（今回は"foo"と"bar"）のみに限定している
  * MyObj型ではプロパティ名に"foo"|"bar"しか指定できないので、"baz"プロパティを作ろうとするとエラーになる
* `[key in ObjKeyType]`で指定されたkeyを元に、`ObjType[key]`とすると、fooならstring、barならnumberしか受け付けなくなる



keyofの実践的な使い方
----

HTML要素を扱うクラスを想定する。

### そのままつくると…

なんの制限もないため、変なタグの要素がつくられてしまう。

```ts
class MyElement {
  el: HTMLElement
  constructor(tagName: string) {
    this.el = document.createElement(tagName)
  }
}

const myElement = new MyElement('hoge')
// myElement.el = <hoge></hoge>
```


### keyofを使って有効なタグ名のみに制限する

`keyof`を使って`lib.dom.d.ts`の`HTMLElementTagNameMap`で定義されているタグ名のみを受け取る。

```ts
// lib.dom.d.ts
interface HTMLElementTagNameMap {
  "a": HTMLAnchorElement;
  "abbr": HTMLElement;
  "address": HTMLElement;
  ...
}
```

```ts
class MyElement {
  el: HTMLElement
  constructor(tagName: keyof HTMLEementTagNameMap) {
    this.el = document.createElement(tagName)
  }
}

const myElement = new MyElement('hoge')
// Argument of type '"hoge"' is not assignable to parameter of type

const myElement = new MyElement('a')
// myElement.el = <a></a>
// actual: myElement.el = HTMLElement
// expect: myElement.el = HTMLAnchorElement
```


### myElement.elをより詳細で適切な型にする

keyofとジェネリクスを使って、HTMLElementではなくHTMLAnchorElementやHTMLDivElementを取得する。

```ts
class MyElement<TagName extends keyof HTMLElementTagNameMap> {
  el: HTMLElementTagNameMap[TagName]

  constructor(tagName: TagName) {
    this.el = document.createElement(tagName)
  }
}

const myElement = new MyElement('a')
// myElement.el = HTMLAnchorElement
```


TIPS💡
----

### 型Mapが便利

JavaScriptでも対応表みたいなマップをつくることがある。

```js
class Server {}
class Disk {}
class Hub {}

const model = {
  server: Server,
  disk: Disk,
  hub: Hub
}

const getModel = (type) => model[type]

// JSだとどんなクラスが返ってくるかわからない
const server = getModel('server') // typeof Server | Disk | Hub
const disk   = getModel('disk')   // typeof Server | Disk | Hub
const hub    = getModel('hub')    // typeof Server | Disk | Hub
```

JavaScriptだと`getModel()`の戻り値がServer|Disk|Hubのどれか、というところまでしか絞り込めない。

TypeScriptで型情報を追加して、期待する型を得る。

```ts
class Server {}
class Disk {}
class Hub {}

const modelMap = {
  server: Server,
  disk: Disk,
  hub: Hub
}

type ModelMapType = typeof modelMap

const getModel = <T extends keyof ModelMapType>(type: T): ModelMapType[T] => modelMap[type]

// リテラルを渡しても期待する型が返ってくる
const server = getModel('server') // typeof Server
const disk   = getModel('disk')   // typeof Disk
const hub    = getModel('hub')    // typeof Hub
```


ちなみに`'server'`のようなリテラルも、`keyof ModelMapType`を使っているので自動補完される。
