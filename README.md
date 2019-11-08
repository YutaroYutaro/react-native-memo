# react-native-memo

# expo
`brew`, `node`, `npm`, `yarn`は入れておく
```
$ brew install watchman
$ yarn global add expo-cli
$ expo init MyProject
$ cd MyProject
$ expo start
```

# 詰まったところ
## ライブラリ選定
管理が大変なため，あまり多くを利用しすぎない．  
[Native Base](https://nativebase.io/)を中心に構築していく．

## TypeScriptの導入
[ここ](https://github.com/piotrwitek/react-redux-typescript-guide/blob/master/README.md#connect-with-react-redux)を参考に型を定義していく．

## 定数
* ヘッダーの色や背景色などはあらかじめ定数にしておくと良い

## 非同期処理
* プロジェクトがあまり大きくないときは`redux-thunk`  
Actionが責務を担う

* プロジェクトが大きいときは`redux-saga`  

## React Navigation の導入
v4以降はモジュールが細分化されているため，それぞれインストールする必要がある
```
$ yarn add react-navigation
$ expo install react-native-gesture-handler react-native-reanimated
$ yarn add react-navigation-stack
$ yarn add react-navigation-tabs
```

## React Navigation の BottomTab が効かない
初期状態でスタイリングされている `alignItems` をコメントアウトする
```typescript
import {StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    // alignItems: 'center',
    justifyContent: 'center',
  },
});
```

## `enum` を用いてAction Typeを定義したときに複数のkeyに状態が保存されてしまう
次のようにAction Typeを`enum`を用いて定義する
```typescript
export enum FooActionType {
  SetFoo,
}

export enum BarActionType {
  SetBar,
}
```

次のようにActionとReducerを定義する
```typescript
export const setFooAction =  (foo: Foo): setFoo => {
  return  {type: FooActionType.SetFoo, payload: foo};
};
```

```typescript
export default combineReducers<RootState>({
  foo: fooReducer,
  bar: barReducer,
});
```

```typescript
const INITIAL_STATE: FooState = {
  foo: null
};

export default (state: FooState = INITIAL_STATE, action: FooAction): FooState => {
  switch (action.type) {
    case FooActionType.SetFoo:
      return {...state, foo: action.payload};
    default:
      return state;
  }
}
```

```typescript
const INITIAL_STATE: BarState = {
  bar: null
};

export default (state: BarState = INITIAL_STATE, action: BarAction): BarState => {
  switch (action.type) {
    case BarActionType.BarType:
      return {...state, Bar: action.payload};
    default:
      return state;
  }
}
```

このとき`setFooAction`を発行するとReducerで`BarActionType.BarType`にも引っかかり，Storeの`Bar`にも同じ状態が保存されてしまう．  
理由は`FooActionType.SetFoo`と`BarActionType.SetBar`がどちらも`0`だからである．  

* 解決法
```typescript
export enum FooActionType {
  SetFoo = 'setFoo',
}

export enum BarActionType {
  SetBar = 'setBar',
}
```

Reducerは型を見て判断しているわけではないことに気をつける

## `redux-thunk` を用いた場合の `props` への紐付け

```typescript
export interface FooState {
  count: number
}

export interface BarState {
  count: number
}

export interface RootState {
  foo: FooState
  bar: BarState
}

interface OwnProps {
}

interface DispatchProps {
 bar: () => void
}

interface StateToProps {
  foo: number
}

const mapStateToProps = (states: RootState, ownProps: OwnProps): StateToProps => ({
  foo: states.foo.count,
});

const mapDispatchToProps = (dispatch: ThunkDispatch<{}, {}, any>, ownProps: OwnProps): DispatchProps => ({
  bar: async () => {
    await dispatch(barAction())
  }
});
```

* 注意点
  * `barAction`ではなく`barAction()`を記述すること

# ライブラリ
[Native Base](https://nativebase.io/)  
[React Navigation](https://reactnavigation.org/en/)

# 参考
[React/Redux約三年間書き続けたので知見を共有します](http://www.enigmo.co.jp/blog/tech/reactredux%E7%B4%84%E4%B8%89%E5%B9%B4%E9%96%93%E6%9B%B8%E3%81%8D%E7%B6%9A%E3%81%91%E3%81%9F%E3%81%AE%E3%81%A7%E7%9F%A5%E8%A6%8B%E3%82%92%E5%85%B1%E6%9C%89%E3%81%97%E3%81%BE%E3%81%99/)  
[ReactNative + Redux + NativeBase + Firebaseを利用したサンプルアプリ](https://github.com/fumiyasac/AnimationStyleSampleReactNative)