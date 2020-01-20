TypeScriptとは
====

* JavaScriptのすげーやつ。  
  （スーパーセットという）
* 型の表現力がすげー。


……の前に、JavaScriptのツラミは？
----

* ゆるふわ言語（動的型付け）
* typoしてランタイムエラーになる
* 自動補完が息してない


TypeScriptを使うと
-----

* キッチリカッチリ言語（静的型付け）
* typoした時点でエラーになる
* 自動補完バリバリ


JavaScriptとTypeScriptのエラータイミング
----

### JavaScript

```js
let num = 100;
num = 'hundred'

console.log(num)
// "hundred"

const obj = {}
console.log(obj.hey.yo)
// 実行時にエラーになる
// Cannot read property 'yo' of undefined
```

### TypeScript

```ts
let num = 100;
num = 'hundred'
// 型が違うのでエラーになる
// Type '"100"' is not assignable to type 'number'

const obj = {}
console.log(obj.hey.yo)
// objに`hey`がないというエラーになる
// Property 'hey' does not exist on type '{}'
```

### まとめ

* JavaScriptは**実行時**にエラーになる
* TypeScriptは**実行前**にエラーになる（コンパイラが教えてくれる）
