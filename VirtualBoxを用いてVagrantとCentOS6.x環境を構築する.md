# VirtualBoxを用いてVagrantとCentOS6.x環境を構築する

　講習会用の環境を、お手軽に手元で再現できるように情報を整理しました。構築方法が少々高度なので、わからない方は原口の方まで質問してください。また、かなり荒い（特にFireWall周り。セキュリティ的には危険な）設定を施していますので、常用する場合は適切な設定を行ってください。

## 用意するもの（OSの標準装備のアプリケーションは割愛）
Windows：VirtualBox, Vagrant, TeraTerm  
Mac：Homebrew, VirtualBox, Vagrant


## 用語解説

VirtualBox：OS（ホスト、自分のパソコン）内で仮想的にOS環境を構築するためのソフトウェア。  
Vagrant：開発環境の構築と共有を簡単に行うためのツール。本番環境での投入前に実験するための環境構築用ツールとして重宝される。  
TeraTerm：Windows向けのターミナルソフト。  
Homebrew：Mac用のパッケージ管理ソフトウェア（Linuxで言う所のyumやrpmといったもの）。

## 手順（コマンドのみ列挙）

### Windows用初期設定
* VirtualBoxをインストールする。
	* https://www.virtualbox.org/wiki/Downloads
* Vagrantをインストールする。
	* https://www.vagrantup.com/downloads.html
* TeraTermをインストールする。
	* https://osdn.jp/projects/ttssh2/releases/
* 仮想環境作成用に、フォルダを用意します。
	* 特にこだわりがなければ、Documentフォルダに「Vagrant」というフォルダを作成してください。
* コマンドプロンプトで、作業フォルダへ移動する。
	* cd Document/Vagrant

### Mac用初期設定
* Xcodeをインストールする。
	* Mac App Storeでダウンロードできます。
	* インストールしたら、一度起動して、ライセンスに同意してください。
	* その後、Macのターミナルで以下のコマンドを念のため入力してください。

#
	sudo xcodebuild -license
	xcode-select --install


* RubyGemのアップデート

#
	以下のコマンドをターミナルで入力する際に、Macにログインした時のパスワードが必要になります。
	
	sudo gem install rubygems-update
	sudo update_rubygems && sudo gem update && sudo gem clean

* Homebrewをインストールする。

#
	ターミナルで、以下のコマンドを入力します。
	
	ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
	brew doctor
	brew -v
	brew update && brew upgrade && brew cleanup
	brew tap caskroom/cask

* VirtualBoxとVagrantのインストール

# 
	ターミナルで、以下のコマンドを入力します。
	
	brew cask install virtualbox vagrant

* 作業用フォルダを用意する。

# 
	特に好みの場所がなければ、ターミナルで以下のコマンドを入力してください。
	
	mkdir /Users/${USER}/Vagrant
	cd /Users/${USER}/Vagrant

### 共通手順

##### Step 1：コマンドプロンプトまたはターミナルで、以下のコマンドを入力します。

	vagrant box add CentOS65_64 https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box
	vagrant box list

##### Step 2：「CentOS65_64」という名前のBOXができていたら、以下のコマンドを入力します。

	vagrant init CentOS65_64

　vagrantのイニシャライズ（vagrant init CentOS65_64）が完了すると、先ほど作業用に作成したフォルダ内に「Vagrantfile」というファイルができています。

##### Step 3：vagrantに関連するプラグインをインストールす。

vagrant plugin install vagrant-hostsupdater  
vagrant plugin install vagrant-vbguest  

　Windowsの場合は、TeraTerm用に以下のプラグインもインストールします。

	vagrant plugin install vagrant-teraterm  

##### Step 4：Vagrantファイルの設定を書き換える。

　Vagrantfileをメモ帳やVi等のテキストエディタで開き、設定を変更します。

	Vagrant.configure(2) do |config|
	# The most common configuration options are documented and commented below.
	# For a complete reference, please see the online documentation at
	# https://docs.vagrantup.com.

上記文言のすぐ下に、次の文字列を追加します。

	config.vbguest.auto_update = false

次の項目のコメントを外します。「host: 8080」となっている箇所については、番号を「1234」に変えてください。また、「"../data", "/vagrant_data"」となっている箇所を、「".", "/vagrant"」に変更します。

	config.vm.network "forwarded_port", guest: 80, host: 8080
	config.vm.network "private_network", ip: "192.168.33.10"
	config.vm.synced_folder ".", "/vagrant", owner: 'apache', group: 'apache', mount_options: ['dmode=755', 'fmode=644']

##### Step 5：CentOSにログインする。

　コマンドプロンプトまたはターミナルで、以下のコマンドを入力します。

	vagrant up
	vagrant ssh

　Windowsの場合は、このあと次のコマンドも入力してください。

	vagrant teraterm

　これでCentOSにログインができます。

##### Step 6：パッケージ管理ソフトウェアを用いてバージョンアップする。

	yum -y check-update
	sudo yum -y update --exclude=kernel* --exclude=centos*

　アップデートのオプションで「 --exclude=kernel* --exclude=centos*」をつけているのは、講習会環境のCentOS6.5から最新版のOSとカーネルを変更させないためです。上記アップデート後、CentOS6.5からバージョンの変更がないか、以下のコマンドを用いて確認してください。

	cat /etc/issue

##### Step 7：言語設定の確認をする。

	locale

　このコマンドで、CentOSの言語設定状態が確認できます。システムのフォントを確認するためには、以下のコマンドを入力します。

	cat /etc/sysconfig/i18n

　デフォルトだと「en_US.UTF8」となっていると思います。これだと日本語対応していないので、日本語用パッケージの追加と変更を行います。

	sudo yum -y groupinstall "Japanese Support"
	sudo localedef -f UTF-8 -i ja_JP ja_JP.utf8
	sudo vi /etc/sysconfig/i18n

　Viで設定変更が可能になりますので、「en_US.UTF8」を「ja_JP.UTF8」に変更してください。変更したら、反映しているか確認するためにdateコマンドを打ち込んで、日本の日付で表示されているか確認してください。

##### Step 8：Apacheのインストール

	yum search all apache | grep httpd
	yum list | grep httpd

　Apache（httpd）の提供されているバージョンを確認していますが、ここでは標準のバージョンを用います。

	sudo yum -y install httpd.x86_64

　インストールしたら、httpdの（一時）起動、常時起動の設定と、FireWall（iptables）の設定を（一時）停止、常時停止の設定を行います。

	sudo service httpd start
	sudo chkconfig httpd on
	sudo service iptables stop
	sudo chkconfig iptables off

##### Step 9：ホスト（自分のコンピュータ）とゲスト（CentOS）のフォルダ同期設定

　ゲスト側で作成したHTMLファイル等をホスト側で編集可能にするために、フォルダの同期設定を行います。

	mkdir /vagrant/html/
	sudo mv /var/www/html/ /var/www/html.old/
	sudo ln -fs /vagrant/html/ /var/www/

##### Step 10：PHPとMySQLのインストール

　PHPとMySQLをインストールします。その際、WordPressに適切なバージョンであるかを確認してください。もし適切なバージョンでない場合は、バージョン指定してそれぞれのソフトウェアをインストールする必要があります。

	sudo yum -y install php-mysql php php-gd php-	mbstring php-common php-cli php-pear php-xml
	php --version
	
	sudo yum -y install mysql mysql-server
	mysql --version
	sudo service mysqld start
	sudo chkconfig mysqld on

##### Step 11：ブラウザでの動作確認。

　ホスト側のブラウザで動作確認を行います。以下のURLにアクセスしてください。

	http://192.168.33.10

　Apacheの初期設定画面が表示されたら成功です。

##### Step 12：MySQLの設定

　初期設定ではrootにパスワードがかかっていないので、パスワード設定を行います。以下のコマンドを入力したら、指示に従って設定を行ってください。最初のパスワードを聞かれたら、エンターキーを押してください（パスワードがかかっていないので、何も入力する必要なし）。

	sudo mysql_secure_installation

　設定が終わったら、以下のコマンドを入力してMySQLにrootでログインします。その際、使用するパスワードは先ほど設定したものとなります。

	mysql -u root -p

　次に以下のコマンドを入力して、WordPress用のアカウントとデータベースを作成します。

	select user,host from mysql.user;
	show databases;
	create database wordpress_db;
	grant all privileges on wordpress_db.* to wpadmin@localhost identified by '任意のパスワード';
	flush privileges;
	exit

##### Step 13：phpMyAdminのインストール

	sudo yum -y install phpmyadmin


　phpMyAdminのインストールが完了したら、設定ファイルを修正します。以下のコマンドを入力後、以下の情報を見比べて記載内容を修正してください。  
　**1つ目：phpMyAdmin.confの設定**

	sudo vi /etc/httpd/conf.d/phpMyAdmin.conf
	
	   <IfModule mod_authz_core.c>
	     # Apache 2.4
	     <RequireAny>
	       Require ip 127.0.0.1
	       Require ip ::1
	     </RequireAny>
	   </IfModule>
	   <IfModule !mod_authz_core.c>
	     # Apache 2.2
	     Order Deny,Allow
	     Deny from All
	     Allow from 127.0.0.1
	     Allow from All
	   </IfModule>
	</Directory>


　**2つ目：config.inc.phpの設定**

	$cfg['Servers'][$i]['AllowNoPassword']              // Allow logins without a password. Do not change the FALSE
                                     = FALSE;

　このFALSEの箇所を、TRUEに変えてください。

　**3つ目：httpd.confの設定**

	EnableSendfile off　のコメント（;）を外す。
	「AllowOverride None」の「None」を「All」にする。
	　-> 全ての箇所を変更してください。

　上記設定が終わったら、以下のコマンドを入力してください。

	sudo service httpd restart

##### Step 15：phpMyAdminの起動確認
　
　以下のURLにアクセスして、MySQLで作成したアカウントとパスワードを用いてログインしてください。ログインできたら成功です。

	http://192.168.33.10/phpmyadmin/

