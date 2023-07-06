# トラブルシューティング

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


