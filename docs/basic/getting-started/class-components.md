---
id: class_components
title: クラスコンポーネント
---

TypeScript の中では、`React.Component` はジェネリック型（`React.Component<PropType, StateType>`）なので、（任意で）プロップ型とステート型のパラメータを指定できます：

```tsx
type MyProps = {
  // interface も使用できます
  message: string;
};
type MyState = {
  count: number; // like this
};
class App extends React.Component<MyProps, MyState> {
  state: MyState = {
    // 戻り値の型を任意で指定することで型推論の精度を上げることができます
    count: 0,
  };
  render() {
    return (
      <div>
        {this.props.message} {this.state.count}
      </div>
    );
  }
}
```

[View in the TypeScript Playground](https://www.typescriptlang.org/play/?jsx=2#code/JYWwDg9gTgLgBAJQKYEMDG8BmUIjgcilQ3wFgAoCmATzCTgFlqAFHMAZzgF44BvCuHAD0QuAFd2wAHYBzOAANpMJFEzok8uME4oANuwhwIAawFwQSduxQykALjjsYUaTIDcFAL4fyNOo2oAZRgUZW4+MzQIMSkYBykxEAAjFTdhUV1gY3oYAAttLx80XRQrOABBMDA4JAAPZSkAE05kdBgAOgBhXEgpJFiAHiZWCA4AGgDg0KQAPgjyQSdphyYpsJ5+BcF0ozAYYAgpPUckKKa4FCkpCBD9w7hMaDgUmGUoOD96aUwVfrQkMyCKIxOJwAAMZm8ZiITRUAAoAJTzbZwIgwMRQKRwOGA7YDRrAABuM1xKN4eW07TAbHY7QsVhsSE8fAptKWynawNinlJcAGQgJxNxCJ8gh55E8QA)

エクスポートやインポート、拡張によって types/interfaces が再利用できることも覚えておきましょう。

<details>
<summary><b>なぜ<code>state</code>を二度定義するのか</b></summary>

クラスプロパティ `state` にアノテーションをつけることは厳密には必要ではありませんが、`this.state` にアクセスしたり状態を初期化したりする際に、より良い型推論ができるようになります。

これは 2 つの異なる方法で動作するからです。2 番目のジェネリック型パラメータで `this.setState()` が正しく動作するようになります。

詳しくは [@ferdaber のコメント](https://github.com/typescript-cheatsheets/react-typescript-cheatsheet/issues/57)を参照してください。

</details>

<details>
  <summary><b><code>readonly</code>はもう必要ない</b></summary>

よくサンプルコードに `readonly` が含まれているのを見かけますが、これはプロップやステートがイミュータブル（不変）であることを示すものです：

```tsx
type MyProps = {
  readonly message: string;
};
type MyState = {
  readonly count: number;
};
```

`React.Component<P,S>`は既に不変としてマークされているので、`readOnly`は必要ありません。（[PR と議論](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/26813)）を参照してください。

</details>

**クラスメソッド**：通常通り実行されますが、関数の引数はすべて型付けされている必要があることを覚えておいてください。

```tsx
class App extends React.Component<{ message: string }, { count: number }> {
  state = { count: 0 };
  render() {
    return (
      <div onClick={() => this.increment(1)}>
        {this.props.message} {this.state.count}
      </div>
    );
  }
  increment = (amt: number) => {
    // like this
    this.setState((state) => ({
      count: state.count + amt,
    }));
  };
}
```

[View in the TypeScript Playground](https://www.typescriptlang.org/play/?jsx=2#code/JYWwDg9gTgLgBAJQKYEMDG8BmUIjgcilQ3wFgAoCtAGxQGc64BBMMOJADxiQDsATRsnQwAdAGFckHrxgAeAN5wQSBigDmSAFxw6MKMB5q4AXwA0cRWggBXHjG09rIAEZIoJgHwWKcHTBTccAC8FnBWtvZwAAwmANw+cET8bgAUAJTe5L6+RDDWUDxwKQnZcLJ8wABucBA8YtTAaADWQfLpwV4wABbAdCIGaETKdikAjGnGHiWlFt29ImA4YH3KqhrGsz19ugFIIuF2xtO+sgD0FZVTWdlp8ddH1wNDMsFFKCCRji5uGUFe8tNTqc4A0mkg4HM6NNISI6EgYABlfzcFI7QJ-IoA66lA6RNF7XFwADUcHeMGmxjStwSxjuxiAA)

**クラスプロパティ**：後で使用するためにクラスのプロパティを宣言する必要がある場合は、`state`のように代入しないで宣言してください。

```tsx
class App extends React.Component<{
  message: string;
}> {
  pointer: number; // like this
  componentDidMount() {
    this.pointer = 3;
  }
  render() {
    return (
      <div>
        {this.props.message} and {this.pointer}
      </div>
    );
  }
}
```

[View in the TypeScript Playground](https://www.typescriptlang.org/play/?jsx=2#code/JYWwDg9gTgLgBAJQKYEMDG8BmUIjgcilQ3wFgAoCtAGxQGc64BBMMOJADxiQDsATRsnQwAdAGFckHrxgAeAN4U4cEEgYoA5kgBccOjCjAeGgNwUAvgD44i8sshHuUXTwCuIAEZIoJuAHo-OGpgAGskOBgAC2A6JTg0SQhpHhgAEWA+AFkIVxSACgBKGzjlKJiRBxTvOABeOABmMzs4cziifm9C4ublIhhXKB44PJLlOFk+YAA3S1GxmzK6CpwwJdV1LXM4FH4F6KXKp1aesdk-SZnRgqblY-MgA)

何か追加したいなら[イシュー](https://github.com/typescript-cheatsheets/react-typescript-cheatsheet/issues/new)をください。
