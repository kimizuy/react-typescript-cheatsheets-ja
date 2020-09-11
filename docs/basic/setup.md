---
id: setup
title: セットアップ
---

## 前提条件

1. [React](https://reactjs.org) を深く理解していること。
2. [TypeScript の静的型づけ](https://www.typescriptlang.org/docs/handbook/basic-types.html) に馴れ親しんでいること（[2ality's guide](http://2ality.com/2018/04/type-notation-typescript.html) が参考になります。まったくの TypeScript ビギナーの場合は [chibicode’s tutorial](https://ts.chibicode.com/todo/) をチェックしてみてください）。
3. [React 公式ドキュメントの TypeScript セクション](https://reactjs.org/docs/static-type-checking.html#typescript) を読んでいること。
4. [TypeScript playground の React セクション](http://www.typescriptlang.org/play/index.html?jsx=2&esModuleInterop=true&e=181#example/typescript-with-react) を読んでいること（任意：[TypeScript playglound](http://www.typescriptlang.org/play/index.html) の Examples セクションにある 40 以上の example に目を通しましょう）。

このガイドでは、常に最新の TypeScript バージョンで動作することを前提としています。古いバージョンの注記は、折りたたみ可能な `<details>` タグ内に記述されています。

## React + TypeScript を始める

1. [Create React App v2.1+ with TypeScript](https://facebook.github.io/create-react-app/docs/adding-typescript): `npx create-react-app my-app --template typescript`

- 以前は `create-react-app-typescript` を推奨していましたが、現在は[廃止](https://www.reddit.com/r/reactjs/comments/a5919a/createreactapptypescript_has_been_archived_rip/)されています。[移行の手引き](https://vincenttunru.com/migrate-create-react-app-typescript-to-create-react-app/)をご覧ください。

2. [Basarat's guide](https://github.com/basarat/typescript-react/tree/master/01%20bootstrap) は自分の手で環境構築するガイドです（React + TypeScript + Webpack + Babel）。

- 特に `@types/react` と `@types/react-dom` がインストールされていることを確認してください（詳細は [DefinitelyTyped プロジェクト](https://definitelytyped.org/) へ）。
- たくさんの React + TypeScript ボイラープレートもあります。[おすすめの React + TypeScript 資料集](https://github.com/typescript-cheatsheets/react-typescript-cheatsheet#recommended-react--typescript-codebases-to-learn-from)をご覧ください。

## React をインポートする

```tsx
import * as React from "react";
import * as ReactDOM from "react-dom";
```

[TypeScript 2.7+](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-7.html) では、`--allowSyntheticDefaultImports` が実行できます（もしくは tsconfig に `"allowSyntheticDefaultImports": true` を追加します）。通常の jsx のようにインポートします：

```tsx
import React from "react";
import ReactDOM from "react-dom";
```

<details>

<summary>解説</summary>

なぜ `esModuleInterop` ではなく `allowSyntheticDefaultImports` なのでしょうか？ [Daniel Rosenwasser](https://twitter.com/drosenwasser/status/1003097042653073408) は webpack/parcel でより適していると言います。詳細な議論は <https://github.com/wmonk/create-react-app-typescript/issues/214> を確認してください。

それぞれのコンパイラフラグの公式な解説については [TypeScript の新しいドキュメント](https://www.typescriptlang.org/v2/en/tsconfig#allowSyntheticDefaultImports)もチェックしましょう！

</details>
