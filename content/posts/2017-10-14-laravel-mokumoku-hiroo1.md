---
title: "Laravel5.5でVue.jsを使ってみた"
date: "2017-10-14T17:00:00+09:00"
thumbnail: "images/thumbnail/vuejs.jpg"
tags:
  - Laravel
  - Vue.js
---

この記事は[広尾Laravelもくもく会#1](https://hiro-o.connpass.com/event/67795/)にて取り組んだ内容のメモになります。  
前々からReactやVueといったモダンJSフレームワークを使ってみたかったのですが、Laravelには最初からvueが含まれているという話を聞いて、触ってみました。

<!--more-->

## Vue.jsを使うまで
Laravel5.3以降ではpackage.jsonに標準でvueが含まれるため、すぐに使い始めることができます。

``` ruby
# Laravelプロジェクト作成
$ composer create-project laravel/laravel testapp
$ cd testapp
# node_modulesのダウンロード
$ npm install
# assetのコンパイル
$ npm run dev
```

プロジェクト作成時に `resources/assets/js` 内に `app.js` と `components/Example.vue` が入っていて、view側で以下のように記述すれば、`#app` にExampleコンポーネントがマウントされます。

``` app.blade.php
<body>
    <div id="app"></div>
    <script src="{{ mix('js/app.js') }}"></script>
</body>
```

### assetの自動コンパイル
この方法だと、ファイルを編集するごとに`$ npm run dev`を実行してからブラウザをリロードする必要があります。  
代わりに`$ npm run watch`を使うことで、assetファイルの更新を監視して自動コンパイルしてくれます。

ここまでやると、ブラウザのリロードも面倒くさくなりますね。  
[Browsersync](https://www.browsersync.io/)を使うとできそうなのですが、時間内に試すことはできませんでした。


## Bootstrap Vue
元となる管理画面がbootstrapを使っているので、ひとまず`bootstrap vue`などと検索してみたところ、[Bootstrap Vue](https://bootstrap-vue.js.org/)というライブラリを見つけました。

### インストール方法

``` ruby
$ npm i --save bootstrap-vue
```

* app.jsの `window.Vue = require('vue');` の下に追加

``` app.js
// Bootstrap-Vue
window.BootstrapVue = require('bootstrap-vue');
Vue.use(BootstrapVue);
```

* app.scssに追加

``` app.scss
// Bootstrap-Vue
@import "~bootstrap-vue/dist/bootstrap-vue";
```

ここまでやっておきながら、デザインだけなら普通にdivにclassを指定すればいいだけないので、今のところ使う必然性を感じませんでした。実際に開発を進めていくと便利なのかもしれません。


## Vue-Router
[vue-router](https://router.vuejs.org/ja/)はVue.jsでSPAを構築するときに使うルーティングライブラリです。

### インストール方法

``` ruby
$ npm i --save bootstrap-vue
```
* app.jsの `window.Vue = require('vue');` を変更  

``` app.js
import Vue from 'vue';
import VueRouter from 'vue-router';
Vue.use(VueRouter);
```
※ `window.VueRouter = require('vue-router');` だと `Uncaught TypeError: VueRouter is not a constructor` と出て使えませんでした

### 基本的な使い方
* [公式ドキュメントのサンプルコード](https://router.vuejs.org/ja/essentials/getting-started.html)がわかりやすいです

``` app.js
const Foo = { template: '<div>foo</div>' };
const Bar = { template: '<div>bar</div>' };

const routes = [
    { path: '/foo', component: Foo },
    { path: '/bar', component: Bar }
];

const router = new VueRouter({
    mode: 'history', // historyモード
    routes
});

const app = new Vue({
    router
}).$mount('#app');
```
* HTML側に `<router-view></router-view>` と書くと、それがrouterの内容に置き換わります
* デフォルトのhashモードだとURLは `http://localhost/#/foo` みたいな感じになります
* historyモードを使うと#が取れます。 [美しいですね!](https://router.vuejs.org/ja/essentials/history-mode.html)

## 取り組んだ感想
今日は本当に触りの部分だけでAPIとの連携などはできませんでしたが、引き続き触ってみて業務で使えるレベルまで持っていきたいと思っています。  
また、Reactで使えるMobXはVue.jsに似ているという話をよく聞くので、React Nativeを使う際にも還元されればいいかなと思います。


## 参考URL
* [アセットのコンパイル(Laravel Mix) 5.5 Laravel](https://readouble.com/laravel/5.5/ja/mix.html)
* [Laravel 5.4 + Vue 2.1 でSPAアプリケーション作成チュートリアル - Qiita](https://qiita.com/acro5piano/items/f33381fc60408abe9865)
* [はじめに — Vue.js](https://jp.vuejs.org/v2/guide/)
* [Bootstrap Vue](https://bootstrap-vue.js.org/)
* [Introduction · vue-router](https://router.vuejs.org/ja/)
