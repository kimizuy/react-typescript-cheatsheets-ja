---
id: hooks
title: Hooks
---

Hooks は [`@types/react`](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/a05cc538a42243c632f054e42eab483ebf1560ab/types/react/index.d.ts#L800-L1031) でバージョン 16.8 以上からサポートされています。

## useState

型推論はほとんどの場合で有効に機能します。

```tsx
const [val, toggle] = React.useState(false); // val` は boolean であると推測され、`toggle` は boolean のみを受け取ります。
```

推論に頼った複雑な型を使う必要がある場合は、[型推論](https://react-typescript-cheatsheet.netlify.app/docs/basic/troubleshooting/types/#using-inferred-types)のセクションも参照してください。

しかし、多くのフックは null-ish なデフォルト値で初期化されており、どのように型定義するか悩むかもしれません。次の例では明示的に型定義しユニオン型を使用しています。

```tsx
const [user, setUser] = React.useState<IUser | null>(null);

// later...
setUser(newUser);
```

## useReducer

Reducer の動作には [Discriminated Unions（判別共用体）](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-func.html#discriminated-unions) を使うことができます。

```tsx
const initialState = { count: 0 };

type ACTIONTYPE =
  | { type: "increment"; payload: number }
  | { type: "decrement"; payload: string };

function reducer(state: typeof initialState, action: ACTIONTYPE) {
  switch (action.type) {
    case "increment":
      return { count: state.count + action.payload };
    case "decrement":
      return { count: state.count - Number(action.payload) };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = React.useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: "decrement", payload: "5" })}>
        -
      </button>
      <button onClick={() => dispatch({ type: "increment", payload: 5 })}>
        +
      </button>
    </>
  );
}
```

[View in the TypeScript Playground](https://www.typescriptlang.org/play?#code/LAKFEsFsAcHsCcAuACAVMghgZ2QJQKYYDGKAZvLJMgOTyEnUDcooRsAdliuO+IuBgA2AZUQZE+ZAF5kAbzYBXdogBcyAAwBfZmBCIAntEkBBAMIAVAJIB5AHLmAmgAUAotOShkyAD5zkBozVqHiI6SHxlagAaZGgMfUFYDAATNXYFSAAjfHhNDxAvX1l-Q3wg5PxQ-HDImLiEpNTkLngeAHM8ll1SJRJwDmQ6ZIUiHIAKLnEykqNYUmQePgERMQkY4n4ONTMrO0dXAEo5T2aAdz4iAAtkMY3+9gA6APwj2ROvImxJYPYqmsRqCp3l5BvhEAp4Ow5IplGpJhIHjCUABqTB9DgPeqJFLaYGfLDfCp-CIAoEFEFeOjgyHQ2BKVTNVb4RF05TIAC0yFsGWy8Fu6MeWMaB1x5K8FVIGAUglUwK8iEuFFOyHY+GVLngFD5Bx0Xk0oH13V6myhplZEm1x3JbE4KAA2vD8DFkuAsHFEFcALruAgbB4KAkEYajPlDEY5GKLfhCURTHUnKkQqFjYEAHgAfHLkGb6WpZI6WfTDRSvKnMgpEIgBhxTIJwEQANZSWRjI5SdPIF1u8RXMayZ7lSphEnRWLxbFNagAVmomhF6fZqYA9OXKxxM2KQWWK1WoTW643m63pB2u+7e-3SkEQsPamOGik1FO55p08jl6vdxuKcvv8h4yAmhAA)

<details>

<summary><b>`redux` での `Reducer` の使い方</b></summary>

[redux](https://github.com/reduxjs/redux)ライブラリを使って Reducer 関数を書く場合、`Reducer<State, Action>`形式の便利なヘルパーを提供しています。

そのため、上記の Reducer の例は次のようになります：

```tsx
import { Reducer } from 'redux';

export function reducer: Reducer<AppState, Action>() {}
```

</details>

## useEffect

`useEffect`を使うときは、関数や `undefined` 以外のものを返さないように注意してください。さもないと React と TypeScript の両方から怒られます。アロー関数を使う場合にはわかりにくいことがあります。

```ts
function DelayedEffect(props: { timerMs: number }) {
  const { timerMs } = props;
  // 悪い例：アロー関数の本体が中括弧で包まれていないためエラーになります。
  useEffect(
    () =>
      setTimeout(() => {
        /* 処理 */
      }, timerMs),
    [timerMs]
  );
  return null;
}
```

## useRef

`useRef`を使う場合、初期値を持たない Ref コンテナを作成する際には、2 つのオプションがあります。

```ts
const ref1 = useRef<HTMLElement>(null!);
const ref2 = useRef<HTMLElement | null>(null);
```

最初のオプションは `ref1.current` を読み取り専用にし、React が管理する組み込みの `ref` 属性に渡すことを意図しています（React が `current` の値の設定を処理してくれるからです）。

<details>
  <summary><code>null!</code> の最後の <code>!</code> はなに？</summary>

`null!` は non-null アサーション演算子 (`!`) です。この演算子は、その前の式が `null` や `undefined` ではないことを保証します。したがって、`useRef<HTMLElement>(null!)` の場合は、現在の値が `null` である ref のインスタンスを作成しているが、TypeScript には `null` ではないと嘘をついていることを意味します。

```ts
function MyComponent() {
  const ref1 = useRef<HTMLElement>(null!);
  useEffect(() => {
    doSomethingWith(ref1.current); // ref1 と ref1.current で TypeScript は null チェックを必要としません。
  });
  return <div ref={ref1}> etc </div>;
}
```

</details>

2 番目のオプションは `ref2.current` を可変にするもので、自分で管理する「インスタンス変数」のためのものです。

```tsx
function TextInputWithFocusButton() {
  // null で初期化しますが、HTMLInputElement を探していることを TypeScript に伝えます。
  const inputEl = React.useRef<HTMLInputElement>(null);
  const onButtonClick = () => {
    // 厳密な null チェックでは inputEl と current が存在するかどうかをチェックする必要があります。
    // current が存在すれば、それは HTMLInputElement 型なので focus() を持っています! ✅
    if (inputEl && inputEl.current) {
      inputEl.current.focus();
    }
  };
  return (
    <>
      {/* なお、inputElはinput要素でしか使えません。やったー! */}
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

[View in the TypeScript Playground](https://www.typescriptlang.org/play/?jsx=2#code/JYWwDg9gTgLgBAJQKYEMDG8BmUIjgIilQ3wFgAoCzAVwDsNgJa4AVJADxgElaxqYA6sBgALAGIQ01AM4AhfjCYAKAJRwA3hThwA9DrjBaw4CgA2waUjgB3YSLi1qp0wBo4AI35wYSZ6wCeYEgAymhQwGDw1lYoRHCmEBAA1oYA5nCY0HAozAASLACyADI8fDAAoqZIIEi0MFpwaEzS8IZllXAAvIjEMAB0MkjImAA8+cWl-JXVtTAAfEqOzioA3A1NtC1wTPIwirQAwuZoSV1wql1zGg3aenAt4RgOTqaNIkgn0g5ISAAmcDJvBA3h9TsBMAZeFNXjl-lIoEQ6nAOBZ+jddPpPPAmGgrPDEfAUS1pG5hAYvhAITBAlZxiUoRUqjU6m5RIDhOi7iIUF9RFYaqIIP9MlJpABCOCAUHJ0eDzm1oXAAGSKyHtUx9fGzNSacjaPWq6Ea6gI2Z9EUyVRrXV6gC+DRtVu0RBgxuYSnRIzm6O06h0ACpIdlfr9jExSQyOkxTP5GjkPFZBv9bKIDYSmbNpH04ABNFD+CV+nR2636kby+BETCddTlyo27w0zr4HycfC6L0lvUjLH7baHY5Jas7BRMI7AE42uYSUXed6pkY6HtMDulnQruCrCg2oA)

example from [Stefan Baumgartner](https://fettblog.eu/typescript-react/hooks/#useref)

## useImperativeHandle

_この節の解説は多くありません。コード例は [イシューでの議論](https://github.com/typescript-cheatsheets/react/issues/106) からのものです。_

```tsx
type ListProps<ItemType> = {
  items: ItemType[];
  innerRef?: React.Ref<{ scrollToItem(item: ItemType): void }>;
};

function List<ItemType>(props: ListProps<ItemType>) {
  useImperativeHandle(props.innerRef, () => ({
    scrollToItem() {},
  }));
  return null;
}
```

## Custom Hooks

Custom Hooks で配列を返す場合、TypeScript は（実際には配列の各位置に異なる型が欲しい場合に）共用体型を推論します。こういった型推論を避けたいならば代わりに [TS 3.4 const アサーション](https://devblogs.microsoft.com/typescript/announcing-typescript-3-4/#const-assertions) を使用してください：

```tsx
export function useLoading() {
  const [isLoading, setState] = React.useState(false);
  const load = (aPromise: Promise<any>) => {
    setState(true);
    return aPromise.finally(() => setState(false));
  };
  return [isLoading, load] as const; // (boolean | typeof load)[] の代わりに [boolean, typeof load] を推論します。
}
```

[View in the TypeScript Playground](https://www.typescriptlang.org/play/?target=5&jsx=2#code/JYWwDg9gTgLgBAJQKYEMDG8BmUIjgcilQ3wFgAoCpAD0ljkwFcA7DYCZuRgZyQBkIKACbBmAcwAUASjgBvCnDhoO3eAG1g3AcNFiANHF4wAyjBQwkAXTgBeRMRgA6HklPmkEzCgA2vKQG4FJRV4b0EhWzgJFAAFHBBNJAAuODjcRIAeFGYATwA+GRs8uSDFIzcLCRgoRiQA0rgiGEYoTlj4xMdMUR9vHIlpW2Lys0qvXzr68kUAX0DpxqRm1rgNLXDdAzDhaxRuYOZVfzgAehO4UUwkKH21ACMICG9UZgMYHLAkCEw4baFrUSqVARb5RB5PF5wAA+cHen1BfykaksFBmQA)

よって、デストラクチャ時にはデストラクチャの位置に基づいた正しい型を実際に取得できます。

<details>
<summary><b>代替案：タプルの戻り値の型を定義する</b></summary>

[const アサーションで困っている](https://github.com/babel/babel/issues/9800)なら、関数の戻り値の型を定義することもできます。

```tsx
export function useLoading() {
  const [isLoading, setState] = React.useState(false);
  const load = (aPromise: Promise<any>) => {
    setState(true);
    return aPromise.finally(() => setState(false));
  };
  return [isLoading, load] as [
    boolean,
    (aPromise: Promise<any>) => Promise<any>
  ];
}
```

Custom Hooks を多く書く場合は、タプルを自動的に型付けしてくれるヘルパー関数も便利です。

```ts
function tuplify<T extends any[]>(...elements: T) {
  return elements;
}

function useArray() {
  const numberValue = useRef(3).current;
  const functionValue = useRef(() => {}).current;
  return [numberValue, functionValue]; // type is (number | (() => void))[]
}

function useTuple() {
  const numberValue = useRef(3).current;
  const functionValue = useRef(() => {}).current;
  return tuplify(numberValue, functionValue); // type is [number, () => void]
}
```

</details>

しかし、React チームは、2 つ以上の値を返す Custom Hooks はタプルではなく適切なオブジェクトを使用することを推奨していることに注意してください。

もっと Hooks + TypeScript を読む：

- https://medium.com/@jrwebdev/react-hooks-in-typescript-88fce7001d0d
- https://fettblog.eu/typescript-react/hooks/#useref

If you are writing a React Hooks library, don't forget that you should also expose your types for users to use.

React Hooks ライブラリを書く場合は、ユーザが使えるように型を公開することも忘れてはいけません。
Example React Hooks + TypeScript Libraries:

- https://github.com/mweststrate/use-st8
- https://github.com/palmerhq/the-platform
- https://github.com/sw-yx/hooks

[何か追加したいことがあればイシューをください](https://github.com/typescript-cheatsheets/react-typescript-cheatsheet/issues/new)。
