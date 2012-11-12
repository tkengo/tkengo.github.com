---
layout: post
category : tech
tags : [PHP, CakePHP]
title: 日本人ならではのCakePHPへの貢献(CakePHP Advent Calendar 2011)
---
{% include JB/setup %}
この記事は<a href="http://atnd.org/events/22721" target="_blank"> CakePHP Advent Calendar 2011</a>に参加しています。13日目です。

前日は<a href="https://twitter.com/slumbers99" target="_blank">@slumbers99</a>さんの<a href="http://slumbers99.blogspot.jp/2011/12/prefixtheme-cakephpadvent2011.html" target="_blank">Prefixルーティングとthemeのススメ</a>でした。
3キャリア対応サイトをCakePHPで作った時に利用されたというPrefixルーティングとテーマのお話ですね。
テーマは僕も使ったことがありますが、とても便利な機能だと思います！便利なものはどんどん使って楽していきたいですね。


# まえおき

みなさん素晴らしい記事を次々と公開されていく中、なにを書こうかとずっと迷っていましたが、
今回はタイトルの通りCakePHPへの貢献について少し書きたいと思います。自分自身の経験によるものです。

CakePHPへの貢献しているでしょうか？僕は全然やってません。
やってませんが、CakePHPが1.1の頃からずっと使っていて、なにかしら役に立ちたいなぁと思って今まで過ごしてきました。
でも、ド田舎に住む1利用者に過ぎない僕みたいな技術者が、何が出来るだろうとも感じていました。
そういう人って実際に多いんじゃないかな。
CakePHP利用者で僕が昔から知ってる人達の日記を読みながら「勉強会とか楽しそうだな」と羨ましく思ったものです。

でも、最近ちょっと考えが変わってきました。 **自分が出来ること** をすればいいんです。
コアデベロッパでもないのに開発に携わろうとしたり、無理に遠くでやってるイベントに参加しようとしたり、そういうことがダメでした。
大事なので2回言いますが、 **自分が出来ること** をすればいいんです。僕の場合は2つありました。

* ブログでアウトプットする<!-- a -->
* CakePHPのドキュメントの翻訳

です。ブログでのアウトプットの重要性は、もう既に多くの人が声高に叫んでいます。
アウトプットを頑張るようにしました。

それから、僕は英語が好きです。英語力はある方とはいえないけど、読み書きはある程度できます。
なので、それをCakePHPと結びつけて役立てるにはどうしたらいいのか、っていうことを考えた時にドキュメントの翻訳にたどり着きました。

さて今回はこのドキュメントの翻訳について少し書いていきたいと思います。


# 最初にやること
これがCakePHP2.0のドキュメントです。<a href="http://book.cakephp.org/2.0/en/index.html" target="_blank">Cookbook</a>。
既にいくらか日本語化されている部分もあります。
翻訳に参加したければまずはCakePHPドキュメントチームに挨拶ぐらいしておきましょう
（しなくてもいいかもしれませんが、僕はメールを送りました）。
一応<a href="http://book.cakephp.org/2.0/ja/contributing/documentation.html" target="_blank">ここ</a>にも「翻訳に参加する場合はメールください」とありますし。
簡単な自己紹介と翻訳に参加したい旨のメールです。こんな感じ

    Dear CakePHP docs team.
    
    My name is Kengo Tateishi.
    I'm a programmer in Japan.
    
    I have developed a web application with CakePHP since its version was 1.1.x.
    I love CakePHP.
    
    I would like to participate in translating the CakePHP 2.0 document
    into Japanese in order to contribute to CakePHP and its users in
    Japan.
    
    Can I participate in translating?
    Can I send a pull request to the cakephp/docs repository on the github? 

すると次の日にはすぐグラハム(CakePHPのコアデベロッパ)から返信が来ました。

    ドキュメント翻訳への参加を歓迎します。
    github上でプルリクエストを送ってください。
    わからないことがあれば私に聞いてください。

という内容でした。なかなかフレンドリーな感じで、嬉しかったです。すぐお礼の返事を出しましたが、この時はやっぱりちょっとテンションあがりました。


# 翻訳環境の準備

## リポジトリ

CakePHPのドキュメントはgithub上で管理されています。まずはリポジトリをforkすることから。
CakePHPドキュメントのリポジトリにアクセスして、forkします。

![Repository Fork](/assets/img/2011-12-12-cakephp-advent-calendar/cakephp-docs-fork.png)

すると自分のアカウントにリポジトリがコピーされるので、それを作業環境にcloneします。

![Repository Clone](/assets/img/2011-12-12-cakephp-advent-calendar/cakephp-docs-clone.png)

{% highlight console %}
cd /home/tkengo/
git clone git@github.com:tkengo/docs.git
{% endhighlight %}

※sshでcloneするためにはSSHの公開鍵の登録が必要です。別途githubのヘルプを参照してください。

## 必要なツール群

CakePHPのドキュメントはsphinxで構築されます。 僕の場合、OSはFedora Coreを利用しています。 以下のツールが必要です

* Python

    SphinxがPythonで書かれているため必要です。2.7系を使いました。OSにプリインストールされていたものは最初は2.4だったけど、2.4じゃうまくいかなくて、2.7系をソースから入れました。

* Make

    いわずと知れたビルドツールです。大概のOSには入ってます

* Sphinx、PhpDomain for sphinx

    easy_installでインストールします。


easy_installはPythonモジュールを自動的にインストールしてくれるコマンド。setuptoolsというツールをインストールしたら使えるようになります。

http://peak.telecommunity.com/dist/ez_setup.py

ここにあるファイルをダウンロードして

{% highlight console %}
python /path/to/download/ez_setup.py
{% endhighlight %}

とすると使えるようになります。あとは

{% highlight console %}
easy_install sphinx
easy_install sphinxcontrib-phpdomain
{% endhighlight %}

とすればSphinxとPhpDomain for sphinxのインストールが終わって、準備完了。


# 翻訳作業とビルド

こっから頑張って翻訳していきます。CakePHPドキュメントリポジトリの構成は

`config/`  
`en/`  
`es/`  
`fr/`  
`ja/`  
`pt/`  
`ru/`  
`themes/`  
`Makefile`  
`README.mdown`  

となってます。`en/`以下にあるのが英語の原本なので、そこから`.rst`ファイルを`ja/`以下の同じ階層にコピーしてそれを翻訳します。
がりがり翻訳しましょう。

その後はビルドです。.rstファイルからHTMLを生成します。

{% highlight console %}
cd /home/tkengo
make html
{% endhighlight %}

こうすると全言語のHTMLファイルが次々と生成されます。今回は日本語の分だけでいいので

{% highlight console %}
make html-ja
{% endhighlight %}

とすると日本語分だけHTMLの生成をします。ビルドすると`build/`というディレクトリが新しく出来ているはず。そのなかに`.html`ファイルが出来上がります。


# プルリクエスト

実はまだドキュメントのコントローラ部分を翻訳中で、ここまで行ってないのですが、一応今後やる予定の作業です。
まず、更新した`.rst`ファイルをコミットして、githubにpushしましょう。

{% highlight console %}
git push origin master
{% endhighlight %}

さていよいよプルリクエストです。でも、実際にリクエストを送る前にちょっと立ち止まって。
自分が翻訳した部分のオリジナルのリポジトリに変更が加わってないかどうか確認をします。
もしここで、オリジナルに変更があった場合、その変更まで翻訳してから、プルリクエストを送りましょう。

![Pull Request](/assets/img/2011-12-12-cakephp-advent-calendar/cakephp-docs-pull-request.png)

ここまで来たら、あとはオリジナルのリポジトリに取り込まれるのを待つのみ。実際に取り込まれたらたぶんまたテンションが上がると思います。


# 終わりに

長々と書いてきましたが、どうでしたか。こんな僕でも、私でも、なにかしら **出来ること** があるよ、っていうアピールになればと思います。
大それたことじゃなくても良いと思います。

14日目は<a href="https://twitter.com/kaz_29" target="_blank">@kaz_29</a>さんです。ブログは<a href="http://d.hatena.ne.jp/kaz_29/" target="_blank">ここ</a>ですね。
よろしくおねがいします！
