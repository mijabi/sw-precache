#  [![NPM version][npm-image]][npm-url] [![Build Status][travis-image]][travis-url] [![Dependency Status][daviddm-url]][daviddm-image]

# Service Worker Precache

Service Worker Precache is a module for generating a service worker that
precaches resources. It integrates with your build process. Once configured, it
detects all your static resources (HTML, JavaScript, CSS, images, etc.) and
generates a hash of each file's contents. Information about each file's URL and
versioned hash are stored in the generated service worker file, along with logic
to serve those files cache-first, and automatically keep those files up to date
when changes are detected in subsequent builds.

<small>Service Worker Precache は、リソースを事前定義するサービスワーカーを生成するためのモジュールです。ビルドプロセスと統合されています。設定が完了すると、静的リソース（HTML、JavaScript、CSS、イメージなど）がすべて検出され、各ファイルの内容のハッシュが生成されます。生成されたサービスワーカーファイルには、各ファイルの URL およびバージョン化されたハッシュに関する情報が格納され、それらのファイルを cache-first で処理するロジックが格納され、後続のビルドで変更が検出されると自動的にそのファイルが最新の状態に保たれます。</small>

Serving your local static resources cache-first means that you can get all the
crucial scaffolding for your web app—your App Shell—on the screen without having
to wait for any network responses.

<small>ローカルの静的リソースをキャッシュ優先で配信するということは、ネットワークの応答を待つことなく、画面上で Web アプリケーション（App Shell）の重要な足場をすべて取得できることを意味します。</small>

The module can be used in JavaScript-based build scripts,
like those written with [`gulp`](http://gulpjs.com/), and it also provides a
[command-line interface](#command-line-interface). You can use the module
directly, or if you'd prefer, use one of the [wrappers](#wrappers-and-starter-kits)
around `sw-precache` for specific build environments, like
[`webpack`](https://webpack.github.io/).

<small>このモジュールは、gulp で書かれたもののような JavaScript ベースのビルドスクリプトで使用でき、更にコマンドラインインターフェイスも提供します。モジュールを直接使用することもできますし、必要に応じて、webpack のような特定のビルド環境に sw-precache のラッパーを使用することもできます。</small>

It can be [used alongside](sw-precache-and-sw-toolbox.md) the [`sw-toolbox`](https://github.com/GoogleChrome/sw-toolbox)
library, which works well when following the App Shell + dynamic content model.

<small>これは [sw-toolbox ライブラリと一緒に使う](https://github.com/mijabi/sw-precache/blob/master/sw-precache-and-sw-toolbox.md)ことができます。これは、App Shell + 動的コンテンツモデルに従うときにうまくいきます。</small>

The full documentation is in this README, and the
[getting started guide](GettingStarted.md) provides a quicker jumping off point.

<small>完全なドキュメントはこの README にあり、[Getting Started guide](https://github.com/mijabi/sw-precache/blob/master/sw-precache-and-sw-toolbox.md) はより迅速なジャンプポイントを提供します。</small>

To learn more about the internals of the generated service worker, you can read
[this deep-dive](https://medium.com/@Huxpro/how-does-sw-precache-works-2d99c3d3c725)
by [Huang Xuan](https://twitter.com/Huxpro).

<small>生成されたサービスワーカーの内部について詳しくは、Huang Xuan の [deep-dive](https://medium.com/@Huxpro/how-does-sw-precache-works-2d99c3d3c725) をご覧ください。</small>

# Table of Contents
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Install](#install)
- [Usage](#usage)
  - [Overview](#overview)
  - [Example](#example)
  - [Considerations](#considerations)
  - [Command-line interface](#command-line-interface)
- [Runtime Caching](#runtime-caching)
- [API](#api)
  - [Methods](#methods)
    - [generate(options, callback)](#generateoptions-callback)
    - [write(filePath, options, callback)](#writefilepath-options-callback)
  - [Options Parameter](#options-parameter)
    - [cacheId [String]](#cacheid-string)
    - [clientsClaim [Boolean]](#clientsclaim-boolean)
    - [directoryIndex [String]](#directoryindex-string)
    - [dontCacheBustUrlsMatching [Regex]](#dontcachebusturlsmatching-regex)
    - [dynamicUrlToDependencies [Object&#x27e8;String,Buffer,Array&#x27e8;String&#x27e9;&#x27e9;]](#dynamicurltodependencies-objectstringbufferarraystring)
    - [handleFetch [boolean]](#handlefetch-boolean)
    - [ignoreUrlParametersMatching [Array&#x27e8;Regex&#x27e9;]](#ignoreurlparametersmatching-arrayregex)
    - [importScripts [Array&#x27e8;String&#x27e9;]](#importscripts-arraystring)
    - [logger [function]](#logger-function)
    - [maximumFileSizeToCacheInBytes [Number]](#maximumfilesizetocacheinbytes-number)
    - [navigateFallback [String]](#navigatefallback-string)
    - [navigateFallbackWhitelist [Array&#x27e8;RegExp&#x27e9;]](#navigatefallbackwhitelist-arrayregexp)
    - [replacePrefix [String]](#replaceprefix-string)
    - [runtimeCaching [Array&#x27e8;Object&#x27e9;]](#runtimecaching-arrayobject)
    - [skipWaiting [Boolean]](#skipwaiting-boolean)
    - [staticFileGlobs [Array&#x27e8;String&#x27e9;]](#staticfileglobs-arraystring)
    - [stripPrefix [String]](#stripprefix-string)
    - [stripPrefixMulti [Object]](#stripprefixmulti-object)
    - [templateFilePath [String]](#templatefilepath-string)
    - [verbose [boolean]](#verbose-boolean)
- [Wrappers and Starter Kits](#wrappers-and-starter-kits)
  - [CLIs](#clis)
  - [Starter Kits](#starter-kits)
  - [Recipes for writing a custom wrapper](#recipes-for-writing-a-custom-wrapper)
- [Acknowledgements](#acknowledgements)
- [Support](#support)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## Install

Local build integration:
```sh
$ npm install --save-dev sw-precache
```

Global command-line interface:
```sh
$ npm install --global sw-precache
```


## Usage

### Overview

1. **Make sure your site is served using HTTPS!**  
HTTPSを使用してサイトが配信されていることを確認してください。

Service worker functionality is only available on pages that are accessed via HTTPS.
(`http://localhost` will also work, to facilitate testing.) The rationale for this restriction is
outlined in the
["Prefer Secure Origins For Powerful New Features" document](http://www.chromium.org/Home/chromium-security/prefer-secure-origins-for-powerful-new-features).

<small>サービスワーカー機能は、HTTPS 経由でアクセスされるページでのみ使用できます。 （テストを容易にするために、http://localhost も機能します）。この制限の根拠は、[Prefer Secure Origins For Powerful New Features](http://www.chromium.org/Home/chromium-security/prefer-secure-origins-for-powerful-new-features) のドキュメントで概説されています。</small>

2. **Incorporate `sw-precache` into your `node`-based build script.**  
sw-precache を node ベースのビルドスクリプトに組み込みます。

It should work well with either `gulp` or `Grunt`, or other build scripts that run on
`node`. In fact, we've provided examples of both in the `demo/` directory. Each
build script in `demo` has a function called `writeServiceWorkerFile()` that
shows how to use the API. Both scripts generate fully-functional JavaScript code
that takes care of precaching and fetching all the resources your site needs to
function offline. There is also a [command-line interface](#command-line-interface)
available, for those using alternate build setups.

<small>これは、gulp または Grunt、またはノード上で実行されるその他のビルドスクリプトでうまくいくはずです。実際には、両方の例をdemo/ ディレクトリに用意しました。デモの各ビルドスクリプトには、API の使用方法を示す writeServiceWorkerFile() という関数があります。両方のスクリプトでは、サイトがオフラインで機能するために必要なすべてのリソースのプリキャッシングとフェッチを行う完全機能の JavaScript コードが生成されます。代替のビルド設定を使用している場合は、コマンドラインインターフェイスも利用できます。</small>

3. **Register the service worker JavaScript.**  
サービスワーカー JavaScript を登録します。

The JavaScript that's generated
needs to be registered as the controlling service worker for your pages. This
technically only needs to be done from within a top-level "entry" page for your
site, since the registration includes a [`scope`](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-registration-scope)
which will apply to all pages underneath your top-level page. [`service-worker-registration.js`](/demo/app/js/service-worker-registration.js) is a sample
script that illustrates the best practices for registering the generated service
worker and handling the various [lifecycle](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-state.1) events.

<small>生成された JavaScript は、ページの制御用サービスワーカーとして登録する必要があります。これは技術的には、トップレベルのページの下にあるすべてのページに適用されるスコープが登録に含まれているため、サイトのトップレベルの「エントリ」ページ内から行う必要があります。 service-worker-registration.js は、生成されたサービスワーカーを登録し、さまざまなライフサイクルイベントを処理するためのベストプラクティスを示すサンプルスクリプトです。</small>

### Example

The project's [sample `gulpfile.js`](/demo/gulpfile.js) illustrates the full use of sw-precache
in context. (Note that the sample gulpfile.js is the one in the `demo` folder,
not the one in the root of the project.) You can run the sample by cloning this
repo, using [`npm install`](https://docs.npmjs.com/) to pull in the
dependencies, changing to the `demo/` directory, running `` `npm bin`/gulp serve-dist ``, and
then visiting http://localhost:3000.

<small>プロジェクトの[サンプル gulpfile.js](https://github.com/mijabi/sw-precache/blob/master/demo/gulpfile.js) は、コンテキスト内で sw-precache を完全に使用する方法を示しています。サンプル gulpfile.js は、プロジェクトのルートにあるサンプルではなく、デモフォルダ内のサンプルです。サンプルを実行するには、このリポジトリを複製し、npm install を使用して依存関係を取得し、demo/ ディレクトリに変更し、 `npm bin /gulp serve-dist` を実行し、http://localhost:3000 にアクセスしてください。</small>

There's also a [sample `Gruntfile.js`](/demo/Gruntfile.js) that shows service worker generation in
Grunt. Though, it doesn't run a server on localhost.

<small>Grunt のサービスワーカーの生成を示す[サンプル Gruntfile.js](https://github.com/mijabi/sw-precache/blob/master/demo/Gruntfile.js) もあります。しかし、localhost ではサーバを実行しません。</small>

Here's a simpler gulp example for a basic use case. It assumes your site's resources are located under
`app` and that you'd like to cache *all* your JavaScript, HTML, CSS, and image files.

<small>ここでは、基本的なユースケースの簡単な例を示します。あなたのサイトのリソースが app の下にあり、JavaScript、HTML、CSS、およびイメージファイルをすべてキャッシュしたいと仮定しています。</small>

```js
gulp.task('generate-service-worker', function(callback) {
  var swPrecache = require('sw-precache');
  var rootDir = 'app';

  swPrecache.write(`${rootDir}/service-worker.js`, {
    staticFileGlobs: [rootDir + '/**/*.{js,html,css,png,jpg,gif,svg,eot,ttf,woff}'],
    stripPrefix: rootDir
  }, callback);
});
```

This task will create `app/service-worker.js`, which your client pages need to
[register](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#navigator-service-worker-register) before it can take control of your site's
pages. [`service-worker-registration.js`](/demo/app/js/service-worker-registration.js) is a ready-to-
use script to handle registration.

<small>このタスクは app/service-worker.js を作成します。クライアントページはサイトのページを管理する前に登録する必要があります。 service-worker-registration.js は、登録を処理するためにすぐに使えるスクリプトです。</small>

### Considerations 懸念事項

- Service worker caching should be considered a progressive enhancement. If you follow the model of
conditionally registering a service worker only if it's supported (determined by
`if('serviceWorker' in navigator)`), you'll get offline support on browsers with service workers and
on browsers that don't support service workers, the offline-specific code will never be called.
There's no overhead/breakage for older browsers if you add `sw-precache` to your build.  
<small>サービスワーカーのキャッシュは、pregressive enhancement と見なすべきです。サポートされている場合 if( 'serviceWorker' in navigator)、条件付きでサービスワーカーを登録するモデルに従うと、サービスワーカーを持つブラウザとサービスワーカーをサポートしていないブラウザでオフラインサポートが得られ、オフライン固有のコードは決して呼び出されません。sw-precache をビルドに追加しても、古いブラウザのオーバーヘッド/ブレークはありません。</small>

- **All** resources that are precached will be fetched by a service worker running in a separate
thread as soon as the service worker is installed. You should be judicious in what you list in the
`dynamicUrlToDependencies` and `staticFileGlobs` options, since listing files that are non-essential
(large images that are not shown on every page, for instance) will result in browsers downloading
more data than is strictly necessary.  
<small>プリキャッシュされたすべてのリソースは、サービスワーカーがインストールされるとすぐに別のスレッドで実行されるサービスワーカーによってフェッチされます。 dynamicUrlToDependencies と staticFileGlobs オプションでは、必須ではないファイル（すべてのページに表示されない大きな画像）をリストすると、必要以上にデータをダウンロードするブラウザが生じるため、慎重に選択する必要があります。</small>

- Precaching doesn't make sense for all types of resources (see the previous
point). Other caching strategies, like those outlined in the [Offline Cookbook](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/), can be used in
conjunction with `sw-precache` to provide the best experience for your users. If
you do implement additional caching logic, put the code in a separate JavaScript
file and include it using the `importScripts()` method.  
<small>すべてのタイプのリソースに対して、プレキャッシングが働くわけではありません（前のポイントを参照）。[Offline Cookbook](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/) に記載されているような他のキャッシング戦略は、sw-precache と組み合わせて使用​​して、ユーザーに最高のエクスペリエンスを提供することができます。追加のキャッシュロジックを実装する場合は、コードを別の JavaScript ファイルに置き、importScripts() メソッドを使用してコードをインクルードします。</small>

- `sw-precache` uses a [cache-first](http://jakearchibald.com/2014/offline-cookbook/#cache-falling-back-to-network) strategy, which results in a copy of
any cached content being returned without consulting the network. A useful
pattern to adopt with this strategy is to display a toast/alert to your users
when there's new content available, and give them an opportunity to reload the
page to pick up that new content (which the service worker will have added to
the cache, and will be available at the next page load). The sample [`service-worker-registration.js`](/demo/app/js/service-worker-registration.js) file [illustrates](https://github.com/GoogleChrome/sw-precache/blob/7688ee8ccdaddd9171af352384d04d16d712f9d3/demo/app/js/service-worker-registration.js#L51)
the service worker lifecycle event you can listen for to trigger this message.  
<small>sw-precache は [cache-first](https://jakearchibald.com/2014/offline-cookbook/#cache-falling-back-to-network) です。その結果、キャッシュされたコンテンツのコピーが、ネットワークに相談せずに返されます。この戦略を採用するのに便利なパターンは、[利用可能な新しいコンテンツがあるときにユーザーにトースト/アラートを表示し、ページをリロードして新しいコンテンツを取得する機会を与えられる](https://github.com/GoogleChromeLabs/sw-precache/blob/7688ee8ccdaddd9171af352384d04d16d712f9d3/demo/app/js/service-worker-registration.js#L51)ことです（サービスワーカーがキャッシュに追加し 、次のページのロード時に利用可能になります）。サンプル service-worker-registration.js ファイルは、このメッセージをトリガーするために待機できるサービスワーカーライフサイクルイベントを示しています。</small>

### Command-line interface

For those who would prefer not to use `sw-precache` as part of a `gulp` or
`Grunt` build, there's a [command-line interface](cli.js) which supports the
[options listed](#options-parameter) in the API, provided via flags or an
external JavaScript configuration file.

<small>gulp や Grunt ビルドの一部として sw-precache を使用したくない人には、API にリストされたオプションをサポートするコマンドラインインターフェイスが用意されています。これは、フラグや外部の JavaScript 設定ファイルによって提供されます。</small>

Hypenated flags are converted to camelCase [options](#options-parameter).  
Options starting with `--no` prefix negate the boolean value. For example, `--no-clients-claim` sets the value of `clientsClaim` to `false`.

<small>ハイフンフラグはキャメルケースオプションに変換されます。
--no プレフィックスで始まるオプションはブール値を否定します。たとえば、--no-clients-claim は、clientsClaim の値をfalse に設定します。</small>

**Warning:** When using `sw-precache` "by hand", outside of an automated build process, it's your
responsibility to re-run the command each time there's a change to any local resources! If `sw-precache`
is not run again, the previously cached local resources will be reused indefinitely.

<small>警告：sw-precache を "手作業で" 使用する場合、Grunt や Gulp などの自動ビルドプロセスの外では、ローカルリソースが変更されるたびにコマンドを再実行する必要があります。 sw-precache が再度実行されない場合、以前にキャッシュされたローカルリソースは無期限に再利用されます。</small>

Sensible defaults are assumed for options that are not provided. For example, if you are inside
the top-level directory that contains your site's contents, and you'd like to generate a
`service-worker.js` file that will automatically precache all of the local files, you can simply run

<small>提供されていないオプションについては、わかりやすいデフォルトが想定されます。たとえば、サイトのコンテンツを含むトップレベルディレクトリの内部にあり、すべてのローカルファイルを自動的にプリキャッシュする service-worker.js ファイルを生成する場合は、単に実行するだけです</small>

```sh
$ sw-precache
```

Alternatively, if you'd like to only precache `.html` files that live within `dist/`, which is a
subdirectory of the current directory, you could run

<small>あるいは、現在のディレクトリのサブディレクトリである dist/ 内に存在する .html ファイルのみをプリキャッシュする場合は、以下のように書けるでしょう</small>

```sh
$ sw-precache --root=dist --static-file-globs='dist/**/*.html'
```

**Note:** Be sure to use quotes around parameter values that have special meanings
to your shell (such as the `*` characters in the sample command line above,
for example).

<small>注意：シェルに特別な意味を持つパラメータ値については、引用符を使用してください（たとえば、上記のサンプルコマンドラインの * 文字など）。</small>

Finally, there's support for passing complex configurations using `--config <file>`.
Any of the options from the file can be overridden via a command-line flag.
We strongly recommend passing it an external JavaScript file defining config via
[`module.exports`](https://nodejs.org/api/modules.html#modules_module_exports).
For example, assume there's a `path/to/sw-precache-config.js` file that contains:

<small>最後に、--config <file> を使用して複雑な設定を渡すサポートがあります。ファイルからの任意のオプションは、コマンドラインフラグで上書きすることができます。私たちは、module.exports 経由で config を定義する外部 JavaScript ファイルを渡すことを強くお勧めします。たとえば、次のものを含む path/to/sw-precache-config.js ファイルがあるとします。</small>

```js
module.exports = {
  staticFileGlobs: [
    'app/css/**.css',
    'app/**.html',
    'app/images/**.*',
    'app/js/**.js'
  ],
  stripPrefix: 'app/',
  runtimeCaching: [{
    urlPattern: /this\\.is\\.a\\.regex/,
    handler: 'networkFirst'
  }]
};
```

That file could be passed to the command-line interface, while also setting the
`verbose` option, via

<small>このファイルは、コマンドラインインターフェイスに渡すこともできます。また、冗長オプションを設定することもできます。</small>

```sh
$ sw-precache --config=path/to/sw-precache-config.js --verbose
```

This provides the most flexibility, such as providing a regular expression for
the `runtimeCaching.urlPattern` option.

<small>これにより、runtimeCaching.urlPattern オプションの正規表現を提供するなどの柔軟性が最も高くなります。</small>

We also support passing in a JSON file for `--config`, though this provides
less flexibility:

<small>--config の JSON ファイルを渡すこともサポートしていますが、これは柔軟性に欠けます：</small>

```json
{
  "staticFileGlobs": [
    "app/css/**.css",
    "app/**.html",
    "app/images/**.*",
    "app/js/**.js"
  ],
  "stripPrefix": "app/",
  "runtimeCaching": [{
    "urlPattern": "/express/style/path/(.*)",
    "handler": "networkFirst"
  }]
}
```

## Runtime Caching

It's often desireable, even necessary to use precaching and runtime caching together. You may have seen our [`sw-toolbox`](https://github.com/GoogleChrome/sw-toolbox) tool, which handles runtime caching, and wondered how to use them together. Fortunately, `sw-precache` handles this for you.

<small>プリキャッシュとランタイムキャッシングを一緒に使用することは、しばしば望ましいことです。[sw-toolbox ツール](https://github.com/GoogleChrome/sw-toolbox)で、ランタイムキャッシング処理を見たことがあるかもしれず、それらを一緒に使用する方法を考えたかもしれません。幸いにも、sw-precache はそれを扱えます。</small>

The `sw-precache` module has the ability to include the `sw-toolbox` code and configuration alongside its own configuration. Using the `runtimeCaching` configuration option in `sw-precache` ([see below](#runtimecaching-arrayobject)) is a shortcut that accomplishes what you could do manually by importing `sw-toolbox` in your service worker and writing your own routing rules.

<small>sw-precache モジュールには、sw-toolbox のコードと独自の構成をともに含めることができます。 sw-precache（後述）の runtimeCaching 設定オプションを使用すると、サービスワーカーで sw-toolbox をインポートして独自のルーティングルールを作成することで、手動で行うことができるようになります。</small>

## API

### Methods

The `sw-precache` module exposes two methods: `generate` and `write`.

<small>sw-precache モジュールは、generate と write という2つのメソッドを公開しています。</small>

#### generate(options, callback)

`generate` takes in [options](#options), generates a service worker
from them and passes the result to a callback function, which must
have the following interface:

<small>generate オプションを使用して、それらからサービスワーカーを生成し、その結果をコールバック関数に渡します。コールバック関数には、次のインタフェースが必要です。</small>

`callback(error, serviceWorkerString)`

In the 1.x releases of `sw-precache`, this was the default and only method
exposed by the module.

<small>sw-precache の 1.x リリースでは、これはモジュールによって公開されるデフォルトの唯一のメソッドでした。</small>

Since 2.2.0, `generate()` also returns a
[`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

<small>2.2.0以降、generate() も Promise を返します。</small>

#### write(filePath, options, callback)
`write` takes in [options](#options), generates a service worker from them,
and writes the service worker to a specified file. This method always
invokes `callback(error)`. If no error was found, the `error` parameter will
be `null`

<small>write はオプションをとり、そこからサービスワーカーを生成し、サービスワーカーを指定されたファイルに書き込みます。このメソッドは常にコールバック（エラー）を呼び出します。エラーが見つからなかった場合、error パラメータは null になります。</small>

Since 2.2.0, `write()` also returns a [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

<small>2.2.0 以降、write() は常に Promsie を返します</small>

### Options Parameter

Both the `generate()` and `write()` methods take the same options.

<small>generate() と write() メソッドは同じオプションを取ります。</small>

#### cacheId [String]
A string used to distinguish the caches created by different web applications that are served off
of the same origin and path. While serving completely different sites from the same URL is not
likely to be an issue in a production environment, it avoids cache-conflicts when testing various
projects all served off of `http://localhost`. You may want to set it to, e.g., the `name`
property from your `package.json`.

<small>同じ起点とパスから提供される異なる Web アプリケーションによって作成されたキャッシュを区別するために使用される文字列。運用環境では、同じ URL から完全に異なるサイトを提供することは問題ではありませんが、http://localhost で提供されているさまざまなプロジェクトをテストする際のキャッシュ競合を回避します。たとえば、package.json の name プロパティに設定することができます。</small>

_Default_: `''`

#### clientsClaim [Boolean]
Controls whether or not the generated service worker will call
[`clients.claim()`](https://developer.mozilla.org/en-US/docs/Web/API/Clients/claim)
inside the `activate` handler.

<small>生成されたサービスワーカーが activate ハンドラ内で clients.claim() を呼び出すかどうかを制御します。</small>

Calling `clients.claim()` allows a newly registered service worker to take
control of a page immediately, instead of having to wait until the next page
navigation.

<small>clients.claim() を呼び出すと、新規に登録されたサービスワーカーは、次のページナビゲーションまで待つ必要はなく、直ちにページを制御できます。</small>

_Default_: `true`

#### directoryIndex [String]
Sets a default filename to return for URL's formatted like directory paths (in
other words, those ending in `'/'`). `sw-precache` will take that translation
into account and serve the contents a relative `directoryIndex` file when
there's no other match for a URL ending in `'/'`. To turn off this behavior,
set `directoryIndex` to `false` or `null`. To override this behavior for one
or more URLs, use the `dynamicUrlToDependencies` option to explicitly set up
mappings between a directory URL and a corresponding file.

<small>ディレクトリパスのようにフォーマットされた URL（つまり、'/' で終わる URL）を返すためのデフォルトのファイル名を設定します。 sw-precache は、その変換を考慮に入れ、 '/' で終わる URL に一致するものが他にない場合、相対 directoryIndex ファイルの内容を提供します。この動作をオフにするには、directoryIndex を false または null に設定します。 1つ以上の URL のこの動作を無効にするには、dynamicUrlToDependencies オプションを使用して、ディレクトリ URL と対応するファイルの間のマッピングを明示的に設定します。</small>

_Default_: `'index.html'`

#### dontCacheBustUrlsMatching [Regex]
It's very important that the requests `sw-precache` makes to populate your cache
result in the most up-to-date version of a resource at a given URL. Requests
that are fulfilled with out-of-date responses (like those found in your
browser's HTTP cache) can end up being read from the service worker's cache
indefinitely. Jake Archibald's [blog post](https://jakearchibald.com/2016/caching-best-practices/#a-service-worker-can-extend-the-life-of-these-bugs)
provides more context about this problem.

<small>sw-precache 要求が、指定された URL にある最新のバージョンのリソースでキャッシュ結果を取り込むことが非常に重要です。古くなったレスポンス（ブラウザの HTTP キャッシュにあるものなど）で満たされたリクエストは、サービスワーカーのキャッシュから無期限に読み込まれる可能性があります。 Jake Archibald のブログ記事では、この問題に関する詳細なコンテキストを提供しています。</small>

In the interest of avoiding that scenario, `sw-precache` will, by default,
append a cache-busting parameter to the end of each URL it requests when
populating or updating its cache. Developers who are explicitly doing "the right
thing" when it comes to setting HTTP caching headers on their responses might
want to opt out of this cache-busting. For example, if all of your static
resources already include versioning information in their URLs (via a tool like
[`gulp-rev`](https://github.com/sindresorhus/gulp-rev)), and are served with
long-lived HTTP caching headers, then the extra cache-busting URL parameter
is not needed, and can be safely excluded.

<small>このシナリオを避けるため、sw-precache はデフォルトで、キャッシュを作成または更新するときに要求する各 URL の末尾にキャッシュ無効化パラメータを追加します。応答に HTTP キャッシュヘッダーを設定する際に、明示的に「正しいこと」を行っている開発者は、このキャッシュ無効化をオプトアウトすることができます。たとえば、すべての静的リソースにすでに URL にバージョニング情報が含まれている場合（gulp-rev などのツールを使用）、長寿命の HTTP キャッシュヘッダーが提供されている場合は、余分なキャッシュ無効化 URL パラメータは不要です。安全に除外することができます。</small>

`dontCacheBustUrlsMatching` gives you a way of opting-in to skipping the cache
busting behavior for a subset of your URLs (or all of them, if a catch-all value
like `/./` is used).
If set, then the [pathname](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/pathname)
of each URL that's prefetched will be matched against this value.
If there's a match, then the URL will be prefetched as-is, without an additional
cache-busting URL parameter appended.

<small>dontCacheBustUrlsMatching を使用すると、URL のサブセット（/./ などのキャッチオール値が使用されている場合はそのすべて）のキャッシュ破棄動作をスキップすることができます。設定されている場合、プリフェッチされた各 URL のパス名はこの値と照合されます。一致するものがある場合、追加のキャッシュ無効化 URL パラメータを追加することなく、URL がそのままプリフェッチされます。</small>

Note: Prior to `sw-precache` v5.0.0, `dontCacheBustUrlsMatching` matched against
the entire request URL. As of v5.0.0, it only matches against the URL's
[pathname](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/pathname).

<small>注意：sw-precache v5.0.0 以前は、dontCacheBustUrlsMatching がリクエスト URL 全体と一致していました。 v5.0.0 以降、URL のパス名とのみ一致します。</small>

_Default_: not set

#### dynamicUrlToDependencies [Object&#x27e8;String,Buffer,Array&#x27e8;String&#x27e9;&#x27e9;]
Maps a dynamic URL string to an array of all the files that URL's contents
depend on. E.g., if the contents of `/pages/home` are generated server-side via
the templates `layout.jade` and `home.jade`, then specify `'/pages/home':
['layout.jade', 'home.jade']`. The MD5 hash is used to determine whether
`/pages/home` has changed will depend on the hashes of both `layout.jade` and
`home.jade`.

<small>動的 URL 文字列を、URL の内容が依存するすべてのファイルの配列にマップします。たとえば、/pages/home のコンテンツがテンプレート layout.jade と home.jade を介してサーバー側で生成された場合は、 '/pages/home':['layout.jade', 'home.jade'] を指定します。 MD5 ハッシュは、/pages/home が変更されたかどうかを、layout.jade と home.jade のハッシュに依存するかどうかを判断するために使用します。</small>

An alternative value for the mapping is supported as well. You can specify
a string or a Buffer instance rather than an array of file names. If you use this option, then the
hash of the string/Buffer will be used to determine whether the URL used as a key has changed.
For example, `'/pages/dynamic': dynamicStringValue` could be used if the contents of
`/pages/dynamic` changes whenever the string stored in `dynamicStringValue` changes.


<small>マッピングの代替値もサポートされています。ファイル名の配列ではなく文字列または Buffer インスタンスを指定できます。このオプションを使用すると、キーとして使用されるURLが変更されたかどうかを判断するために、文字列/バッファのハッシュが使用されます。たとえば、 '/pages/dynamic'：dynamicStringValue は、dynamicStringValue に格納された文字列が変更されるたびに /pages/dynamic の内容が変更された場合に使用できます。</small>

_Default_: `{}`

#### handleFetch [boolean]
Determines whether the `fetch` event handler is included in the generated
service worker code. It is useful to set this to `false` in development builds,
to ensure that features like live reload still work. Otherwise, the content
would always be served from the service worker cache.

<small>フェッチイベントハンドラが生成されたサービスワーカーコードに含まれるかどうかを決定します。ライブリロードのような機能が動作するように、開発ビルドでこれを false に設定すると便利です。それ以外の場合、コンテンツは常にサービスワーカーキャッシュから提供されます。</small>

_Default_: `true`

#### ignoreUrlParametersMatching [Array&#x27e8;Regex&#x27e9;]
`sw-precache` finds matching cache entries by doing a comparison with the full request URL. It's
common for sites to support URL query parameters that don't affect the site's content and should
be effectively ignored for the purposes of cache matching. One example is the
[`utm_`-prefixed](https://support.google.com/analytics/answer/1033867) parameters used for tracking
campaign performance. By default, `sw-precache` will ignore `key=value` when `key` matches _any_ of
the regular expressions provided in this option.
To ignore all parameters, use `[/./]`. To take all parameters into account when matching, use `[]`.

<small>sw-precache は、完全なリクエスト URL との比較を行うことで、一致するキャッシュエントリを見つけます。サイトでは、サイトのコンテンツに影響を与えない URL クエリパラメータをサポートすることが一般的であり、キャッシュマッチングの目的では効果的に無視する必要があります。 1つの例は、キャンペーンの掲載結果を追跡するために使用される utm_ 接頭辞パラメータです。デフォルトでは、このオプションで指定された正規表現に key が一致すると、sw-precache は key=value を無視します。すべてのパラメータを無視するには、[/./] を使用します。一致するときにすべてのパラメータを考慮するには、[] を使用します。</small>

_Default_: `[/^utm_/]`

#### importScripts [Array&#x27e8;String&#x27e9;]
Writes calls to [`importScripts()`](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/basic_usage#Importing_scripts_and_libraries)
to the resulting service worker to import the specified scripts.

<small>結果のサービスワーカーに importScripts() への呼び出しを書き込み、指定されたスクリプトをインポートします。</small>

_Default_: `[]`

#### logger [function]

Specifies a callback function for logging which resources are being precached and
a precache size. Use `function() {}` if you'd prefer that nothing is logged.
Within a `gulp` script, it's recommended that you use [`gulp-util`](https://github.com/gulpjs/gulp-util) and pass in `gutil.log`.

<small>どのリソースがプリキャッシュされているかを記録するためのコールバック関数とプリキャッシュサイズを指定します。何も記録されないようにするには、function(){} を使用してください。 gulp スクリプト内では、gulp-util を使用し、gutil.log を渡すことをお勧めします。</small>

_Default_: `console.log`

#### maximumFileSizeToCacheInBytes [Number]
Sets the maximum allowed size for a file in the precache list.

<small>プリキャッシュリスト内のファイルの最大許容サイズを設定します。</small>

_Default_: `2097152` (2 megabytes)

#### navigateFallback [String]
Sets an HTML document to use as a fallback for URLs not found in the `sw-precache` cache. This
fallback URL needs to be cached via `staticFileGlobs` or `dynamicUrlToDependencies` otherwise it
won't work.

<small>sw-precache キャッシュにない URL のフォールバックとして使用する HTML ドキュメントを設定します。この代替 URL は staticFileGlobs または dynamicUrlToDependencies 経由でキャッシュする必要があります。それ以外の場合は機能しません。</small>

```js
// via staticFileGlobs
staticFileGlobs: ['/shell.html']
navigateFallback: '/shell.html'

// via dynamicUrlToDependencies
dynamicUrlToDependencies: {
  '/shell': ['/shell.hbs']
},
navigateFallback: '/shell'
```

This comes in handy when used with a web application that performs client-side URL routing
using the [History API](https://developer.mozilla.org/en-US/docs/Web/API/History). It allows any
arbitrary URL that the client generates to map to a fallback cached HTML entry. This fallback entry
ideally should serve as an "application shell" that is able to load the appropriate resources
client-side, based on the request URL.

<small>これは、ヒストリー API を使用してクライアント側 URL ルーティングを実行する Web アプリケーションで使用すると便利です。これは、クライアントが生成した任意の URL を、フォールバックキャッシュされた HTML エントリにマップすることを許可します。このフォールバックエントリは、理想的には、要求 URL に基​​づいてクライアント側で適切なリソースをロードできる「アプリケーションシェル」として機能する必要があります。</small>

**Note:** This is **not** intended to be used to route failed navigations to a
generic "offline fallback" page. The `navigateFallback` page is used whether the
browser is online or offline. If you want to implement an "offline fallback",
then using an approach similar to [this example](https://googlechrome.github.io/samples/service-worker/custom-offline-page/)
is more appropriate.

<small>注：これは、失敗したナビゲーションを一般的な「オフラインフォールバック」ページにルーティングするために使用するためのものではありません。 navigateFallback ページは、ブラウザがオンラインであるかオフラインであるかにかかわらず使用されます。 「オフラインのフォールバック」を実装する場合は、この例のようなアプローチを使用する方が適切です。</small>

_Default_: `''`

#### navigateFallbackWhitelist [Array&#x27e8;RegExp&#x27e9;]
Works to limit the effect of `navigateFallback`, so that the fallback only
applies to requests for URLs with paths that match at least one
[`RegExp`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp).

<small>navigateFallback の効果を制限し、少なくとも1つの RegExp と一致するパスを持つ URL の要求にのみフォールバックが適用されるようにします。</small>

This option is useful if you want to fallback to the cached App Shell for
certain specific subsections of your site, but not have that behavior apply
to all of your site's URLs.

<small>このオプションは、サイトの特定のサブセクションに対してキャッシュされた App シェルにフォールバックしたいが、その動作がサイトのすべての URL に適用されないようにする場合に便利です。</small>

For example, if you would like to have `navigateFallback` only apply to
navigation requests to URLs whose path begins with `/guide/`
(e.g. `https://example.com/guide/1234`), the following configuration could be
used:

<small>たとえば、navigateFallback を /guide/ で始まる URL（たとえばhttps://example.com/guide/1234）の URL へのナビゲーションリクエストにのみ適用する場合は、次の設定を使用できます。</small>

```js
navigateFallback: '/shell',
navigateFallbackWhitelist: [/^\/guide\//]
```

If set to `[]` (the default), the whitelist will be effectively bypassed, and
`navigateFallback` will apply to all navigation requests, regardless of URL.

<small>[]（デフォルト）に設定すると、ホワイトリストは効果的にバイパスされ、URL に関係なくすべてのナビゲーション要求に navigateFallback が適用されます。</small>

_Default_: `[]`

#### replacePrefix [String]
Replaces a specified string at the beginning of path URL's at runtime. Use this
option when you are serving static files from a different directory at runtime
than you are at build time. For example, if your local files are under
`dist/app/` but your static asset root is at `/public/`, you'd strip 'dist/app/'
and replace it with '/public/'.

<small>実行時にパス URL の先頭にある指定された文字列を置き換えます。実行時にビルド時とは異なるディレクトリから静的ファイルを提供する場合は、このオプションを使用します。たとえば、ローカルファイルが dist/app/ の下にあり、静的資産ルートが /public/ にある場合は、 'dist/app/' を削除して '/public/' に置き換えます。</small>

_Default_: `''`

#### runtimeCaching [Array&#x27e8;Object&#x27e9;]
Configures runtime caching for dynamic content. If you use this option, the `sw-toolbox`
library configured with the caching strategies you specify will automatically be included in
your generated service worker file.

<small>動的コンテンツの実行時キャッシュを構成します。このオプションを使用すると、指定したキャッシュ方法で構成された sw-toolbox ライブラリが、生成されたサービスワーカーファイルに自動的に含まれます。</small>

Each `Object` in the `Array` needs a `urlPattern`, which is either a
[`RegExp`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)
or a string, following the conventions of the `sw-toolbox` library's
[routing configuration](https://googlechromelabs.github.io/sw-toolbox/api.html#main). Also required is
a `handler`, which should be either a string corresponding to one of the
[built-in handlers](https://googlechromelabs.github.io/sw-toolbox/api.html#handlers) under the `toolbox.` namespace, or a function corresponding to your custom
[request handler](https://googlechromelabs.github.io/sw-toolbox/api.html#handlers).
Optionally, `method` can be added to specify one of the [supported HTTP methods](https://googlechromelabs.github.io/sw-toolbox/api.html#expressive-approach) (_default: `'get'`_). There is also
support for `options`, which corresponds to the same options supported by a
[`sw-toolbox` handler](https://googlechromelabs.github.io/sw-toolbox/api.html#handlers).

<small>Array 内の各オブジェクトには、sw-toolbox ライブラリのルーティング設定の規則に従い、urlPattern（RegExpまたは文字列）が必要です。また、ハンドラが必要です。このハンドラは、ツールボックスの下のビルトインハンドラに対応する文字列でなければなりません。ネームスペース、またはカスタムリクエストハンドラに対応する関数が含まれます。オプションで、メソッドを追加して、サポートされる HTTP メソッドの1つを指定することができます（デフォルト： 'get'）。 sw-toolbox ハンドラでサポートされているオプションと同じオプションに対応するオプションもサポートされています。</small>

For example, the following defines runtime caching behavior for two different URL patterns. It uses a
different handler for each, and specifies a dedicated cache with maximum size for requests
that match `/articles/`:

<small>たとえば、次の例では、2つの異なる URL パターンのランタイムキャッシング動作を定義しています。これは、それぞれ異なるハンドラを使用し、/articles/ と一致する要求の最大サイズを持つ専用キャッシュを指定します。</small>

```js
runtimeCaching: [{
  urlPattern: /^https:\/\/example\.com\/api/,
  handler: 'networkFirst'
}, {
  urlPattern: /\/articles\//,
  handler: 'fastest',
  options: {
    cache: {
      maxEntries: 10,
      name: 'articles-cache'
    }
  }
}]
```

The [`sw-precache` + `sw-toolbox` explainer](sw-precache-and-sw-toolbox.md) has
more information about how and why you'd use both libraries together.

<small>sw-precache + sw-toolbox の説明文には、両方のライブラリを一緒に使う方法と理由についての詳細があります。</small>

_Default_: `[]`

#### skipWaiting [Boolean]
Controls whether or not the generated service worker will call
[`skipWaiting()`](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/skipWaiting)
inside the `install` handler.

<small>生成されたサービスワーカーがインストールハンドラ内で skipWaiting() を呼び出すかどうかを制御します。</small>

By default, when there's an update to a previously installed
service worker, then the new service worker delays activation and stays in a
`waiting` state until all pages controlled by the old service worker are
unloaded. Calling `skipWaiting()` allows a newly registered service worker to
bypass the `waiting` state.

<small>既定では、以前にインストールされたサービスワーカーの更新があると、新しいサービスワーカーはアクティブ化を遅延させ、古いサービスワーカーによって制御されるすべてのページがアンロードされるまで待機状態にとどまります。 skipWaiting() )を呼び出すと、新規に登録されたサービスワーカーが待機状態をバイパスできます。</small>

When `skipWaiting` is `true`, the new service worker's `activate` handler will
be called immediately, and any out of date cache entries from the previous
service worker will be deleted. Please keep this in mind if you rely on older
cached resources to be available throughout the page's lifetime, because, for
example, you [defer the loading of some resources](https://github.com/GoogleChrome/sw-precache/issues/180)
until they're needed at runtime.

<small>skipWaiting が true の場合、新しいサービスワーカーのアクティブハンドラが直ちに呼び出され、以前のサービスワーカーからの古いキャッシュエントリが削除されます。たとえば、実行時に必要になるまでいくつかのリソースの読み込みを延期するなど、古いキャッシュされたリソースをページの有効期間全体で利用できるようにする場合は、この点を覚えておいてください。</small>

_Default_: `true`

#### staticFileGlobs [Array&#x27e8;String&#x27e9;]
An array of one or more string patterns that will be passed in to
[`glob`](https://github.com/isaacs/node-glob).
All files matching these globs will be automatically precached by the generated service worker.
You'll almost always want to specify something for this.

<small>glob に渡される1つ以上の文字列パターンの配列。これらのグロブに一致するすべてのファイルは、生成されたサービスワーカーによって自動的に事前にキャッシュされます。ほとんどの場合、これに何かを指定する必要があります。</small>

_Default_: `[]`

#### stripPrefix [String]
Removes a specified string from the beginning of path URL's at runtime. Use this
option when there's a discrepancy between a relative path at build time and
the same path at run time. For example, if all your local files are under
`dist/app/` and your web root is also at `dist/app/`, you'd strip that prefix
from the start of each local file's path in order to get the correct relative
URL.

<small>実行時に指定された文字列をパスURLの先頭から削除します。このオプションは、ビルド時の相対パスと実行時の同じパスに不一致がある場合に使用します。たとえば、すべてのローカルファイルが dist/app/ の下にあり、Web ルートもdist/app/ にある場合は、正しい相対URLを取得するために、各ローカルファイルのパスの先頭からその接頭辞を削除します。</small>

_Default_: `''`

#### stripPrefixMulti [Object]
Maps multiple strings to be stripped and replaced from the beginning of URL paths at runtime.
Use this option when you have multiple discrepancies between relative paths at build time and
the same path at run time.
If `stripPrefix` and `replacePrefix` are not equal to `''`, they are automatically added to this option.

<small>実行時にURLパスの先頭から複数の文字列を削除して置換します。このオプションは、ビルド時に相対パスと実行時に同じパスとの間に複数の不一致がある場合に使用します。 stripPrefix と re​​placePrefixが '' と等しくない場合、自動的にこのオプションに追加されます。</small>

```js
stripPrefixMulti: {
  'www-root/public-precached/': 'public/',
  'www-root/public/': 'public/'
}
```

_Default_: `{}`

#### templateFilePath [String]

The path to the  ([lo-dash](https://lodash.com/docs#template)) template used to
generate `service-worker.js`. If you need to add additional functionality to the
generated service worker code, it's recommended that you use the
[`importScripts`](#importscripts-arraystring) option to include extra JavaScript rather than
using a different template. But if you do need to change the basic generated
service worker code, please make a copy of the [original template](https://github.com/googlechrome/sw-precache/blob/master/service-worker.tmpl),
modify it locally, and use this option to point to your template file.

<small>service-worker.jsを 生成するために使用された（lo-dash）テンプレートへのパス。生成されたサービスワーカーコードに機能を追加する必要がある場合は、別のテンプレートを使用するのではなく、JavaScript を追加するために importScripts オプションを使用することをお勧めします。しかし、基本的に生成されたサービスワーカーコードを変更する必要がある場合は、元のテンプレートのコピーを作成し、ローカルで修正し、このオプションを使用してテンプレートファイルを指定してください。</small>

_Default_: `service-worker.tmpl` (in the directory that this module lives in)

#### verbose [boolean]
Determines whether there's log output for each individual static/dynamic resource that's precached.
Even if this is set to false, there will be a final log entry indicating the total size of all
precached resources.

<small>定義済みの静的/動的リソースごとにログ出力を行うかどうかを決定します。これがfalseに設定されていても、すべてのキャッシュされたリソースの合計サイズを示す最終ログエントリがあります。</small>

_Default_: `false`


## Wrappers and Starter Kits

While it's possible to use the `sw-precache` module's API directly within any
JavaScript environment, several wrappers have been developed by members of the
community tailored to specific build environments. They include:

<small>任意の JavaScript 環境で sw-precache モジュールの API を直接使用することは可能ですが、特定のビルド環境に合わせたコミュニティのメンバーによっていくつかのラッパーが開発されています：</small>

- [`sw-precache-webpack-plugin`](https://www.npmjs.com/package/sw-precache-webpack-plugin)
- [`sw-precache-brunch`](https://www.npmjs.com/package/sw-precache-brunch)
- [`grunt-sw-precache`](https://www.npmjs.com/package/grunt-sw-precache)
- [`exhibit-builder-sw-precache`](https://www.npmjs.com/package/exhibit-builder-sw-precache)

There are also several starter kits or scaffolding projects that incorporate
`sw-precache` into their build process, giving you a full service worker out of
the box. The include:

<small>また、sw-precache をビルドプロセスに組み込んだ、いくつかのスターターキットまたは足場プロジェクトがあり、あなたにフルサービスワーカーを提供します：</small>

### CLIs

- [`polymer-cli`](https://github.com/Polymer/polymer-cli)
- [`create-react-pwa`](https://github.com/jeffposnick/create-react-pwa)

### Starter Kits

- [`react-redux-universal-hot-example`](https://github.com/bertho-zero/react-redux-universal-hot-example)
- [Polymer Starter Kit](https://github.com/polymerelements/polymer-starter-kit)
- [Web Starter Kit](https://github.com/google/web-starter-kit)

### Recipes for writing a custom wrapper

While there are not always ready-to-use wrappers for specific environments, this list contains some recipes to integrate `sw-precache` in your workflow:

<small>特定の環境で常に使用可能なラッパーはありませんが、このリストにはワークフローに sw-precache を統合するためのレシピがいくつか含まれています。</small>

- [Gradle wrapper for offline JavaDoc](https://gist.github.com/TimvdLippe/4c39b99e3b0ffbcdd8814a31e2969ed1)
- [Brunch starter for Phoenix Framework](https://gist.github.com/natecox/b19c4e08408a5bf0d4cf4d74f1902260)

## Acknowledgements

Thanks to [Sindre Sorhus](https://github.com/sindresorhus) and
[Addy Osmani](https://github.com/addyosmani) for their advice and code reviews.
[Jake Archibald](https://github.com/jakearchibald) was kind enough to review the service worker logic.

## Support

The team behind `sw-toolbox` and `sw-precache` have been busy creating [Workbox](https://workboxjs.org), which is a collection of libraries and tools that make it easy to build offline web apps. It’s a joining of [sw-toolbox](https://github.com/GoogleChrome/sw-toolbox) and [sw-precache](https://github.com/GoogleChrome/sw-precache) with more features and a modern codebase.

<small>sw-toolbox と sw-precache の背後にあるチームは、オフライン Web アプリケーションを簡単に構築できるようにするライブラリとツールの集まりである Workbox の作成に忙殺されています。これは、より多くの機能と最新のコードベースを備えた sw-toolbox と sw-precache の結合です。</small>

### What does this mean for sw-toolbox?

For now, it means we’ll continue to support both `sw-toolbox` and `sw-precache` with critical bug fixes and releases. However, non-critical bugs are unlikely to be addressed.

<small>今のところ、重要なバグ修正やリリースで sw-toolbox と sw-precache の両方を引き続きサポートする予定です。しかし、クリティカルではないバグには対処できません。</small>

### Should you switch to Workbox?

We would recommend Workbox for new projects, but there is no immediate need to switch if `sw-toolbox` / `sw-precache` meets all your needs in your current project. We will announce a deprecation plan for these modules once Workbox has feature parity with `sw-toolbox` and `sw-precache`.

<small>新しいプロジェクトのために Workbox をお勧めしますが、sw-toolbox/sw-precache が現在のプロジェクトのすべてのニーズを満たしているかどうかをすぐに切り替える必要はありません。 Workbox が sw-toolbox と sw-precache で機能パリティを持つと、これらのモジュールの非推奨プランが発表されます。</small>

In the meantime, you can get updates by following [@workboxjs](https://twitter.com/workboxjs).

<small>その間、@workboxjs に従うことでアップデートを入手できます。</small>

## License

Copyright © 2017 Google, Inc.

Licensed under the [Apache License, Version 2.0](LICENSE) (the "License");
you may not use this file except in compliance with the License. You may
obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

[npm-url]: https://npmjs.org/package/sw-precache
[npm-image]: https://badge.fury.io/js/sw-precache.svg
[travis-url]: https://travis-ci.org/GoogleChrome/sw-precache
[travis-image]: https://travis-ci.org/GoogleChrome/sw-precache.svg?branch=master
[daviddm-url]: https://david-dm.org/googlechrome/sw-precache.svg?theme=shields.io
[daviddm-image]: https://david-dm.org/googlechrome/sw-precache
