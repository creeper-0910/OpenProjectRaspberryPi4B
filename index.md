## [RaspberryPi](https://amzn.asia/d/ikVpfuX)で[OpenProject](https://www.openproject.org/)を動かす 


### はじめに

これはmadewhatnowさんが作成したRaspberryPi用のOpenProjectインストールガイドの日本語訳です。

[OpenProject](https://www.openproject.org/)は、完全な機能を備えたオープンソースのプロジェクト管理ツールボックスです。  
無料のコミュニティ版もリリースされていますが、一部のプレミアム機能は有料のクラウド版とエンタープライズ版に限定されています。  
セルフホストする場合には4GBのRAMを搭載したLinuxベースのシステムが推奨されており、インストールが容易になる.deb/.rpmパッケージやdockerイメージを提供しています。  

新しいRaspberry Pi 4はコンパクトで電力効率に優れ、4コアのSoCと4GBのRAMを搭載し、理論上OpenProjectを実行できます。これは小規模なチームやローカルのテスト設備にとって、非常に消費電力の少ない良い方法になり得るでしょう。  

しかしここで問題が発生します。OpenProjectはARMアーキテクチャをサポートしていないため、.deb/.rpmパッケージやdockerに基づいたインストール方法は利用できません。 
サポートフォーラムには、何年も前からこの件に関するスレッドがいくつかありますが、明確な解決法はほとんどありません。(このページに辿り着いた方々はすでにご存知かもしれません)

## 良いニュース: OpenProjectは(非公式に)ARMで実行することができます!

Raspberry PiでOpenProjectを動作させることは可能です!4GBのRAMを搭載したRPi4(または互換ボード)を強くお勧めします。  
2GBでも十分なスワップ領域があれば何とかなるかもしれませんが、検証されておらず、非推奨です。  
インストールとコンパイルの過程で、メモリ使用量は最大で約3.1GBになります。  

## 今すぐ利用したいのですが、既製のシステムイメージはありますか？

元の著者が作成したイメージがありますが、2021年のものでありセキュリティ的リスクが多いため推奨されません。  

ファイルは[ここ](https://drive.google.com/file/d/1qBzWME8BCVja0HickLo_SOcvsUL5sUEC/view?usp=sharing)にあり、ファイルサイズは約12GBで16GB以上のSDカードで利用できます。RPi4が必要で、2GBか4GBのメモリが必要です。SSH は有効になっており(pi//raspberry)、OpenProjectの管理者アカウントはデフォルトのアカウント名とパスワード(admin//admin) のままです。  
システムに接続するには無線LAN (/etc/wpa-supplicant/wpa-supplicant)かイーサネット二接続する必要があります。OpenProjectでメール通知を設定してください(下記参照)。

問題や成功報告は本家のリポジトリにお願いします。

## ステータス(2021年2月)

2020年2月にmadewhatnowがOpenProject 10のためにこの説明書を作成しました。システム・イメージや様々な修正を求める多数のメールを受け取り、2021年にようやくプロトコルの再検討を行いました。OpenProjectは最近バージョン11をリリース、そして少々驚いたことに、2021ではプロセスがかなりスムーズになっている。RPi 4上のRaspian Liteイメージから始めて、全プロセスは最小限の問題で数時間で完了した。以下のプロトコルは、必要な変更を反映して更新されている。

OpenProjectフォーラムでは、微調整や修正に関する議論が続いています: https://community.openproject.org/topics/6873
## 主な課題

* 随時更新

### OpenprojectをRaspberryPi OSにインストールする方法

この手順は[ここ](https://docs.openproject.org/installation-and-operations/installation/manual/)にある手動インストール手順をベースにしています。公式の手順は古くなっており複数の問題がありますが、手順が細かく記載されています。
## RaspberryPi OSのセットアップ

[RaspberryPi OS Liteのイメージをダウンロード](https://www.raspberrypi.com/software/)し、microSDカードに[書き込み](https://www.balena.io/etcher/)ます(32GBのものを利用していますが、もう少し小さなSDカードでも問題有りません)。
SSHとローカルWifiネットワークを利用する場合:

* 空のSSHファイルを作成し、[SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/)を有効にする(または直接キーボード、マウス、ディスプレイに接続) 
* [wpa_supplicant.conf](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md)ファイルを作成し、ローカル無線LANの認証情報を設定する(またはPiをLANに接続する)。
* ルーターでIPアドレスを確認し、これを使って好きなSSHクライアント(Puttyなど)でログインします。

システムを更新し、必要なシステム・パッケージ、PostgreSQL、オプションのmemcachedパッケージをインストールします。ここでnpmとnodejsをインストールするのも良いかもしれません。

```
sudo apt-get update -y
sudo apt-get full-upgrade -y
sudo apt-get install -y zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libgdbm-dev libncurses5-dev automake libtool bison libffi-dev git curl poppler-utils unrtf tesseract-ocr catdoc libxml2 libxml2-dev libxslt1-dev memcached postgresql postgresql-contrib libpq-dev libsass1 libsass-dev sassc npm nodejs
```

## ファイルシステムを拡張する
SDカードの容量を最大限利用するためにファイルシステムを拡張します;
**raspi-config**を起動し**advanced options**の**expand filesystem**を選択した後、終了し再起動します。


## スワップ領域を4GBに拡張する
基本的には不要ですが、2GB版では役に立つかもしれません(非推奨)。過剰な読み書きによりメモリーカードの破損を避けるため、インストール後は必ず設定を元に戻してください。

```
sudo dphys-swapfile swapoff
sudo nano /etc/dphys-swapfile 
***CONF_SWAPSIZEとCONF_MAXSWAPの行を見つけ、4096またはそれに近い値に変更する***
sudo dphys-swapfile swapon
```

## ユーザーアカウントのセットアップ

openprojectグループ/ユーザーを作成する。標準インストールでは、ここではパスワードに'openproject'を設定した。

```
sudo groupadd openproject
sudo useradd --create-home --gid openproject openproject
sudo passwd openproject 
#パスワードを入力
```

PostgreSQLシステムユーザに切り替えて、データベース用ユーザを作成してください。公式ドキュメントの通り二作成すると権限のないユーザが作成されますが、-sdフラグを指定するとCREATEDB権限を持つスーパーユーザが作成されます。

```
sudo su - postgres
createuser -drsW openproject
```
PostgreSQLのユーザとその権限を確認してください。CREATE DB権限がない場合、インストールはその後失敗します。
```
psql
\du
exit
```
出力は以下のようになります:

```
postgres=# \du
                                    List of roles
  Role name  |                         Attributes                         | Member of
-------------+------------------------------------------------------------+-----------
 openproject | Superuser, Create role, Create DB                          | {}
 postgres    | Superuser, Create role, Creaboglte DB, Replication, Bypass RLS | {}
```

 
データベースを作成し、元のユーザーアカウントに戻します:
 
 ```
 createdb -O openproject openproject
 exit
 ```

## ソフトウェアパッケージの準備

手動インストールの手順に従って、rbenvを使ってRubyをインストールします。可能な限りコンパイルを高速化するために4つのコアを使います(**-j 4**フラグを使用する)。これらの作業はopenprojectユーザーとして行う必要があるため、**su -- openproject**コマンドを使用します。  

```
sudo su -- openproject -login
git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.profile
echo 'eval "$(rbenv init -)"' >> ~/.profile
source ~/.profile
git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
MAKE_OPTS="-j 4" rbenv install 3.2.1
rbenv rehash
rbenv global 3.2.1

```
これには15分ほどかかります。 

バージョンを確認するには
```
ruby --version
```
を実行します。

手動インストールの手順に従って、nodenvを使ってRubyをインストールします。可能な限りコンパイルを高速化するために4つのコアすべてを使う(**-j 4**フラグに注意)。

```
git clone https://github.com/OiNutter/nodenv.git ~/.nodenv
echo 'export PATH="$HOME/.nodenv/bin:$PATH"' >> ~/.profile
echo 'eval "$(nodenv init -)"' >> ~/.profile
source ~/.profile
git clone https://github.com/OiNutter/node-build.git ~/.nodenv/plugins/node-build
MAKE_OPTS="-j 4" nodenv install 16.17.0
nodenv rehash
nodenv global 16.17.0
```

これには1分ほどかかります。 

## OpenProjectとコンパイルとインストール

注意 - 上記リンクのインストール手順ではまだstable/9を使用していますが、現在のリリースはstable/12です(2023年7月現在)。ここではstable/12を使用しています。

```
cd ~
git clone https://github.com/opf/openproject.git --branch stable/12 --depth 1
cd openproject
gem update --system 
gem install bundler
bundle install --deployment --without mysql2 sqlite development test therubyracer docker 
npm install
npm audit fix
```

**gem update --system** これには8分ほどかかります。 

**bundle install** これには5分ほどかかります。 

**npm install** これには15分ほどかかります。 


## 設定ファイルの準備

### データベース:
```
cp config/database.yml.example config/database.yml
nano config/database.yml
```
```
production:
  adapter: postgresql
  encoding: unicode
  database: openproject
  pool: 20
  username: openproject
  password: <パスワード>
```
  
### メールとmemcache:
 
gmail用のアプリパスワードを作成し、設定ファイルに含めます。.ymlのレイアウトはそのままにしてください。
 ```
 cp config/configuration.yml.example config/configuration.yml
 nano config/configuration.yml
 ```
 ```
production:                          
  smtp_address: smtp.gmail.com
  smtp_port: 587
  smtp_domain: smtp.gmail.com
  smtp_user_name: **@gmail.com**
  smtp_password: **enter gsuite app password here**
  smtp_enable_starttls_auto: true
  smtp_authentication: plain
  
** ファイル末尾に追加する:**
 
 rails_cache_store: :memcache
 ```


  
## OpenProjectのセットアップ
秘密鍵を設定し、環境変数 SECRET_KEY_BASE に格納してください。

```
su -- openproject -login
cd ~/openproject
echo "export SECRET_KEY_BASE=$(./bin/rake secret)" >> ~/.profile
source ~/.profile
RAILS_ENV="production" ./bin/rake db:create
RAILS_ENV="production" ./bin/rake db:migrate
RAILS_ENV="production" ./bin/rake db:seed
RAILS_ENV="production" ./bin/rake assets:precompile
exit
```
最後の処理は遅く、最低でも15分ほどかかります。

## ApacheとPassengerをインストール

```
sudo apt-get install -y apache2 libcurl4-gnutls-dev apache2-dev libapr1-dev libaprutil1-dev
sudo chmod o+x "/home/openproject"
sudo apt-get install -y dirmngr gnupg apt-transport-https ca-certificates curl
curl https://oss-binaries.phusionpassenger.com/auto-software-signing-gpg-key.txt | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/phusion.gpg >/dev/null
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger bullseye main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update
sudo apt-get install -y libapache2-mod-passenger
```
これにも20分ほどかかります。

続けて、ルートで実行します:
```
sudo a2enmod passenger
sudo a2enmod expires
sudo apache2ctl restart
```

/etc/apache2/sites-available/openproject.conf を作成します:
```
<VirtualHost *:80>
   ServerName yourdomain.com
   # !!! DocumentRoot を 'public' に指定してください!
   DocumentRoot /home/openproject/openproject/public
   <Directory /home/openproject/openproject/public>
      # これにより、Apacheのセキュリティ設定が緩和されます。
      AllowOverride all
      # マルチビューはオフにする必要があります。
      Options -MultiViews
      # Apache >= 2.4を使用している場合は、コメントアウトを解除してください:
      Require all granted
   </Directory>

   # ブラウザにアセットのキャッシュを要求する
   <Location /assets/>
     ExpiresActive On ExpiresDefault "access plus 1 year"
   </Location>

</VirtualHost>

PassengerDisableAnonymousTelemetry on
```


apacheサービスを再起動する: 

```
sudo systemctl restart apache2
```


And now: OpenProject should be accessible on **raspberry_pi-IP**:80. The standard login is admin // admin. If this does not work, see **Troubleshooting** below. 
OpenProjectは**raspberry_pi-IP**:80でアクセスできるはずです。デフォルトのログインは admin // admin です。これがうまくいかない場合は、以下の **トラブルシューティング** を参照してください。

## メール通知

上記のテンプレートは、対応するgoogleアカウントに「アプリ固有のパスワード」が作成され、同じアカウントでPOP/IMAPオプションが有効になっている場合に機能します。詳細はこちら: https://support.google.com/accounts/answer/185833?hl=en

## おめでとうございます！


電子メール通知を機能させるために、バックグラウンドジョブを有効にしてください。これは検証されていません。

ユーザーopenprojectに戻り、crontabを編集する:

```
sudo su --openproject -login
crontab -e 
**必要な場合はエディタを選択**
```

Insert the following line at the end of the file, make sure to use the correct Ruby version:

```
*/1 * * * * cd /home/openproject/openproject; /home/openproject/openproject/bin/rake jobs:workoff
```

保存して再起動すれば完了です!

## 結論

これがどの程度うまく機能するか、また将来の開発やアップデートに耐えられるかどうかについては、限られた経験しかないため、趣味や家庭、テスト環境以外でこの方法を利用するかどうか、慎重に考えてください。
OpenProjectは、将来ARMをサポート対象にするかもしれません（はっきり言って、サポートするのにそれほど問題はないはずです！）。

当たり前ですがこの手順を利用する場合には、パスワードや管理者アカウントを変更し、Raspberry Piのセキュリティを強化してください!

## トラブルシューティング

2020/02/25
2020/02/26
2020/05/05

### openprojectのログインページは表示されるのですが、admin // admin loginが利用できません
**RAILS_ENV="production" ./bin/rake db:seed** ステップ中に発生したエラーを見逃しているかもしれません。WorkPackages セクションを実行すると、**getaddrinfo** エラーが発生し、**rake aborted** と表示されます。
この問題は、スーパーユーザーとして以下のコマンドを実行し、/etc/hostsと/etc/resolv.confをすべてのユーザーが読めるようにすることで簡単に解決できます。これは確実に問題を解決するものではないようです。2020/02/26 

```
sudo chmod o+r /etc/resolv.conf
sudo chmod o+r /etc/hosts
```
### admin // adminでログインできません

IRBターミナルを開いて、管理者アカウントのパスワードを手動で変更することができる。これは一般的に、ユーザーがパスワードを忘れた場合に便利です。概要は[こちら](https://community.openproject.com/topics/6584)。

openprojectユーザーとして、IRBシェルを開く:

```
cd ~/openproject
RAILS_ENV=production bundle exec rails c
```
コンソールを開いたら、次のように入力します:
```
admin = User.find_by(login: 'admin')
admin.password = 'RaspberryPi' # 定義したパスワードのルールに従うこと！
admin.password_confirmation = 'RaspberryPi'

admin.save! # エラーの出力を確認する
```
最後のコマンドは「true」を返すはずです。新しいパスワードを使ってログインしてください。

### ログインページへアクセスした時のエラーページとエラーメッセージ:
"Could not spawn process for application /home/openproject/openproject: A timeout occurred while starting a preloader process."

Matteo Nespoli氏により発見、解決されました (ありがとうございます！): passenger.confに"PassengerStartTimeout 200"を追加します: https://community.bitnami.com/t/openproject-8-3-2-0-stack-not-working-after-fresh-install/67444/12

### テストメールは機能するが、通知が届かない

バックグラウンドジョブが有効になっていない可能性があります: 

https://docs.openproject.org/installation-and-operations/installation/manual/

### 他に質問は？

メールまたはこちらまでお問い合わせください: https://community.openproject.com/topics/6873


