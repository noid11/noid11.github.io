---
title: "React 公式チュートリアルやったノート"
date: 2021-02-14T19:10:27+09:00
draft: false
toc: true
images:
tags: 
  - React
  - JavaScript
---

三目並べゲームを作成する [React 公式のチュートリアル](https://ja.reactjs.org/tutorial/tutorial.html)を進めたメモ

<!--more-->

## 振り返りノート

- [create-react-app](https://create-react-app.dev/) で React を使ったアプリケーションの土台が作れる
  - [TypeScript](https://create-react-app.dev/docs/adding-typescript/) も使用可能
- React では UI を [React.Component](https://ja.reactjs.org/docs/react-component.html) サブクラスというコンポーネントとして扱う
  - [props (properties)](https://ja.reactjs.org/docs/react-component.html#props)
    - React コンポーネントにデータを連携するもの
    - React では親コンポーネントから子コンポーネントに props を渡すことでアプリ内での情報をやりとりする
    - JavaScript のクラスでは、サブクラスのコンストラクタを定義する際には常に [`super()`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/super) を呼ぶ必要がある
      - [`constructor`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Classes/constructor) を持つ React コンポーネントでは、全てのコンストラクタを `super(props)` の呼び出しから始める
  - [render](https://ja.reactjs.org/docs/react-component.html#render)
    - React コンポーネントが描画するもの
    - JSX と呼ばれるシンタックスシュガーで記述可能
      - [React.createElement](https://ja.reactjs.org/docs/react-api.html#createelement) を使った定義を HTML ライクに書けるというもの
  - [state](https://ja.reactjs.org/docs/react-component.html#state)
    - React コンポーネントに持たせる状態を扱うもの
      - その React コンポーネント内でプライベートなデータとして扱う
    - state は対象 React コンポーネントのコンストラクタ以外では直接変更せず [`setState()`](https://ja.reactjs.org/docs/react-component.html#setstate) メソッドによって変更を行う
    - state の更新は非同期に行われる場合がある
      - `this.props`, `this.state` は非同期に更新されるため、これらの値に依存するような実装はしない
      - オブジェクトではなく関数を受け取る `setState()` を使うと、良い
        - 第1引数: 設定する state 
        - 第2引数: 更新が適用される時点での props
    - state の更新はマージされる
      - state に複数の値を使用している場合、複数の値を独立して更新することができる
        - ので state で取り扱う値を変更する場合、全ての値を変更せず、変更したい値だけを変更すれば良い
    - state のリフトアップ
      - state の管理を親 React コンポーネントに変更すること
      - 複数の子要素からデータを集めたい、または 2つ の子コンポーネントに互いにやりとりさせたい場合、代わりに親コンポーネント内で共有の state を宣言する必要がある
        - 親コンポーネントは props を使うことで、子に情報を返すことができる
        - これによって子コンポーネントが兄弟同士や親との間で、常に同期されるようになる
      - state を親コンポーネントにリフトアップすることは React コンポーネントのリファクタリングでよくあること
  - [関数コンポーネント](https://ja.reactjs.org/docs/components-and-props.html#function-and-class-components)
    - `render` メソッドだけを有して、自分の state を持たないコンポーネントを、よりシンプルに書くための方法
    - `React.Component` を継承するクラスを定義する代わりに props を入力として受け取り、表示すべき内容を返す関数を定義する
    - 関数コンポーネントはクラスよりも書くのが楽
    - `function Welcome(props) { return <h1>Hello, {props.name}</h1>; }`
  - [key](https://ja.reactjs.org/docs/lists-and-keys.html#keys)
    - React によって予約されているプロパティ
    - リストアイテムをレンダリングする際に、各アイテムを識別するために指定する ID のようなもの
      - `<li key={user.id}>{user.name}: {user.taskCount} tasks left</li>`
    - 動的なリストを構築する場合には、正しい key を割り当てることが強く推奨されている
      - key はグローバルに一意である必要は無い 
      - コンポーネントと、その兄弟の間で一意であれば十分
- React Developer Tools
  - React 開発者向けのツールが [Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en) と [Firefox](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/) 向けに公開されている
  - React を使った開発をする際にはインストールしておこう
- immutability の重要性
  - 直接データを mutate する
    - 配列データを直接変更したり
  - imutability を保つ
    - 配列データを直接変更するのではなく、配列データのコピーを作って操作したり
      - [slice](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/slice), [concat](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/concat), [map](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/map) とか、新しい配列を返すような関数を使う
    - メリット
      1. 複雑な機能が簡単に実装できる
      1. 変更があったことを検出できる
        - immutable object を比較すれば良い
      1. React の再レンダータイミングの決定
        - [pure component](https://ja.reactjs.org/docs/react-api.html#reactpurecomponent) を構築しやすくなる
        - イミュータブルなデータは変更があったかどうか簡単に分かるため、コンポーネントをいつ再レンダーすべきなのか決定しやすくなる
        - [shouldComponentUpdate() および pure component をどのように作成するのかについて](https://ja.reactjs.org/docs/optimizing-performance.html#examples)

## ローカル環境で開発する準備

### create-react-app

```zsh
npx create-react-app react-tutorial
```

- [create-react-app](https://create-react-app.dev): React 使ったモダンなアプリケーション開発環境をセットアップしてくれるツール
  - [ディレクトリ構成](https://create-react-app.dev/docs/folder-structure)
  - [TypeScript を使いたい場合](https://create-react-app.dev/docs/adding-typescript): 
    - `npx create-react-app my-app --template typescript`
    - `npm install --save typescript @types/node @types/react @types/react-dom @types/jest`
    - [React TypeScript Cheatsheets](https://react-typescript-cheatsheet.netlify.app/)


### src ディレクトリの中身を削除

```zsh
cd react-tutorial/src
rm -f *
cd ../
```


### 基本となるファイルを作成

```zsh
touch index.css
touch index.js
```

index.css

```css
body {
    font: 14px "Century Gothic", Futura, sans-serif;
    margin: 20px;
}

ol, ul {
    padding-left: 30px;
}

.board-row:after {
    clear: both;
    content: "";
    display: table;
}

.status {
    margin-bottom: 10px;
}

.square {
    background: #fff;
    border: 1px solid #999;
    float: left;
    font-size: 24px;
    font-weight: bold;
    line-height: 34px;
    height: 34px;
    margin-right: -1px;
    margin-top: -1px;
    padding: 0;
    text-align: center;
    width: 34px;
}

.square:focus {
    outline: none;
}

.kbd-navigation .square:focus {
    background: #ddd;
}

.game {
    display: flex;
    flex-direction: row;
}

.game-info {
    margin-left: 20px;
}
```

index.js

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';

class Square extends React.Component {
  render() {
    return (
      <button className="square">
        {/* TODO */}
      </button>
    );
  }
}

class Board extends React.Component {
  renderSquare(i) {
    return <Square />;
  }

  render() {
    const status = 'Next player: X';

    return (
      <div>
        <div className="status">{status}</div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    );
  }
}

class Game extends React.Component {
  render() {
    return (
      <div className="game">
        <div className="game-board">
          <Board />
        </div>
        <div className="game-info">
          <div>{/* status */}</div>
          <ol>{/* TODO */}</ol>
        </div>
      </div>
    );
  }
}

// ========================================

ReactDOM.render(
  <Game />,
  document.getElementById('root')
);
```


### 動作確認

```zsh
npm start
```


## React とは？

- React はユーザインターフェイスを構築するための、宣言型で効率的で柔軟な JavaScript ライブラリ
- 複雑な UI を、「コンポーネント」と呼ばれる小さく独立した部品から組み立てることができる
- まずは React.Component サブクラスというコンポーネントをみていく

```js
class ShoppingList extends React.Component {
  render() {
    return (
      <div className="shopping-list">
        <h1>Shopping List for {this.props.name}</h1>
        <ul>
          <li>Instagram</li>
          <li>WhatsApp</li>
          <li>Oculus</li>
        </ul>
      </div>
    );
  }
}
```

- コンポーネントは、 React に何を描画するのか伝えるもの
  - データが変更されると React はコンポーネントを効率よく更新して再度レンダリングを行う
- `ShoppingList` は React コンポーネントクラスまたは React コンポーネント型
  - コンポーネントは props (properties の略) を呼ばれるパラメーターを受け取り、 `render` メソッドを通して、表示するビューの階層構造を返す
- `render` メソッドが返すのは画面上に表示するものの説明書き
  - React は説明書きを受け取って画面に描画する
    - 具体的には、 `render` は、描画すべきものの軽量な記述形式である React 要素というものを返す
    - たいていの React 開発者は、これらの構造を簡単に記述できる [JSX](https://ja.reactjs.org/docs/jsx-in-depth.html) と呼ばれる構文を使用する
      - `<div />` という構文は、ビルド時に `React.createElement('div')` に変換される
      - https://ja.reactjs.org/docs/react-api.html#createelement
  - 上記の例は以下のコードと同等

```js
return React.createElement('div', {className: 'shopping-list'},
  React.createElement('h1', /* ... h1 children ... */),
  React.createElement('ul', /* ... ul children ... */)
);
```

- JSX では JavaScript のすべての能力を使うことができる
  - どのような JavaScript の式も JSX 内で中括弧に囲んで記入することができる
  - 各 React 要素は、変数に格納したりプログラム内で受け渡ししたりできる JavaScript のオブジェクト
- 上記の ShoppingList コンポーネントは `<div />` や `<li />` といった組み込みの DOM コンポーネントのみをレンダーしているが、自分で書いたカスタム React コンポーネントを組み合わせることも可能
  - 例えば `<ShoppingList />` と書いてショッピングリスト全体を指し示すことができる
  - それぞれの React のコンポーネントはカプセル化されており、独立して動作する
  - これにより単純なコンポーネントから複雑な UI を作成することができる


## スターターコードの中身を確認

- `src/index.js` には 3つ の React コンポーネントがある
  1. Square: 正方形のマス `<button>`
  2. Borad: 盤面 (3x3 の Square)
  3. Game: Board とゲーム情報


## データを Props 経由で渡す

- Board の `renderSquare` メソッド内で、 props として value という名前の値を Square に渡すようにコードを変更

```js
class Boards extends React.Component {
  renderSquare(i) {
    return <Square value={i}>
  }
}
```

- Square の `render` メソッドで、渡された値を表示するように、`{/* TODO */}` を `{this.props.value}` に書き換える

```js
class Square extends React.Component {
  render() {
    return (
      <button className="square">
        {this.props.value}
      </button>
    );
  }
}
```

- これで親である Board コンポーネントから、子である Square コンポーネントに「props を渡す」ことができた
- React では、親から子へと props を渡すことで、アプリ内を情報が流れていく

[この時点でのコード](https://codepen.io/gaearon/pen/aWWQOG?editors=0010)


## インタラクティブなコンポーネントを作る 

- Square コンポーネントがクリックされたら X と表示するように実装する
  - アロー関数を使う場合 onClick プロパティに渡しているのは関数であることに注意
    - React はクリックされるまでこの関数を実行しない
  - `onClick={alert('click')}` のように書いてしまうとコンポーネントをレンダリングしなおす度に関数が呼ばれてしまう

```js
class Square extends React.Component {
  render() {
    return (
      <button className="square" onClick={function() { alert('click') }}>
      <!-- アロー関数を使う場合: onClick={() => alert('click')} -->

        {this.props.value}
      </button>
    );
  }
}
```

- これで Square をクリックした際にアラートが表示されるはず
- コンポーネントに状態を持たせたい場合、 state を使う
  - クリックされたら X にする・・・というのが今回持たせたい状態
  - https://ja.reactjs.org/docs/state-and-lifecycle.html
    1. state はコンストラクタ以外では直接変更せず `setState` を使う
      - https://ja.reactjs.org/docs/react-component.html#setstate
    2. state の更新は非同期に行われる可能性がある
      - React はパフォーマンスのために複数の `setState` 呼び出しを一度の更新にまとめて処理する場合がある
      - `this.props` と `this.state` は非同期に更新されるため state を求める際にはこれらの値に依存するべきではない
        - オブジェクトではなく関数を受け取る `setState` を使うと、 state を最初の引数として受け取り、更新が適用される時点での props を第2引数として受け取る
    3. state の更新はマージされる
      - `setState` を呼び出した場合 React は与えられたオブジェクトを現在の state にマージする
      - state に複数の値を使用している場合、複数の値を独立して更新することができる
- React コンポーネントはコンストラクタで `this.state` を設定することで、状態を持つことができるようになる
  - `this.state` は、それが定義されているコンポーネント内でプライベートと見なすべきもの
  - 現在の Square の状態を `this.state` に保存して、マス目がクリックされた時にそれを変更するようにする
- JavaScript のクラスでは、サブクラスのコンストラクタを定義する際には常に `super` を呼ぶ必要がある
  - `constructor` を持つ React コンポーネントでは、全てのコンストラクタを `super(props)` の呼び出しから始めるべき

クラスにコンストラクタを追加して state を初期化

```js
class Square extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: null,
    };
  }

  render() {
    return(
      <button className="square" onClick={() => alert('click')}>
        {this.props.value}
      </button>
    );
  }
}
```

クリックされたら state が変わるようにする

```js
class Square extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: null,
    }
  }

  render() {
    render (
      <button
        className="square"
        onClick={() => this.setState({value: 'X'})}
      >
        {this.state.value}
      </button>
    );
  }
}
```

- Square の `render` メソッド内に書かれた `onClick` ハンドラ内で `this.setState` を呼び出すことで、 React に `<button>` がクリックされたら常に再レンダーするように伝えることができる
- データ更新のあと、この Square の `this.state.value` は `X` になるので、盤面に X と表示される
- `setState` をコンポーネント内で呼び出すと React は内部の子コンポーネントも自動的に更新する

[この時点でのコード](https://codepen.io/gaearon/pen/VbbVLg?editors=0010)


## React Developer Tools

- React 開発者向けのツールが [Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en) と [Firefox](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/) 向けに公開されている
- ブラウザの開発者向けツールを拡張するもので、 React コンポーネントのツリーを確認することができる


## ゲームを完成させる

- ここまでの作業で三目並べゲームの基本的な部品が揃った
- 完全なゲームにするためのタスクは以下
  - 盤面に X と O を交互に置けるようにする
  - どちらのプレイヤーが勝利したのか判定できるようにする


### State のリフトアップ

- 現時点では、それぞれの Square コンポーネントがゲームの状態を保持している
  - どちらのプレイヤーが勝利したのか判定するために 9個 のマス目の値を 1箇所 で管理するようにする
    - Board が各 Square に、現時点の state がどうなっているのか問い合わせればよいだけ？
      - 可能ではあるが、コードが難解になり、メンテしづらくなるので非推奨
    - ゲームの状態を各 Square の代わりに、親となる Board コンポーネントで管理するのがベストな解決策
      - Board コンポーネントは、それぞれの Square に props を渡すことで、何を表示すべきかを伝えられる
        - Square に番号を常時させた時と同じ
- 複数の子要素からデータを集めたい、または 2つ の子コンポーネントに互いにやりとりさせたい場合、代わりに親コンポーネント内で共有の state を宣言する必要がある
  - 親コンポーネントは props を使うことで、子に情報を返すことができる
  - これによって子コンポーネントが兄弟同士や親との間で、常に同期されるようになる
- state を親コンポーネントにリフトアップすることは React コンポーネントのリファクタリングでよくあること

Board にコンストラクタを追加し、初期 state として 9個 のマス目に対応する 9個 の null 値をセットする

```js
class Board extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            squares: Array(9).fill(null),
        };
    }

    renderSquare(i) {
        return <Square value={i} />;
    }
...
```

- `renderSquare` が `i` を使ってマス目を表示していたが、 X マークを表示するようにしたので、 Board から渡されている `value` プロパティの値は、現状無視されている
- props を渡すメカニズムを使う
  - Board を書き換えて、それぞれの個別の Square に現在の値 (X, O, null) を伝えるようにする
  - `squares` 配列は Board コンストラクタで定義しているので Board の `renderSquare` が値を読み込むように書き換える
  - `fill` メソッド
    - https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/fill
    - 配列に対して値を設定するメソッド
    - `Array(9).fill(null)` は 9つの要素 を持つ配列を作成して、それぞれに null を設定するという意味

[この時点でのコード](https://codepen.io/gaearon/pen/gWWQPY?editors=0010)

- マス目がクリックされたときの挙動を変更する
  - 現状、どのマス目に何が入っているのかを管理しているのは Board
  - Square が Board の state を更新できるようにする必要がある
  - state は、それを定義しているコンポーネント内でプライベートなものなので、 Square から Board の state を直接書き換えることはできない
  - Board から Square に関数を渡すことにして、マス目がクリックされた時に Square にその関数を呼んでもらうようにする

```js
    renderSquare(i) {
        return (
            <Square 
                value={this.state.squares[i]}
                onClick={() => this.handleClick(i)}
            />
        );
    }
```

- 現状 Board から Square には props として 2つ の値を渡している
  1. value
  2. onClick
    - マス目がクリックされた時に Square が呼び出すためのもの
    - 以下のような実装をしていく
      - Square の `render` メソッド内の `this.state.value` を `this.props.value` に書き換える
      - Square の `render` メソッド内の `this.setState()` を `this.props.onClick()` に書き換える
      - Square は、ゲームの状態を管理しないので `constuctor` を削除

```js
class Square extends React.Component {
    render() {
        return (
            <button 
                className="square" 
                onClick={() => this.props.onClick()}
            >
                {this.props.value}
            </button>
        );
    }
}
```

- Square がクリックされると Board から渡された `onClick` 関数がコールされる
- 流れのおさらい
  1. 組み込みの DOM コンポーネントである `<button>` に `onClick` プロパティが設定されているため、 React がクリックに対するイベントリスナーを設定
    - DOM 要素である `<button>` は組み込みコンポーネントなので `onClick` 属性は React にとって特別な意味を持っている
    - Square のようなカスタムコンポーネントでは、名前の付け方は開発者が自由に決めれる
    - Square の `onClick` プロパティや Board の `handleClick` メソッドについては別名を付けたとしても同じように動作する
    - React の慣習
      - イベントを表す props には `on[Event]` という名前
      - イベントを処理するメソッドには `handle[Event]` という名前
  2. ボタンがクリックされると React は Square の `render()` メソッド内に定義されている `onClick` イベントハンドラをコール
  3. このイベントハンドラが `this.props.onClick()` をコール
    - Square の `onClick` プロパティは Board から渡されているもの
  4. Board は Square に `onClick={() => this.handleClick(i)}` を渡していたので Square はクリックされた時に `this.handleClick(i)` を呼び出す
  5. まだ `handleClick()` を定義していないので、コードがクラッシュする
    - Square をクリックすると "this.handleClick is not a function" といったエラー画面が表示されるハズ

`handleClick` を Board クラスに実装する

```js
class Board extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            squares: Array(9).fill(null),
        };
    }

    handleClick(i) {
        const squares = this.state.squares.slice();
        squares[i] = 'X';
        this.setState({ squares: squares });
    }

    renderSquare(i) {
        return (
            <Square 
                value={this.state.squares[i]}
                onClick={() => this.handleClick(i)}
            />
        );
    }
...
```

[この時点でのコード](https://codepen.io/gaearon/pen/ybbQJX?editors=0010)

- 再びマス目をクリックすることで値が書き込めるようになった
  - 状態は個々の Square コンポーネントではなく Board コンポーネント内に保存されている
  - Board の state が変更になると、個々の Square コンポーネントも自動的に再レンダーされる
  - 全てのマス目の状態を Board コンポーネント内で保持するようにしたことで、この後で勝利者を判定する処理をシンプルに実装できる
- Square コンポーネントは自分で state を管理しないので Board コンポーネントから値を受け取って、クリックされた時にはクリックされた事を Board コンポーネントに伝えるだけの存在になった
  - React 用語で表現すると Square コンポーネントは 制御されたコンポーネント (controlled component) になった
- `handleClick` 内では `squares` を直接変更する代わりに `slice()` を呼び出して配列のコピーを作成している
  - https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/slice


### イミュータビリティは何故重要？

- `slice()` メソッドで `squares` 配列のコピーを作成し、それを変更する実装とした
  - immutability と重要性について解説
- 一般的に、変化するデータに対しては 2種類 のアプローチがある
  1. データの値を直接操作する mutate
  2. 望む変更を加えた新しいデータのコピーで古いデータを置き換える
- 最終的な結果は同じだが、直接データの mutate をしないことで、以下のメリットが有る
  - 複雑な機能が簡単に実装できる
    - ヒストリを保って再利用するとか
  - 変更の検出
    - 参照しているイミュータブルオブジェクトが前と別のものであれば変更があったと判定できる
  - React の再レンダータイミングの決定
    - React で pure component を構築しやすくなる
    - イミュータブルなデータは変更があったかどうか簡単に分かるため、コンポーネントをいつ再レンダーすべきなのか決定しやすくなる
    - [shouldComponentUpdate() および pure component をどのように作成するのかについて](https://ja.reactjs.org/docs/optimizing-performance.html#examples)


### 関数コンポーネント

- Square を関数コンポーネントに書き換える
- 関数コンポーネント
  - `render` メソッドだけを有して自分の state を持たないコンポーネントを、よりシンプルに書くための方法
  - `React.Component` を継承するクラスを定義する代わりに `props` を入力として受け取り、表示すべき内容を返す関数を定義する
- 関数コンポーネントはクラスよりも書くのが楽
  - 多くのコンポーネントは関数コンポーネントで書くことができる
- `onClick={() => this.props.onClick()}` をより短い `onClick={props.onClick}` に書き換える

```js
function Square(props) {
    return (
        <button className="square" onClick={props.onClick}>
            {props.value}
        </button>
    );
}
```

[この時点でのコード](https://codepen.io/gaearon/pen/QvvJOv?editors=0010)


### 手番の処理

- クリックでの書き換えを交互に X と O で切り替えるよう実装していく
- デフォルトで先手を X にする
  - Board のコンストラクタで state の初期値を変えれば OK

```js
class Board extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            squares: Array(9).fill(null),
            xIsNext: true,
        };
    }
...
```

- プレイヤーが着手する度に、どちらのプレイヤーの手番なのか決める `xIsNext` が反転され、ゲームの状態が保存されるようにする
- Board の `handleClick` 関数を書き換えて `xIsNext` の値を反転させるようにする

```js
    handleClick(i) {
        const squares = this.state.squares.slice();
        squares[i] = this.state.xIsNext ? 'X' : 'O';
        this.setState({
            squares: squares,
            xIsNext: !this.state.xIsNext,
        });
    }
```

- この変更により X と O が交互に着手できるようになる
- Board の `render` 内にある status も変更して、どちらのプレイヤーの手番なのか表示するように修正する

```js
    render() {
        const status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
...
```

[この時点でのコード](https://codepen.io/gaearon/pen/KmmrBy?editors=0010)


### ゲーム勝者の判定

- ゲームが決着して次の手番がなくなった時に、それを表示する
- ヘルパー関数を追加

```js
function calculateWinner(squares) {
    const lines = [
        [0, 1, 2],
        [3, 4, 5],
        [6, 7, 8],
        [0, 3, 6],
        [1, 4, 7],
        [2, 5, 8],
        [0, 4, 8],
        [2, 4, 6],
    ];

    for (let i = 0; i < lines.length; i++) {
        const [a, b, c] = lines[i];
        if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
            return squares[a];
        }
    }

    return null;
}
```

- `calculateWinner` は 9つ の square 配列が与えられると、勝者を判断し X or O or null を返す
- Board の `render` 関数内で `calculateWinner(squares)` を呼び出して、どのプレイヤーが勝利したか判定する
  - 決着が着いた場合は `Winner: X` or `Winner: O` のようなテキストを表示する
  - Borad の `render` 関数の `status` 宣言を書き換える

```js
    render() {
        const winner = calculateWinner(this.state.squares);
        let status;
        if (winner) {
            status = 'Winner: ' + winner;
        } else {
            status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
        }
...
```

- Board の `handleClick` を書き換えて、ゲームの決着が既についている場合や、クリックされたマス目が既に埋まっている場合には早期に return するようにする

```js
    handleClick(i) {
        const squares = this.state.squares.slice();
        if (calculateWinner(squares) || squares[i]) {
            return;
        }
        squares[i] = this.state.xIsNext ? 'X' : 'O';
        this.setState({
            squares: squares,
            xIsNext: !this.state.xIsNext,
        });
    }
```

[この時点でのコード](https://codepen.io/gaearon/pen/LyyXgK?editors=0010)


## タイムトラベル機能の追加

- 以前の着手まで時間を巻き戻すことができるようにする


### 着手履歴の保存

- `squares` 配列をミューテートしていたら、タイムトラベルの実装は困難
- しかし、着手がある度に `squares` のコピーを作り、この配列をイミュータブルなものとして扱う実装をしてきた
  - `squares` の過去バージョンを全て保存しておいて、過去の手番を遡ることができるようになる
- 過去の `squares` 配列を `history` という名前で別配列に保存する
  - この `history` 配列は初手から最後までの盤面の全ての状態を表現
  - 以下のような構造

```js
history = [
    // Before first move
    {
        squares: [
            null, null, null,
            null, null, null,
            null, null, null,
        ]
    },
    // After first move
    {
        squares: [
            null, null, null,
            null, 'X', null,
            null, null, null,
        ]
    },
    // After second move
    {
        squares: [
            null, null, null,
            null, 'X', null,
            null, null, 'O',
        ]
    },
    // ...
]
```

- この `history` の状態は、どのコンポーネントが保持するべきか？


### State のリフトアップ、再び

- トップレベルの Game コンポーネント内で、過去の着手履歴を表示する
  - そのため Game コンポーネントが `history` にアクセスできる必要があるので `history` という state は Game コンポーネントに置く
- `history` state を Game コンポーネント内に置くことで `squares` の state を、子である Board コンポーネントから取り除くことができる
  - Square コンポーネントにあった state をリフトアップして Board コンポーネントに移動した時と同様に、今度は Board にある state を Game コンポーネントにリフトアップする
  - これにより Game コンポーネントは Board のデータを完全に制御することになり `history` 内の過去の手番データを Board にレンダーさせることができるようになる
- Game コンポーネントの初期 state をコンストラクタ内でセットする

```js
class Game extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            history: [{
                squares: Array(9).fill(null),
            }],
            xIsNext: true,
        };
    }
...
```

- 次に Board コンポーネントが `squares` と `onClick` プロパティを Game コンポーネントから受け取るようにする
  - Board 内には多数のマス目に対応するクリックハンドラが 1つ だけあるので Square の位置を `onClick` ハンドラに渡して、どのマス目がクリックされたのかを伝えるようにする
  - 以下の手順で Board コンポーネントを書き換える
    1. Board の `constructor` を削除
    2. Board の `renderSquare` にある `this.state.squares[i]` を `this.props.squares[i]` に置き換える
    3. Board の `renderSquare` にある `this.handleClick(i)` を `this.props.onClick(i)` に置き換える
- Board コンポーネントは以下のようになる

```js
class Board extends React.Component {
    handleClick(i) {
        const squares = this.state.squares.slice();
        if (calculateWinner(squares) || squares[i]) {
            return;
        }
        squares[i] = this.state.xIsNext ? 'X' : 'O';
        this.setState({
            squares: squares,
            xIsNext: !this.state.xIsNext,
        });
    }

    renderSquare(i) {
        return (
            <Square 
                value={this.props.squares[i]}
                onClick={() => this.props.onClick(i)}
            />
        );
    }
...
```

- Game コンポーネントの render 関数を更新して、ゲームのステータステキストの決定や、表示の際に最新の履歴が使われるようにする

```js
class Game extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            history: [{
                squares: Array(9).fill(null),
            }],
            xIsNext: true,
        };
    }

    render() {
        const history = this.state.history;
        const current = history[history.length - 1];
        const winner = calculateWinner(current.squares);
        let status;
        if (winner) {
            status = 'Winner: ' + winner;
        } else {
            status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
        }

        return (
            <div className="game">
                <div className="game-board">
                    <Board
                        squares={current.squares}
                        onClick={(i) => this.handleClick(i)}
                    />
                </div>
            <div className="game-info">
                <div>{status}</div>
                <ol>{/* TODO */}</ol>
            </div>
            </div>
        );
    }
}
```

- Game コンポーネントがゲームのステータステキストを表示するようになったので、対応するコードは Board 内 `render` メソッドから削除できる
- このリファクタリングの後で、 Board の `render` メソッドは以下のようになる

```js
    render() {
        return (
            <div>
                <div className="board-row">
                    {this.renderSquare(0)}
                    {this.renderSquare(1)}
                    {this.renderSquare(2)}
                </div>
                <div className="board-row">
                    {this.renderSquare(3)}
                    {this.renderSquare(4)}
                    {this.renderSquare(5)}
                </div>
                <div className="board-row">
                    {this.renderSquare(6)}
                    {this.renderSquare(7)}
                    {this.renderSquare(8)}
                </div>
            </div>
        );
    }
```

- 最後に `handleClick` メソッドを Board コンポーネントから Game コンポーネントに移動する
  - Game コンポーネントの state は異なる形で構成されているので、 `handleClick` の中身も修正する
  - Game 内の `handleClick` メソッドで新しい履歴エントリを `history` に追加する 

```js
    handleClick(i) {
        const history = this.state.history;
        const current = history[history.length - 1];
        const squares = current.squares.slice();

        if (calculateWinner(squares) || squares[i]) {
            return;
        }

        squares[i] = this.state.xIsNext ? 'X' : 'O';

        this.setState({
            history: history.concat([{
                squares: squares,
            }]),
            xIsNext: !this.state.xIsNext,
        });
    }
```

- `push()` とは異なり `concat()` は元の配列をミューテートしないため、こちらを利用する
  - `Array.prototype.concat()`
    - https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/concat
    - 2つ 以上の配列を結合して、新しい配列を返すメソッド
  - `Array.prototype.push()`
    - https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/push
    - 配列の末尾に 1つ 以上の要素を追加し、新しい配列の要素数を返すメソッド
- Board コンポーネントに必要なのは `renderSquare` と `render` メソッドだけ
  - ゲームの状態と `handleClick` は Game コンポーネントで管理

[この時点でのコード](https://codepen.io/gaearon/pen/EmmOqJ?editors=0010)


### 過去の着手を表示

- 三目並べの履歴を記録しているので、これを過去の着手リストとしてプレイヤーに表示することが可能
- React 要素は第一級 JavaScript オブジェクトであり、それらをアプリケーション内で受け渡しできる
  - React で複数の要素を描画するには React 要素の配列を使うことができる
- JavaScript では配列に `map()` メソッドが存在しており、これはデータを別のデータにマップするのによく使われる
  - https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/map
  - 与えられた関数を配列の全要素に対して呼び出して、その結果となる新しい配列を生成する

```js
const numbers = [1, 2, 3];
const doubled = numbers.map(x => x * 2); // [2, 4, 6]
```

- `map()` を使うことで、着手履歴の配列をマップして画面上のボタンを表現する React 要素を作り出し、過去の手番にジャンプするためのボタンの一覧を表示できる
- Game の `render` メソッド内で `history` に `map()` を使う

```js
    render() {
        const history = this.state.history;
        const current = history[history.length - 1];
        const winner = calculateWinner(current.squares);

        const moves = history.map((step, move) => {
            const desc = move ?
                'Go to move #' + move :
                'Go to game start';
            return (
                <li>
                    <button onClick={() => this.jumpTo(move)}>{desc}</button>
                </li>
            );
        });

        let status;
        if (winner) {
            status = 'Winner: ' + winner;
        } else {
            status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
        }

        return (
            <div className="game">
                <div className="game-board">
                    <Board
                        squares={current.squares}
                        onClick={(i) => this.handleClick(i)}
                    />
                </div>
            <div className="game-info">
                <div>{status}</div>
                <ol>{moves}</ol>
            </div>
            </div>
        );
    }
```

[この時点でのコード](https://codepen.io/gaearon/pen/EmmGEa?editors=0010)

- ゲーム履歴内にある三目並べの、それぞれの着手に対応して、ボタンを有するリストアイテムを作成
  - ボタンには `onClick` ハンドラがあり、それは `this.jumpTo()` メソッドを呼び出す
  - `jumpTo()` は未だ実装していない
- ひとまず、このコードにより、ゲーム内で行われた着手のリストが表示されるようになったが、コンソールに以下の警告が表示されるようになった

```
Warning: Each child in an array or iterator should have a unique “key” prop. Check the render method of “Game”.
```

- この警告について対策していく


### key を選ぶ

- リストをレンダーする際、リストの項目それぞれについて、 React はとある情報を保持する
  - リストが変更になった場合 React は、どのアイテムが変更になったのかを知る必要がある
  - リストのアイテムは追加された可能性も、削除された可能性も、並び替えられた可能性も、中身自体が変更になった可能性もある

以下のリストが

```html
<li>Alexa: 7 tasks left</li>
<li>Ben: 5 tasks left</li>
```

以下のリストに遷移する場合を考える

```html
<li>Ben: 9 tasks left</li>
<li>Claudia: 8 tasks left</li>
<li>Alexa: 5 tasks left</li>
```

- タスクの数も変化しているが、人間が見た場合 Alexa と Ben の順番が変わって、その間に Claudia が追加されていると考える
- React はリストの項目それぞれに対して key プロパティを与えることで、兄弟要素の中で、そのアイテムが区別できるようにする必要がある
- このケースでは `alexa`, `ben`, `claudia` の文字列を使う方法がある
  - データベースからのデータを表示している場合は、データベースで使用しているそれぞれの項目の ID を Key として使うこともできる

```js
<li key={user.id}>{user.name}: {user.taskCount} tasks left</li>
```

- リストが再レンダーされる際 React はそれぞれのリスト項目の key について、前回のリスト項目内に同じ key を持つものがないか探す
  - もし、以前になかった key がリストに含まれていれば React はコンポーネントを作成する
  - もし、以前のリストにあった key が新しいリストに含まれていなければ React はコンポーネントを破棄する
  - もし、 2つ の key がマッチした場合、対応するコンポーネントは移動される
- key はそれぞれのコンポーネントの同一性に関する情報を React に与え、それにより React は再レンダー間で state を保持できるようになる
  - もしコンポーネントの key が変化していれば、コンポーネントは破棄されて新しい state で再作成される
- `key` は特別なプロパティであり React によって予約されている
  - より応用的な機能である `ref` も同様
  - 要素が作成される際 React は `key` プロパティを引き抜いて、返される要素に直接その `key` を格納する
  - `key` は `props` の一部のようにも思えるが `this.props.key` で参照できない
  - React は、どの子要素を更新すべきかを決定する際に `key` を自動的に使用する
  - コンポーネントが自身の `key` について確認する方法はない
- 動的なリストを構築する場合は、正しい key を割り当てることが強く推奨される
  - 適切な key が無い場合、データ構造を再構成して、そのような key が存在するようにするべきかも
- key が指定されなかった場合 React は警告を表示し、デフォルトで key として配列のインデックスを使用する
  - 配列のインデックスを key として使うことは、項目を並べ替えたり挿入・削除する際に問題の原因となる
  - 明示的に `key={i}` と渡すことで警告を消すことはできるが、配列のインデックスを使う場合と同様な問題が生じるため、殆どの場合は非推奨
- key はグローバルに一意である必要は無い
  - コンポーネントと、その兄弟の間で一意であれば十分


### タイムトラベルの実装

- 三目並べゲームの履歴内においては、全ての着手には、それに関連付けられた一意な ID が存在する
  - 着手順の連番数字
- 着手はゲームの最中に並びがかわったり、削除されたり、挿入されたりすることは無いので、着手のインデックスを key として使うのは安全
- Game コンポーネントの `render` メソッド内で key は `<li key={move}>` のようにして加えることができ、これで React の key に関する警告は表示されなくなる

```js
        const moves = history.map((step, move) => {
            const desc = move ?
                'Go to move #' + move :
                'Go to game start';
            return (
                <li key={move}>
                    <button onClick={() => this.jumpTo(move)}>{desc}</button>
                </li>
            );
        });
```

[この時点でのコード](https://codepen.io/gaearon/pen/PmmXRE?editors=0010)

- まだ `jumpTo` メソッドが未定義なので、このリスト項目内のボタンをクリックするとエラーが発生する
  - `jumpTo` を実装する前に Game コンポーネントの state に stepNumber という値を加える
  - これは、いま何手目の状態を見ているのかを表すのに使う
- Game の `constructor` 内で state の初期値として `stepNumber: 0` を加える

```js
class Game extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            history: [{
                squares: Array(9).fill(null),
            }],
            stepNumber: 0,
            xIsNext: true,
        };
    }
...
```

- 次に Game 内に `jumpTo` メソッドを定義してその `stepNumber` が更新されるようにする
- また、更新しようとしている `stepNumber` の値が偶数だった場合は `xIsNext` を true に設定する

```js
    jumpTo(step) {
        this.setState({
            stepNumber: step,
            xIsNext: (step % 2) === 0,
        });
    }
```

- マス目をクリックしたときに実行される Game の `handleClick` メソッドに、いくつか変更を加える
- 今加えた state である `stepNumber` は、現在ユーザーに見せている着手を反映している
  - 新しい着手が発生した場合は `this.setState` の引数の一部として `stepNumber: history.length` を加えることで `stepNumber` を更新する必要がある
- `this.state.history` から読み取っているところを `this.state.history.slice(0, this.state.stepNumber + 1)` に書き換える
  - これにより、時間の巻き戻しをしてから、その時点で新しい着手を起こした場合に、そこから見て将来にある履歴を確実に捨て去ることができる

```js
    handleClick(i) {
        const history = this.state.history.slice(0, this.state.stepNumber + 1);
        const current = history[history.length - 1];
        const squares = current.squares.slice();

        if (calculateWinner(squares) || squares[i]) {
            return;
        }

        squares[i] = this.state.xIsNext ? 'X' : 'O';

        this.setState({
            history: history.concat([{
                squares: squares,
            }]),
            stepNumber: history.length,
            xIsNext: !this.state.xIsNext,
        });
    }
```

- Game コンポーネントの `render` を書き換えて、常に最後の着手後の状態をレンダーするのではなく、 `stepNumber` によって現在選択されている着手をレンダーするようにする

```js
    render() {
        const history = this.state.history;
        const current = history[this.state.stepNumber];
        const winner = calculateWinner(current.squares);

        const moves = history.map((step, move) => {
            const desc = move ?
                'Go to move #' + move :
                'Go to game start';
            return (
                <li key={move}>
                    <button onClick={() => this.jumpTo(move)}>{desc}</button>
                </li>
            );
        });
```

- ゲーム内履歴の、どの手番をクリックした場合でも、三目並べの盤面は、該当着手が発生した直後の状態を表示するよう更新される

[この時点でのコード](https://codepen.io/gaearon/pen/gWWZgR?editors=0010)


## まとめ

- 以下のような機能を有する三目並べゲームが完成した
  1. 三目並べが遊べる
  1. 決着が着いた時に表示ができる
  1. ゲーム進行に合わせて履歴が保存される
  1. 着手の履歴の見直しや、顔面の以前の状態が参照できる

[最終結果のコード](https://codepen.io/gaearon/pen/gWWZgR?editors=0010)

- 改良アイディア
  1. 履歴内のそれぞれの着手位置を (col, row) というフォーマットで表示する
  1. 着手履歴のリスト中で、現在選択されているアイテムをボールドにする
  1. Board でマス目を並べる部分を、ハードコーディングではなく 2つ のループを使用するように書き換える
  1. 着手履歴リストを昇順・降順いずれでも並び替えられるよう、トグルボタンを追加する
  1. どちらかが勝利した際に、勝利に繋がった 3つ のマス目をハイライトする
  1. どちらも勝利しなかった場合、結果が引き分けになったというメッセージを表示する
- このチュートリアルを通して 要素, コンポーネント, props, state といった React の概念に触れた
  - これらのトピックについて更に掘り下げた説明はドキュメントの続きを参照
    - https://ja.reactjs.org/docs/hello-world.html
  - コンポーネントの作成方法について、より詳細に学ぶには React.Component API リファレンスを参照
    - https://ja.reactjs.org/docs/react-component.html


## この記事を試した環境

```zsh
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H512
% node -v
v12.16.1
% npm -v
6.14.11
% npx -v
6.14.11
```

package.json

```json
{
  "name": "react-tutorial",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^5.11.9",
    "@testing-library/react": "^11.2.5",
    "@testing-library/user-event": "^12.7.0",
    "react": "^17.0.1",
    "react-dom": "^17.0.1",
    "react-scripts": "4.0.2",
    "web-vitals": "^1.1.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```