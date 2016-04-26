# CentOS7.1xを用いてWordPress環境を構築する

## Step 1：WordPressのインストール

　以下のコマンドを入力します。

	cd /vagrant/html/
	sudo yum -y install wget
	sudo wget https://ja.wordpress.org/wordpress-4.4.2-ja.zip
	sudo yum -y install unzip
	sudo unzip -q wordpress-4.4.2-ja.zip
	sudo chown -R apache:apache /var/www/html/wordpress

　次にPHPの設定を変更します。

	sudo vi /etc/php.ini

　このコマンドを入力後、「extension=msql.so」と書かれた場所のコメントを外します（「;」を削除する）。設定変更後、httpdの再起動を行います。

	sudo service httpd restart

## Step 2：ブラウザでのWordPress設定
　以下のURLにアクセスします。

	http://192.168.33.10/wordpress/wp-admin/index.php

　説明に従い設定を行ってください。もしブラウザ経由で設定ができない場合、作成した作業フォルダの中に「wordpress」というフォルダがありますので、そのフォルダを開いてください。
　その中に「wp-config-sample.php」というファイルがあるので、複製して「wp-config.php」に変更して、メモ帳やVi等で開いてください。以下記述を参考に、wp-config.phpの内容を変更してください。

	/** WordPress のためのデータベース名 */
	define('DB_NAME', 'wordpress_db');
	/** MySQL データベースのユーザー名 */
	define('DB_USER', 'wpadmin');
	/** MySQL データベースのパスワード */
	define('DB_PASSWORD', 'データベースに設定したパスワード');
	/** MySQL のホスト名 */
	define('DB_HOST', 'localhost');
	/** データベースのテーブルを作成する際のデータベースの文字セット */
	define('DB_CHARSET', 'utf8');
	/** データベースの照合順序 (ほとんどの場合変更する必要はありません) */
	define('DB_COLLATE', '');


　上記のすぐ下辺りに、以下の3つの記述を新規追加します。

	/** 本家にリダイレクトさせないようにするため追加 */
	define ('WP_SITEURL', 'http://192.168.33.10/wordpress/');
	define ('WP_HOME', 'http://192.168.33.10/wordpress/');
	
	/** プラグインのインストールを行うために追加 */
	define ('FS_METHOD', 'direct');



　また、[こちら（https://api.wordpress.org/secret-key/1.1/salt/）](https://api.wordpress.org/secret-key/1.1/salt/ "秘密鍵サービス")にある[WordPress.orgの秘密鍵サービス](WordPress.org "secret")を用いて、次の項目を変更してください。

	define('AUTH_KEY',         'put your unique phrase here');
	define('SECURE_AUTH_KEY',  'put your unique phrase here');
	define('LOGGED_IN_KEY',    'put your unique phrase here');
	define('NONCE_KEY',        'put your unique phrase here');
	define('AUTH_SALT',        'put your unique phrase here');
	define('SECURE_AUTH_SALT', 'put your unique phrase here');
	define('LOGGED_IN_SALT',   'put your unique phrase here');
	define('NONCE_SALT',       'put your unique phrase here');


## WordPressのセキュリティについて

https://wpdocs.osdn.jp/WordPress_の安全性を高める


## 色々なツール
nmap：ポート確認など  
iLogScanner：ウェブサイトの攻撃兆候検出ツール  
ApacheLogViewer：Apache（Webアクセス）のログビューア  
wireshark：パケットキャプチャソフトウェア  
visitors：Mac/Linux用のApacheLogViewer
