# Nagios-install 
2016-02-15 開始撰寫。

目的: Install Nagios core 4.1.1 on Ubuntu 14.04.3 LTS  
流程:  
環境: VirtualBox 4.3.18, Vagrant 1.7.2, Ubuntu 14.04.3 LTS, Java 1.7.0_80-64bit, MongoDB 3.0.8  
軟體: nagios-core 4.1.1, nagios-plugins 2.1.1  
成果:  
附檔:  
參考: http://sites.box293.com/nagios/guides/installing-nagios/4-0-x/ubuntu-14-04    (英文教學)   
註 : "環境" 的安裝步驟不在此贅述，請自行安裝 。 本範例 ~ 的路徑為: /home/vagrant  

## About Nagios  
Nagios是一個企業級的開源軟件，可用於監控網絡和硬體設施。  
使用Nagios，我們可以監控 servers, switches, applications and services。  
當系統出問題或狀態恢復正常，都可以告知管理者。  

## Features  
使用 Nagios，可以：
- 監控全部的 IT 設備
- 事先發現問題
- 問題發生時立即知道
- 分享可用資訊給相關人員
- 檢測安全漏洞
- 設備升級的參考依據
- 減少停機造成的損失

## Scenario  
遠端監控 services時,  
nagios-server (伺服器端 = 監控端)會安裝 : nagios-core, nagios-plugins, NRPE-plugins  
nagios-client (用戶端 = 被監控端)會安裝 : nagios-plugins, NRPE  
註：core-4 好像沒有下載 xinetd 服務 (用來交握)  

#### Nagios server  
Operating system : Ubuntu 14.04.3 Server
Description : 筆電的 VM (參考 nagios01/Vagrantfile)
IP Address : 192.168.1.103
Hostname : nagios-server  


#### Nagios client (安裝 MongoDB )
Operating System : Ubuntu 14.04.3 Server
Description : 筆電的 VM (參考 nagios02/Vagrantfile)
IP Address : 192.168.1.104
Hostname : nagios-client


## Prerequisites   
nagios-server 需要安裝 LAMP ，否則會出錯。(有參考其他的blog)
```
sudo apt-get update
sudo apt-get -y install wget build-essential apache2 apache2-utils unzip php5 openssl perl make php5-gd wget libgd2-xpm-dev libapache2-mod-php5 libperl-dev libssl-dev daemon 
```

## Create Nagios User And Group  
新增一個 nagios 使用者
```
sudo useradd -m nagios
sudo passwd nagios
```
新增一個 nagcmd 群組，讓外部指令可以透過 web 介面提交。
將 nagios 和 apache 使用者，加入此群組。
```
sudo groupadd nagcmd
sudo usermod -a -G nagcmd nagios
sudo usermod -a -G nagcmd www-data
```

## Download Nagios And Plugins
到 http://sourceforge.net/projects/nagios/files/  下載最新的 nagios-core  
```
cd ~
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.1.1.tar.gz
```

到 http://nagios-plugins.org/download/   下載最新的 nagios-plugin  
Nagios plugins 讓你透過 Nagios 監控 hosts, devices, services, protocols, and applications
```
cd ~
wget http://www.nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz
```

## Install Nagios And Plugin

Install nagios
```
cd ~
tar xzf nagios-4.1.1.tar.gz
rm nagios-4.1.1.tar.gz
cd ~/nagios-4.1.1/
sudo ./configure --with-command-group=nagcmd
sudo make all
sudo make install
sudo make install-init
sudo make install-config
sudo make install-commandmode
```   

Install Nagios Web interface
```
cd ~/nagios-4.1.1/
sudo /usr/bin/install -c -m 644 sample-config/httpd.conf /etc/apache2/sites-enabled/nagios.conf
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

Check if nagios.conf is placed in /etc/apache2/sites-enabled directory.
```
sudo ls -l /etc/apache2/sites-enabled/    
```

Restart apache web server
```
echo "ServerName localhost"  | sudo tee -a /etc/apache2/apache2.conf
sudo service apache2 restart
```

Install Nagios plugins
```
cd ~
tar xzf nagios-plugins-2.1.1.tar.gz
rm nagios-plugins-2.1.1.tar.gz
cd nagios-plugins-2.1.1/
sudo ./configure --with-nagios-user=nagios --with-nagios-group=nagios
sudo make
sudo make install
```

# Configure Nagios  
Nagios 的配置文件樣本，位於 /usr/local/nagios/etc 目錄下。 這些配置文件應該可以正常工作了。
然而，你可以設定 /usr/local/nagios/etc/objects/contacts.cfg 裡面的 email address，方便接收到示警。  

      sudo nano /usr/local/nagios/etc/objects/contacts.cfg


若想限制登入nagios console的 IP，可以在 /etc/apache2/sites-enabled/nagios.conf 裡面設定

      sudo nano /etc/apache2/sites-enabled/nagios.conf

>
```
## Comment the following lines ##
#   Order allow,deny
#   Allow from all
## Uncomment and Change lines as shown below ##
Order deny,allow
Deny from all
Allow from 127.0.0.1 192.168.1.103 192.168.1.104
```


Enable Apache’s rewrite and cgi modules:

      sudo a2enmod rewrite
      sudo a2enmod cgi

重啟 apache2

      sudo service apache2 restart
   
檢查 nagios.cfg 有沒有語法錯誤

      sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg   

如果沒有錯誤, 啟動 nagios 服務，並設成登入自動啟動。

      sudo service nagios start
      sudo ln -s /etc/init.d/nagios /etc/rcS.d/S99nagios

# Access Nagios Web Interface
打開瀏覽器，並導向到 http://[nagios-server-ip]/nagios  
帳號 : nagiosadmin  
密碼 : (之前步驟所建立的)  

