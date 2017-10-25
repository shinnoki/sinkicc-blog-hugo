---
title: "React NativeでTinder風UIの作り方を調査した"
date: 2017-07-28T16:17:33+00:00
thumbnail: "images/thumbnail/react-native.jpg"
tags:
  - React Native
---
近ごろはReact Native Meetup #6に参加したりと、React Nativeの情報を追っています。  
今回は、Tinder風にカードをYes/NoでSwipeしていくようなUIが作りたくて2つのプロジェクトを調査しました。

<!--more-->

* react-native-animated-demo-tinder
* react-native-tinder-swipe-cards

の2つのgithubプロジェクトを調査したのですが、ハマりどころがあったのでメモしておきます。

### [react-native-animated-demo-tinder](https://github.com/brentvatne/react-native-animated-demo-tinder)

Tinder風な画面のデモですが、現時点で更新は2016年3月で止まっている様子。  
動かすために2点修正する必要がありました。

3行目あたり

```
//import React, { AppRegistry, StyleSheet, Text, View, Animated, Component, PanResponder, } from 'react-native';
// ↓に修正
import React, { Component } from 'react';
import { AppRegistry, StyleSheet, Text, View, Animated, PanResponder, } from 'react-native';
```

75行目あたり

```
Animated.decay(this.state.pan, {
velocity: {x: velocity, y: vy},
deceleration: 0.98
//}).start(this._resetState)
// ↓に修正
}).start(this._resetState.bind(this))
```

内容については、Animatedの使い方をまだ勉強していなかったのでなるほどなといった感じです。  
動きについて割と丁寧にrender()内に記述する必要があるみたいですが、逆に言えば、自由にアニメーションを作れるということですかね。

### [react-native-tinder-swipe-cards](https://github.com/meteor-factory/react-native-tinder-swipe-cards)

上のreact-native-animated-demo-tinderをベースにnpmパッケージ化したもの。  
`npm install --save react-native-tinder-swipe-cards`したらおもむろにreact-nativeが削除されてしまったのは依存関係の問題でしょうか？  
再度`npm install`したら一応は動きました。

react-native-animated-demo-tinderにはないNoMoreCardsやMaybeの機能が実装されているので、実際に自分のアプリに組み込むならまずはこれを使ってみることになりそうです。

それにしても、React Nativeの情報は賞味期限が短いので追っていくのが大変ですね。  
javascriptでネイティブアプリを作ろうというコンセプトでは今までもTitaniumとかPhoneGapとかありましたが、Facebookが注力しているということで期待したいです。  
日本語の情報がまだ少ないというのもあって、調べたことはこまめにメモをしていきたいです。
