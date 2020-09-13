---
id: function_components
title: 関数コンポーネント
---

関数コンポーネントは `props` 引数を受け取り JSX 要素を返す通常の関数のように書くことができます。

```tsx
type AppProps = { message: string }; /* interface も使用可能 */
const App = ({ message }: AppProps) => <div>{message}</div>;
```

<details>

<summary><b>`React.FC`/`React.FunctionComponent`/`React.VoidFunctionComponent`とは？</b></summary>

`React.FunctionComponent` を使って書くこともできます（省略形は `React.FC` で、意味は同じです）:

```tsx
const App: React.FunctionComponent<{ message: string }> = ({ message }) => (
  <div>{message}</div>
);
```

「通常の関数」といくつかの違いがあります：

- `React.FunctionComponent` は戻り値の型を明示的に指定しますが、通常の関数では暗黙的に指定します（あるいは追加のアノテーションが必要です）。

- また `displayName`, `propTypes`, `defaultProps` のような静的プロパティの型チェックと入力補完を提供します。

- `React.FunctionComponent` と一緒に `defaultProps` を使うと、いくつかの既知の問題があることに注意してください。詳細は [LibraryManagedAttributes が機能しない問題](https://github.com/typescript-cheatsheets/react-typescript-cheatsheet/issues/87) を参照してください。別の `defaultProps` セクションを用意していますので、そちらでも確認できます。

- 以下の関数コンポーネントは `children` の暗黙の定義を提供しています - しかし暗黙の `children` 型にはいくつかの問題があります（例：[DefinitelyTyped#33006](https://github.com/DefinitelyTyped/DefinitelyTyped/issues/33006)）。

```tsx
const Title: React.FunctionComponent<{ title: string }> = ({
  children,
  title,
}) => <div title={title}>{children}</div>;
```

<details>
<summary>

[@types/react PR #46643](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/46643)（TODO: @types/react のバージョンをマージして更新）の時点で、受け入れられる `children` を明示的に宣言したい場合は、新しい `React.VoidFunctionComponent` または `React.VFC` 型を使用することができるようになりました。これは、FunctionComponent がデフォルトで子要素を受け付けないようになるまでの暫定的な解決策です（React 18 で予定）。

</summary>

```ts
type Props = { foo: string };

// 現在はOK、将来エラーになります
const FunctionComponent: React.FunctionComponent<Props> = ({
  foo,
  children,
}: Props) => {
  return (
    <div>
      {foo} {children}
    </div>
  ); // OK
};

// 現在はエラー、将来、廃止されます
const VoidFunctionComponent: React.VoidFunctionComponent<Props> = ({
  foo,
  children,
}) => {
  return (
    <div>
      {foo}
      {children}
    </div>
  );
};
```

</details>

- _In the future_, it may automatically mark props as `readonly`, though that's a moot point if the props object is destructured in the parameter list.

- 将来的には、自動的に `readonly` を props オブジェクトに付記するようになるかもしれませんが、パラメータリストで props オブジェクトが破壊された場合には意味がありません。

ほとんどの場合、どちらの構文を使用するかの違いはほとんどありませんが、より明示性のある `React.FunctionComponent` のほうが好ましいかもしれません。

</details>

<details>
<summary><b>小さな落とし穴</b></summary>

以下のパターンはサポートされていません：

**Conditional rendering**

```tsx
const MyConditionalComponent = ({ shouldRender = false }) =>
  shouldRender ? <div /> : false; // JS でもやらないでください
const el = <MyConditionalComponent />; // エラー
```

これはコンパイラの制限により、関数コンポーネントは JSX 式か `null` 以外の型を返すことができないためです。さもないと、他の型は `Element` に代入できないという暗号のようなエラーメッセージを表示します。

```tsx
const MyArrayComponent = () => Array(5).fill(<div />);
const el2 = <MyArrayComponent />; // エラー
```

**Array.fill**

残念ながら、関数の型をアノテーションするだけでは何の役にも立ちません。そのため React がサポートする他のエキゾチックな型を返す必要が本当にある場合は、型アサーションを実行する必要があります。

```tsx
const MyArrayComponent = () => (Array(5).fill(<div />) as any) as JSX.Element;
```

詳しくは [@ferdaber のコメント](https://github.com/typescript-cheatsheets/react-typescript-cheatsheet/issues/57) を参照してください。

</details>
