## [RaspberryPi](https://amzn.asia/d/ikVpfuX)で[OpenProject](https://www.openproject.org/)を動かす 


### はじめに

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

2020年2月にmadewhatnowさんがOpenProject 10のためにこの説明書を作成しました。システム・イメージや様々な修正を求める多数のメールを受け取り、2021年にようやくプロトコルの再検討を行いました。OpenProjectは最近バージョン11をリリース、そして少々驚いたことに、2021ではプロセスがかなりスムーズになっている。RPi 4上のRaspian Liteイメージから始めて、全プロセスは最小限の問題で数時間で完了した。以下のプロトコルは、必要な変更を反映して更新されている。

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
Expand the filesystem to take full advantage of the size of SD card chosen;
Start **raspi-config** and go to **advanced options**, **expand filesystem**.  
Quit and reboot.


## Increase swap space to 4 GB
Not actually necessary, but possibly useful for the 2 GB board version (not recommended). Make sure to reverse the setting after the installation to avoid burning out the memory card by excessive read/write cycles.

```
sudo dphys-swapfile swapoff
sudo nano /etc/dphys-swapfile 
***find the CONF_SWAPSIZE=100 line and change to 4096 or similar***
sudo dphys-swapfile swapon
```

## Set up user accounts

Create the openproject group/user. For the standard installation I set 'openproject' as the password.

```
sudo groupadd openproject
sudo useradd --create-home --gid openproject openproject
sudo passwd openproject #(***pick a password***)
```

Switch to the PostgreSQL system user and create the database user. The official documentation creates a user without privileges, the -sd flags will create a superuser with CREATEDB privileges.

```
sudo su - postgres
createuser -drsW openproject
```
Check PostgreSQL users and their privileges. If the CREATE DB privilege is missing, the installation will fail at a later point.
```
psql
\du
exit
```
The output should look like this:

```
postgres=# \du
                                    List of roles
  Role name  |                         Attributes                         | Member of
-------------+------------------------------------------------------------+-----------
 openproject | Superuser, Create role, Create DB                          | {}
 postgres    | Superuser, Create role, Creaboglte DB, Replication, Bypass RLS | {}
```

 
 Create the database and revert to the standard 'pi' user account:
 
 ```
 createdb -O openproject openproject
 exit
 ```

## Preparing software packages

Following the manual installation suggestions, rbenv is used to install Ruby. Whenever possible, I use all four cores to speed up compiling (hence the **-j 4** flags). These tasks have to be done as the openproject user, hence the **su -- openproject** command.  

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
This will take 15 minutes. 

Earlier version of the protocol required an older (2.6) version, now 2.7.2 compiles just fine. 

Check version with
```
ruby --version
```

Following the manual installation suggestions, nodenv is used to install Ruby. Whenever possible, I use all four cores to speed up compiling (watch out for the **-j 4** flags).

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

This will take 1 minute.  

## Compile and install OpenProject

Careful - the manual installation I linked to above still uses stable/9, the current release is stable/11 (as of Feb 2021). So, using release stable/11 here. Earlier issues with bcrypt and other gems appear to have resolved themselves. 

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

**gem update --system** will take about 8 minutes.

**bundle install** will take about 5 minutes.

**npm install** will take  15 minutes.


## Prepare config files

### Database:
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
  
### Email & memcache:
 
Create an app password for gmail, and include it in the config file. Make sure to keep the .yml layout intact. 
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
  
** Add at the end of the file:**
 
 rails_cache_store: :memcache
 ```


  
## Setup OpenProject
Set secret key and store in environmental variable SECRET_KEY_BASE. 

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
Last one is the slow one - expect to wait for 15 minutes. 


## Install Apache & Passenger

```
sudo apt-get install -y apache2 libcurl4-gnutls-dev apache2-dev libapr1-dev libaprutil1-dev
sudo chmod o+x "/home/openproject"
sudo apt-get install -y dirmngr gnupg apt-transport-https ca-certificates curl
curl https://oss-binaries.phusionpassenger.com/auto-software-signing-gpg-key.txt | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/phusion.gpg >/dev/null
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger bullseye main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update
sudo apt-get install -y libapache2-mod-passenger
```
This will, again, take a while - expect 20 minutes. 

Once passenger is nearly done it lists the config information for Apache2. Open a second shell as root (or copy the lines into a text editor) and perform the edits as given. Use the information below as a guideline, the actual ones are likely to be different! Raspian uses a split Apache2 configuration, where information is spread out in several files (and not centralized in /etc/apache.conf)
Install passenger for 'ruby', when asked. 

In  /etc/apache2/mods-available/passenger.load:
```
LoadModule passenger_module /home/openproject/.rbenv/versions/2.6.3/lib/ruby/gems/2.6.0/gems/passenger-6.0.4/buildout/apache2/mod_passenger.so
```

In /etc/apache2/mods-available/passenger.conf:
```
<IfModule mod_passenger.c>
     PassengerRoot /home/openproject/.rbenv/versions/2.6.3/lib/ruby/gems/2.6.0/gems/passenger-6.0.4
     PassengerDefaultRuby /home/openproject/.rbenv/versions/2.6.3/bin/ruby
</IfModule>

```

Then, still as root:
```
sudo a2enmod passenger
sudo a2enmod expires
sudo apache2ctl restart
```

Create the /etc/apache2/sites-available/openproject.conf file:
```
<VirtualHost *:80>
   ServerName yourdomain.com
   # !!! Be sure to point DocumentRoot to 'public'!
   DocumentRoot /home/openproject/openproject/public
   <Directory /home/openproject/openproject/public>
      # This relaxes Apache security settings.
      AllowOverride all
      # MultiViews must be turned off.
      Options -MultiViews
      # Uncomment this if you're on Apache >= 2.4:
      Require all granted
   </Directory>

   # Request browser to cache assets
   <Location /assets/>
     ExpiresActive On ExpiresDefault "access plus 1 year"
   </Location>

</VirtualHost>
```
 
Edit the /etc/apache2/apache2.conf file:

```
<Directory /home/openproject/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
        Allow from all
</Directory>
```

This might create some security risks, but is required as long as the openproject folder lives in /home/openproject instead of the /var/www/ folders. 


Reboot or restart the apache service again: 

```
sudo systemctl restart apache2
```


And now: OpenProject should be accessible on **raspberry_pi-IP**:80. The standard login is admin // admin. If this does not work, see **Troubleshooting** below. 

## Email notifications

The template configuration above works, if an 'app specific password' is created in the corresponding google account, and the POP/IMAP option is activated for the same account. See details here: https://support.google.com/accounts/answer/185833?hl=en

## Congratulations!


For email notification to work well, background jobs have to be enabled. This is untested. 

Switch back to the user openproject, and edit crontab:

```
sudo su --openproject -login
crontab -e **select the editor of choice if required**
```

Insert the following line at the end of the file, make sure to use the correct Ruby version:

```
*/1 * * * * cd /home/openproject/openproject; /home/openproject/.rvm/gems/ruby-2.6.3/wrappers/rake jobs:workoff
```

Save & reboot. Enjoy. 

## Final Thoughts

Given the limited experience of how well this works, and if it will survive future development and updates, think carefully about whether you want to use this installation for anything other than a hobby, home or testing environment. OpenProject might feel inspired to make ARM a supported architecture in the future (clearly, it shouldn't be too much of a problem!). 

And obviously, if you plan to make this installation available on any sort of network, change the various passwords and user accounts, and shore up security on Raspian!


## Troubleshooting

2020/02/25
2020/02/26
2020/05/05

### I made it through the process, the openproject login page shows up, but I cannot use the admin // admin login. 

You might have missed an error that triggered during the **RAILS_ENV="production" ./bin/rake db:seed** step. When the WorkPackages section is executed, a **getaddrinfo** error is triggered, and **rake aborted** is displayed. 
This can be easily fixed by executing the following commands as a superuser (e.g. from the pi account), to make sure that /etc/hosts and /etc/resolv.conf can be read by all users. This doesn't seem to reliably fix the problem - removed from the above instructions. 2020/02/26 

```
sudo chmod o+r /etc/resolv.conf
sudo chmod o+r /etc/hosts
```
### I still cannot login with admin // admin

You can manually change the password for the admin account by opening an IRB terminal. This is generally useful if a user forgot their password, and outlined originally [here](https://community.openproject.com/topics/6584).

As the openproject user, open an IRB shell:

```
cd ~/openproject
RAILS_ENV=production bundle exec rails c
```
Once the console is open, type out the following:
```
admin = User.find_by(login: 'admin')
admin.password = 'RaspberryPi' # Must confirm to the password rules you defined!
admin.password_confirmation = 'RaspberryPi'

admin.save! # Watch the output for errors
```
The final command should return 'true'. Use the new password to login.

### Error page when accessing the login page & error message the error log:
"Could not spawn process for application /home/openproject/openproject: A timeout occurred while starting a preloader process."

Discovered, solved & reported by Matteo Nespoli (thanks!): Add "PassengerStartTimeout 200" line to passenger.conf, as outlined here: https://community.bitnami.com/t/openproject-8-3-2-0-stack-not-working-after-fresh-install/67444/12

### Test emails work, but notifications don't arrive

Background jobs are probably not activated, follow these steps: 

https://docs.openproject.org/installation-and-operations/installation/manual/

### More questions?

Email or ask here: https://community.openproject.com/topics/6873


