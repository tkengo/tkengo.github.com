---
layout: post
category : tech
tags : [Rails]
title: Asset Pipeline について勉強した
---
{% include JB/setup %}

ペパボは今、開発言語をRubyに切り替え始めています。

僕も今、Ruby on Railsを使って開発をしているので、本格的にRuby周りの勉強をし始めているのですが、未だにAsset Pipelineがよくわかってないので、この機会にドキュメントを読んできちんと調べてみました。

参考にしたのはここ。
<a href="http://guides.rubyonrails.org/asset_pipeline.html" target="_blank">http://guides.rubyonrails.org/asset_pipeline.html</a>

やっぱり公式のドキュメントが一番わかりやすいかな？自分自身がちゃんと覚えるように読みながらまとめてみました。

# Asset Pipelineってなに？
Rails3.1より実装された仕組みで、大きくわけて以下の3つの機能があるようです。

1. CSSやJavaScriptなどの、いわゆるasset(資源)と呼ばれるファイルを1つにまとめてくれます。普通cssやjsなどのファイルは、開発しやすいように機能毎や画面毎に分けて作業すると思いますが、これはブラウザにとってはファイルの分だけサーバーへのリクエストが発生して効率が悪いです。ブラウザにとっては、ファイルが1つにまとまっていたほうが効率がいいのです。
2. まとめられたファイルは、同時にミニファイ(圧縮)されます。スペースやタブ、改行などを取り除き、コンパクトに圧縮してくれます。jsファイルに限っては、それだけではなくもう少し複雑なアルゴリズムで圧縮されるようです。
3. CSSはSASS、JavaScriptはCoffeeScript、を使って記述できます。もちろんSASSやCoffeeScriptを使わずにそのままCSSやJavaScriptで記述することもできます。また、ERB(Embedded Ruby)でも記述できます。

こういった機能がそなわっています。


# コードの結合 - マニフェストファイル
まず1つ目の機能の資源の結合について、これは`Sprocket`というgemがあって、そいつがいろいろとよしなにしてくれるわけですが、そのためにマニフェストファイルというものを使います。これは、どのファイルを読みこんで、1ファイルにまとめればいいのかをSprocketが把握するためのものです。Railsでアプリケーションを新しく作った時、app/assets/javascripts/application.jsというファイルがデフォルトであると思います。このファイルを開いて中身を見てみると

{% highlight js %}
//= require jquery
//= require jquery_ujs
//= require_tree .
{% endhighlight %}

といった行があります。`//=`で始まる行はSprocketのディレクティブで、この例では、requireとrequire_treeというコマンドが使われています。名前でなんとなくわかると思いますが、requireは1つのファイルを、require_treeは再帰的にディレクトリ以下の全ファイルを、読み込もうとします。この場合、jquery.js、jquery_ujs.js、それとapp/assets/javascripts/以下にあるすべての.jsファイルを1つのファイルにまとめてくれます。

同じように`app/assets/stylesheets/applications.css`というファイルもデフォルトであると思いますので、このファイルの中身を覗いてみると

{% highlight css %}
/**
 *
 *
 *= require_self
 *= require_tree .
 */
{% endhighlight %}

こういう行があるのがわかります。cssの場合は`*=`で始まるんですね。

それから、各ディレクトリのindexという名前のファイルもマニフェストファイルとして読み込んでくれるようです。
例えば、`lib/assets/library_name/`というディレクトリに何かしらのJavaScriptのライブラリがあったとすると、
`lib/assets/library_name/index.js`というファイルに

{% highlight js %}
//= require_tree .
{% endhighlight %}

と書いておけば、そのライブラリに関するファイルを1ファイルにまとめてくれるとのことです。

# コードの圧縮 - コンプレッサー
2つ目の機能でコードのミニファイをあげましたが、これは特になにも指定しなくても勝手にやってくれます。ただ、デフォルトの圧縮以外にもCSSやJavaScriptの圧縮機構を自分で指定することができます。app/config/application.rbで

{% highlight ruby %}
config.assets.css_compressor = :yui
{% endhighlight %}

と指定すると、YUI CSS compressorのアルゴリズムを使ってCSSを圧縮します。JavaScriptの場合は

{% highlight ruby %}
config.assets.js_compressor = :uglifier
{% endhighlight %}

と指定すると、uglifierを使ってJavaScriptを圧縮します。また、外部ライブラリではなく自分が開発した独自の圧縮アルゴリズムを指定することもできるみたいです。compressというメソッドを持つクラスを定義して、そのインスタンスを指定します。

まぁこのコード圧縮のアルゴリズムはほとんどの場合、デフォルトから変えなくても問題ないレベルなんだと思います。

# コードの事前処理 - プリプロセッシング
3つ目の機能に、CSSはSASS、JavaScriptはCofeeScript、を使って記述できると紹介しましたが、これは拡張子で判断してくれます。たとえばSASSを使ってスタイルシートを書きたい場合は、以下のようなファイル名でファイルを作ります。

`app/assets/stylesheets/projects.css.scss`

拡張子が.scssになってるのがわかります。これはSASSで処理されて、その結果がブラウザに渡るようになっています。CoffeeScriptも同じで

`app/assets/javascripts/projects.js.coffee`

とすると、CoffeeScriptで処理された結果がブラウザに渡ります。また、拡張子は複数つなぐこともできて、たとえば

`app/assets/stylesheets/projects.css.scss.erb`

というファイルがあれば、まずERBで処理されて、その結果がSASSで処理され、さらにその結果がブラウザに渡ります。すげー。

# Asset Pipelineの使い方
Rails3.1からは `app/assets/` `lib/assets/` `vendor/assets/` というディレクトリがあります。

* `app/assets/` アプリケーション内で利用する資源を配置
* `lib/assets/` 共通に利用されるライブラリ内で利用される資源を配置
* `vendor/assets/` サードパーティー製のライブラリやプラグインで利用される資源を配置

上記3つのディレクトリ以下にそれぞれ、stylesheets / javascripts / images というディレクトリがあり、その中に実際にCSS、JavaScript、画像を配置します。これらの資源は、Railsでは

{% highlight erb %}
<%= stylesheet_link_tag "hogehoge" %>
<%= javascript_include_tag "fugafuga" %>
<%= image_tag "piyopiyo.png" %>
{% endhighlight %}

などのようにして呼び出せば、上記3つのディレクトリを検索して、そのうちのどこかに目的のファイルがあればそれを読み込みます。

また、以下のようにマニフェストファイル(app/assets/stylesheets/application.css)を指定した場合

<%= stylesheet_link_tag "application" %>

development環境では結合されずにそのまま複数のファイルで読み込まれますが、production環境になればコードは結合されて1つのファイルにまとまって読み込まれるようになっています。


# 終わりに
最近Railsをさわってて思いますが、本当に良くできたフレームワークですね。Rubyそれ自体もすごく使いやすくて拡張しやすい言語だと思いますが、その機能をフルに使いこなしたRailsは素晴らしいです。

AssetPipelineについては、他にもFingerprintingやPrecompileなどもあるようなのですが、これはまた別の機会にでも。。。

個人的にはもともとCakePHPを使ってたので、CakePHPのほうが詳しくて、サクサク開発できるんですが、Railsも便利なので、どっちともウォチしていこうかなーと思います。CakePHPもRailsも、これから進化し続けていくことに期待！
