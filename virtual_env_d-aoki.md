# バージョン一覧

| ソフトウェア | ver |
| - | - |
| PHP | 7.3 |
| Laravel | 6.0 |
| MYSQL | 5.7 |
| Nginx | 1.21.0 |
| OS | CentOS7 |

# 環境構築手順
## ホストOS情報  
- OS:Windows10  
- Virtual Box:ver6.1.14  
- vagrant_2.2.16  

## ゲストOSの起動
vagrant boxのダウンロード
```
vagrant box add centos/7

# 今回使用するソフトはVirtualBoxのため3を選択  

1) hyperv
2) libvirt
3) virtualbox
4) vmware_desktop

Enter your choice: 3
```
Vagrantの作業ディレクトリを用意  

任意の名称でディレクトリを作成
```
mkdir virtual_env_manual

# 作成したフォルダの中で以下のコマンドを実行
cd virtual_env_manual

# ダウンロードしたboxを使用
vagrant init centos/7
```
Vagrantfileの編集
```
# 変更点① コメントイン
config.vm.network "forwarded_port", guest: 80, host: 8080

# 変更点②　コメントイン
config.vm.network "private_network", ip: "192.168.33.19"

# 変更点③
config.vm.synced_folder "../data", "/vagrant_data"
# ↓ 以下に編集
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```
Vagrant プラグイン,vagrant-vbguestのインストール
```
vagrant plugin install vagrant-vbguest
```
Vagrantを使用してゲストOSの起動
```
vagrant up
```
ゲストOSへのログイン
```
vagrant ssh

# ゲストOSに正常にログインされている場合以下文言が表示
[vagrant@localhost ~]$
```

## パッケージのインストール
下記コマンド実行でgitなどの開発に必要なパッケージを一括でインストール
```
sudo yum -y groupinstall "development tools"
```

## PHPのインストール

Laravelを動作させるためにはPHPのバージョン7以上が必要  
そのためyumではなく外部パッケージツールをダウンロードして、そこからPHPをインストールを行う  
(yumコマンドを使用してPHPをインストールした場合古いバージョンがインストールされる為)
```
sudo yum -y install epel-release wget
sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
sudo rpm -Uvh remi-release-7.rpm
sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
```
## composerのインストール
PHPのパッケージ管理ツールであるcomposerをインストール
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

# どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動
sudo mv composer.phar /usr/local/bin/composer
```

## データベースのインストール
MySQLのversion5.7をインストール  
```
sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
sudo yum install -y mysql-community-server
```

## Nginxのインストール
```
sudo vi /etc/yum.repos.d/nginx.repo

#以下の内容をファイル内に書き込み

[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
```
上記ファイル作成後以下コマンド実行にてNginxのインストール
```
sudo yum install -y nginx
```
Nginxの起動
```
sudo systemctl start nginx
```

## Laravelを動かす為の変更
Nginxの設定ファイルを編集  
```
sudo vi /etc/nginx/conf.d/default.conf

#以下変更点
server {
  listen       80;
  server_name  192.168.33.19;　

  root /vagrant/laravel_app/public; # 追記
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  location ~ \.php$ {
#    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;
      include        fastcgi_params;
  }
```
php-fpm の設定ファイルを編集
```
sudo vi /etc/php-fpm.d/www.conf

#以下変更点
;24行目近辺
user = apache
group = apache
# ↓ 以下に編集
user = nginx
group = nginx
```
ログインファイルへの書き込み権限の付与
```
cd /vagrant/laravel_app
sudo chmod -R 777 storage
```
起動
```
sudo systemctl restart nginx
sudo systemctl start php-fpm
```

# 所感
サーバーレッスンを通じて、コマンド操作に苦戦を強いられた。

GUIの操作に慣れ親しんでいた為、ソフトウェアの公式ページを開きダウンロードしたアプリケーションをもとにインストールを行っていたものがコマンドのみで行えることがイメージを持ち難かった。  
特にディレクトリの位置関係でコマンド操作が行えないこともありワーキングディレクトリからの相対パスや絶対パスの記述に関する位置関係は強く意識付けられた。  
また権限の変更など根幹に関わる重要な操作も可能であるため扱いに慎重さが求められること、今まで以上に確かな知識を身に付け幅を広げていきたいと感じさせられた。
# 参考サイト
- [NotePM MarkdownでTable(表テーブル)を書く](https://notepm.jp/help/markdown-table)  
- [Qiita 更新！！Laravel6/7 (laravel/ui)でのLogin機能の実装方法〜MyMemo](https://qiita.com/daisu_yamazaki/items/a914a16ca1640334d7a5)  
