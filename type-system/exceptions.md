# 例外のハンドリング

JavaScriptには、例外のために使用できる`Error`クラスがあります。あなたは`throw`キーワードでエラーを投げます。あなたは`try`/`catch`ブロックの対でそれを捕まえることができます。

```javascript
try {
  throw new Error('Something bad happened');
}
catch(e) {
  console.log(e);
}
```

## エラーサブタイプ

ビルトインされた`Error`クラスのほかに、JavaScriptランタイムが投げることができる`Error`を継承するいくつかの組み込みエラークラスがあります：

### RangeError

数値変数またはパラメータが有効な範囲外にあるときに発生するエラーを表すインスタンスを作成します。

```javascript
// Call console with too many arguments
console.log.apply(console, new Array(1000000000)); // RangeError: Invalid array length
```

### ReferenceError

参照されたのが無効な参照だった場合に発生するエラーを表すインスタンスを作成します。例えば

```javascript
'use strict';
console.log(notValidVar); // ReferenceError: notValidVar is not defined
```

### SyntaxError

有効でないJavaScriptを解析する際に発生する構文エラーを表すインスタンスを作成します。

```javascript
1***3; // SyntaxError: Unexpected token *
```

### TypeError

変数またはパラメータが有効な型でないときに発生するエラーを表すインスタンスを作成します。

```javascript
('1.2').toPrecision(1); // TypeError: '1.2'.toPrecision is not a function
```

### URIError

`encodeURI()`または `decodeURI()`に無効なパラメータが渡されたときに発生するエラーを表すインスタンスを生成します。

```javascript
decodeURI('%'); // URIError: URI malformed
```

## 常にエラーを使用する

初心者のJavaScriptデベロッパーは、たまに生の文字列のみをエラーとして投げます。

```javascript
try {
  throw 'Something bad happened';
}
catch(e) {
  console.log(e);
}
```

_これはしないでください_。`Error`オブジェクトの基本的な利点は、`stack`プロパティを使ってビルドされた場所を自動的に追跡することです。

生の文字列は非常に苦しいデバッグ経験をもたらし、ログからのエラー分析を複雑にします。

## あなたはエラーをスローする必要はありません

`Error`オブジェクトを渡すことは大丈夫です。これは、Node.jsのコールバックスタイルコードでは、最初の引数をエラーオブジェクトとしてコールバックを受け取ります。

```typescript
function myFunction (callback: (e?: Error)) {
  doSomethingAsync(function () {
    if (somethingWrong) {
      callback(new Error('This is my error'))
    } else {
      callback();
    }
  });
}
```

## 例外的なケース

「例外は例外的でなければならない」は、コンピュータサイエンスの一般的な言葉です。これがJavaScript\(およびTypeScript\)にも当てはまる理由はいくつかあります。

### どこに投げられるのか不明

次のコードを考えてみましょう。

```javascript
try {
  const foo = runTask1();
  const bar = runTask2();
}
catch(e) {
  console.log('Error:', e);
}
```

次の開発者は、どの関数がエラーを投げるかわかりません。コードをレビューしている人は、task1/task2のコードとそれが呼び出す可能性のある他の関数を読み取ることなく知ることができません。

### 優雅なハンドリングを困難にする

スローする可能性のあるものの周りを明示的にキャッチして優雅にしようとすることができます：

```javascript
try {
  const foo = runTask1();
}
catch(e) {
  console.log('Error:', e);
}
try {
  const bar = runTask2();
}
catch(e) {
  console.log('Error:', e);
}
```

しかし、最初のタスクから2番目のタスクに物事を渡す必要がある場合、コードは乱雑になります。\(`foo`のミューテションに`let` が必要になる + 明示的に型をアノテートしなければならなくなることに注意してください。これは、`runTask1`の戻り値から型を推測できないためです\) ：

```typescript
let foo: number; // Notice use of `let` and explicit type annotation
try {
  foo = runTask1();
}
catch(e) {
  console.log('Error:', e);
}
try {
  const bar = runTask2(foo);
}
catch(e) {
  console.log('Error:', e);
}
```

### 型システムでうまく表現されていない

次の関数を考えてみましょう。

```typescript
function validate(value: number) {
  if (value < 0 || value > 100) throw new Error('Invalid value');
}
```

このような場合に`Error`を使うのは、validate関数の型定義\(`(value:number) => void`\)で表現されていないので、悪い考えです。代わりに、検証メソッドを作成するためのより良い方法は次のとおりです。

```typescript
function validate(value: number): {error?: string} {
  if (value < 0 || value > 100) return {error:'Invalid value'};
}
```

そして今、型システムで表現されています。

> 非常に一般的な\(シンプル/キャッチオールなど\)の方法でエラーを処理しない限り、エラーをスローしないでください。
