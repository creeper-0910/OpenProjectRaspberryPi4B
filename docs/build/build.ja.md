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
  password: openproject
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
これには2分ほどかかります。

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


OpenProjectは**raspberry_pi-IP**:80でアクセスできるはずです。デフォルトのログインは admin // admin です。これがうまくいかない場合は、以下の **トラブルシューティング** を参照してください。

## メール通知

上記のテンプレートは、対応するgoogleアカウントに「アプリ固有のパスワード」が作成され、同じアカウントでPOP/IMAPオプションが有効になっている場合に機能します。詳細はこちら: https://support.google.com/accounts/answer/185833?hl=en

## おめでとうございます！


電子メール通知を機能させるために、バックグラウンドジョブを有効にしてください。

ユーザーopenprojectに戻り、crontabを編集する:

```
sudo su -- openproject -login
crontab -e 
**必要な場合はエディタを選択**
```

ファイルの末尾に以下の行を挿入し、Rubyのバージョンが正しいことを確認します:

```
*/1 * * * * cd /home/openproject/openproject; RAILS_ENV="production" ./bin/rake jobs:workoff
```
Rails 6以降では"config.cache_classes"をfalseに設定する必要があります:

```
nano /home/openproject/openproject/environments/production.rb
***config.cache_classes=trueの行を見つけ、falseに変更する***
```

保存して再起動すれば完了です!

## 結論

これがどの程度うまく機能するか、また将来の開発やアップデートに耐えられるかどうかについては、限られた経験しかないため、趣味や家庭、テスト環境以外でこの方法を利用するかどうか、慎重に考えてください。
OpenProjectは、将来ARMをサポート対象にするかもしれません（はっきり言って、サポートするのにそれほど問題はないはずです！）。

当たり前ですがこの手順を利用する場合には、パスワードや管理者アカウントを変更し、Raspberry Piのセキュリティを強化してください!
