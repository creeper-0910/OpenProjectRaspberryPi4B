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
