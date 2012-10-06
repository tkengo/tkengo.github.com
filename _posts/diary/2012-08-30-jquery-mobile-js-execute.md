---
layout: post
category : tech
tags : [jQuery Mobile]
title: jQuery Mobileでscriptタグが2回実行されてハマった
---
{% include JB/setup %}

jQuery mobileを使ってると、scriptタグに書いたJavaScriptがなぜか2回実行されてしまって、かなりはまりました。

こんなコードで

{% highlight html %}
<html>
<head>
  :
  :
<script src="jquery-mobile.js"></script>
</head>
<body>
</body>
</html>
<script type="text/javascript">
alert('hoge');
</script>
{% endhighlight %}

これだと、最後の行のscriptタグが2回実行されて、hogeというアラートが2回表示されてしまいました。これが何故2回実行されてしまうのかが、まったくわからずに半日くらいはまってましたが、リファレンス読んだら解決しました。

ドキュメント読め俺...

で、解決策はというと

{% highlight html %}
<body>
<div data-role="page">
</div>
</body>
{% endhighlight %}

という風に、 `data-role` 属性に **page** という値を指定してあげると解決しました。jQuery mobileはページ間移動にはAjaxを使い、そのレスポンスをbodyに挿入して現在のページと差し替えます。Ajaxのレスポンスの中にdata-role="page"という属性があると、data-role="page"のタグに囲まれた部分だけを挿入する、という仕組みのようです。

リファレンスを読む限りはこの属性値は通常、ページ毎に配置するように書かれています。

参考
<a href="http://dev.screw-axis.com/doc/jquery_mobile/" target="_blank">jQuery mobile リファレンス</a>
