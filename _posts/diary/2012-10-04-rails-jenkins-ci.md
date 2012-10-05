---
layout: post
category : tech
tags : [Jenkins, Rails]
title: RailsプロジェクトでJenkinsを使ってCIしてみる
---
{% include JB/setup %}

会社のプロジェクトでRuby on Railsを使っていて、Jenkinsを使ってCI環境を整えたくって
ずっと試行錯誤していたのですが、最近ようやく出来上がったのでその全貌を明らかにします！

ちなみに全部Linuxでの話なので、Windowsの人は残念ながら役に立ちません...

# 目標

今回Jenkinsを使って以下のようなことがやりたかった。

1. githubにあるリポジトリのRSpecのテストを自動的に実行したい
2. Capybaraを使ったテストもやってるので、ブラウザが起動する環境で自動テストを実行したい。
   これは別途デスクトップ環境(今回の場合はUbuntuデスクトップ)を準備してそこでテストさせたい。
3. コードカバレッジを出力してるのでJenkinsから見れるようにしたい
4. テストケース一覧もJenkinsから見れるようにしたい

この目標に向かっていろいろと設定をしていきます。

# Javaのインストール

Jenkinsを動作させるにはJavaが必要ですのでJDKをインストールします。
この記事執筆時点では<a href="http://www.oracle.com/technetwork/java/javase/downloads/index.html" target="_blank">こちら</a>から
最新のJDKがダウンロードできます。JDKのダウンロードボタンをクリックして **Accept License Agreement** の
ラジオボタンにチェックを入れてからLinux用のJDKをダウンロードしてインストールします。

{% highlight console %}
$ cd /usr/local/src
$ wget http://download.oracle.com/otn-pub/java/jdk/7u7-b10/jdk-7u7-linux-x64.rpm
$ rpm -ivh jdk-7u7-linux-x64.rpm
{% endhighlight %}

これでインストール完了。

{% highlight console %}
$ java -version
{% endhighlight %}

と打って、Javaのバージョン番号が表示されれば成功。

# Jenkinsのインストール

yumで簡単にインストールできます。以下rootユーザーで作業します。

{% highlight console %}
$ wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
$ rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
$ yum install jenkins
{% endhighlight %}

これだけです。そして以下のコマンドで起動できます。

{% highlight console %}
$ /etc/init.d/jenkins start
{% endhighlight %}

この状態で`http://hostname:8080/`にアクセスするとJenkinsの画面が開きます。

![JenkinsDashBoard](/assets/img/2012-10-04-rails-jenkins-ci/jenkins_dashboard.png)

# プラグインのインストール

Jenkinsには豊富なプラグインが存在します。
今回の目標としていることも、プラグインなしには実現出来ないので、まず最初にプラグインをインストールします。
**Jenkinsの管理** ＞ **プラグインの管理** とたどっていき、 **利用可能** のタブをクリックします。
今回使うプラグインは以下の3つです。

* **Jenkins GIT plugin** Jenkinsからgitを扱えるようになります。
* **Jenkins Rake plugin** JenkinsからRakeタスクを実行できるようになります。
* **Jenkins ruby metrics plugin** Rcov形式のテストカバレッジのレポートをJenkins上で表示できるようになります。

![JenkinsDashBoard](/assets/img/2012-10-04-rails-jenkins-ci/jenkins_plugin_filter.png)

プラグインは大量にあって見つけるのが大変なので、フィルターにプラグイン名の一部などを入力して検索します。
プラグインが見つかったらチェックボックスをつけて画面下側にあるボタンをクリックしてインストールします。
**再起動せずにインストール** の方が再起動しないので、時間もかからず簡単そうです。

![JenkinsDashBoard](/assets/img/2012-10-04-rails-jenkins-ci/jenkins_plugin_install_button.png)

# スレーブサーバーの用意

Capybaraのテストをブラウザを使って実行するためのUbuntuデスクトップ環境を別途準備します。
(別にUbuntuデスクトップをマスターにしてもいいんですが、今回スレーブを試してみたかったので...)

## 各種インストール
スレーブサーバーにはマスターと同じようにJavaやJenkinsをインストールします。
Jenkinsはインストールするだけで起動はしなくても良いです。
また、ここでテストを実行することになるのでgitやrubyもインストールしておきます。

## 鍵の登録
このサーバーでgithubからcloneすることになるので、githubにJenkins実行ユーザーの鍵を登録しておきます。
yumでJenkinsをインストールした時に自動的に`jenkins`というユーザーができていて、そいつがJenkinsの実行ユーザーなっています。
ただ、`jenkins`というユーザーにはログインシェルがなくてログインすらできずに、鍵を作れなかったのでとりあえずログインシェルを登録して鍵を生成。

{% highlight console %}
$ usermod -s /bin/bash jenkins # rootユーザーで。とりあえずbashを設定
$ su - jenkins
$ ssh-keygen -t rsa # パスフレーズは空で。Enterキー連打で鍵生成
$ cat ~/.ssh/id_rsa.pub
{% endhighlight %}

`id_rsa.pub`の内容をgithubに登録します。鍵の登録はgithubにアクセスして、 **(画面右上の)Account Settings** > **SSH Keys** > **Add SSH key**
からできます。念のため登録がうまくいってるかどうか確認。

{% highlight console %}
$ ssh git@github.com
$ Hi tkengo! Youve successfully authenticated, but Github does not provide shell access.
{% endhighlight %}

# Jenkins(マスター側)の設定
さて、ここからJenkinsの設定をしていきます。
最初はどこに何を指定していいのかが全然わからないんですよね、これ...

## プロジェクトの新規作成
**新規ジョブ作成** から **フリースタイル・プロジェクトのビルド** を選択。
**ジョブ名** は適当に自分がわかる名前をいれればよいです。
![CreateNewJob](/assets/img/2012-10-04-rails-jenkins-ci/jenkins_create_new_job.png)

## スレーブの登録
先に作ったUbuntuのデスクトップを、マスターのJenkinsにスレーブとして登録します。
**Jenkinsの管理** > **ノードの登録** > **新規ノード作成** とたどっていきます。

![RegisterSlave](/assets/img/2012-10-04-rails-jenkins-ci/jenkins_register_slave.png)

こんな感じに設定します。 **ホスト** のところは自分の環境のホスト名に置き換えてください。

## 鍵の登録
スレーブを設定した時、起動方法に **SSH経由でUnixマシンのスレーブエージェントを起動** を選択しているので、
スレーブの`jenkins`ユーザーにマスターの`jenkins`ユーザーの鍵を登録します。
スレーブサーバーで鍵を作ったのと同じ要領でコマンドを実行します。
鍵ができたら、それをスレーブサーバーに転送して登録します。
コマンドラインでやるとするとこんな感じ。

{% highlight console %}
$ # マスターサーバーのjenkinsユーザーにログインしてやる
$ # スレーブサーバー名は適宜自分の環境の名前に置き換えて下さい
$ scp ~/.ssh/id_rsa.pub jenkins@jenkins.ubuntu.local:/home/jenkins
$ ssh jenkins@jenkins.ubuntu.local
$ # ここからスレーブサーバーのjenkinsユーザーでの操作
$ cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 600 ~/.ssh/authorized_keys
{% endhighlight %}

念のためパスワードなしでログイン出来るかどうか確認。

{% highlight console %}
$ # マスターサーバーのjenkinsユーザーで
$ ssh jenkins@jenkins.ubuntu.local
{% endhighlight %}

## SCM
githubを使うので当然gitを選択します。
リポジトリ名とブランチ名を入力するエリアが出てくるのでそれぞれ入力します。

![SettingSCM](/assets/img/2012-10-04-rails-jenkins-ci/jenkins_setting_scm.png)

リポジトリを入力すると認証に失敗した旨のエラーメッセージが出ますが、認証のための鍵は、
実際にテストを実行するスレーブサーバー(Ubuntuデスクトップ)側で登録しているので無視します。

## ビルド・トリガ
テストを走らせるためのトリガを定義します。githubにpushされたら自動的にテスト実行というのが理想的なのですが、
Jenkinsが社内ローカル環境にあるため、githubからのpush通知を受け取れません。

なので、githubを5分毎に見に行って(ポーリング)更新があればテストを走らせるようにトリガを設定しました。

![SettingBuildTrigger](/assets/img/2012-10-04-rails-jenkins-ci/jenkins_setting_build_trigger.png)

## ビルド
いよいよここでテスト実行の設定をします。といっても、単にテストの時に手動で打ってるコマンドを実行するだけなんですが。
**シェルの実行** を選んで、今回の場合はこんな感じ。

![SettingExecuteShell](/assets/img/2012-10-04-rails-jenkins-ci/jenkins_setting_execute_shell.png)

*export DISPLAY=:0.0* の部分はUbuntuデスクトップでコマンドラインからGUIのブラウザを起動させるために必要な設定です。
GUIのプログラムを起動するためのディスプレイの番号を指定しないといけないようです。
これを指定していないとFirefoxが起動せずに、1日くらいはまりました。

## ビルド後の処理
ビルドが終わった後の処理を定義していきます。
まず、今回のRailsプロジェクトでは<a href="https://github.com/nicksieger/ci_reporter" target="_blank">ci_reporter</a>というgemを使って、
テスト結果をJUnit互換のxml形式に吐き出しています。
それをJenkinsに食べさせてJenkinsから参照できるようにします。

![SettingAfterBuild1](/assets/img/2012-10-04-rails-jenkins-ci/jenkins_setting_after_build1.png)

それと、<a href="https://github.com/colszowka/simplecov" target="_blank">simplecov</a>と<a href="https://github.com/fguillen/simplecov-rcov" target="_blank">simplecov-rcov</a>というgemを使ってコードカバレッジも出力しています。
これも同様にJenkinsに食べさせることで、Jenkinsから参照できるようにします。

![SettingAfterBuild2](/assets/img/2012-10-04-rails-jenkins-ci/jenkins_setting_after_build2.png)

# ビルド実行
さて、ようやくここまででJenkinsの設定が終わりました。どきどきしながら **ビルド実行** をクリックします。

![SettingExecuteBuild](/assets/img/2012-10-04-rails-jenkins-ci/jenkins_execute_build.png)

VNCでUbuntuデスクトップを覗いてみると...おおおおFirefoxが起動してテストしてくれてる！
試行錯誤を繰り返した末にきちんと動いてくれたのは感動的でした。

あとはJenkinsが取り込んでくれたテスト結果やコードカバレッジのグラフなんかをみてニヤニヤします。

# 終わりに
どうでしたか。導入がなかなか大変だとは思いますが、環境を作ってしまえばとても楽しいです。
ここに紹介した他に、今うちのチームではビルド後の処理として、テストが終わる度にIRCチャンネルに通知するという処理をやっています。
それでこんな運用をしています。

* プログラマがソースコードを修正してgithubにpush
* Jenkinsが5分に一回拾ってくれて自動的にテストを走らせる
* ディレクターやリーダーなどがIRCでその通知を確認
* Jenkinsからgitのログを参照して、変更点を確認する

まだまだ始めたばかりで試験的ではありますが、なによりちゃんとCIされてる感じで楽しいです。

CIライフはじめてみませんか。
