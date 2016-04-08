# Nagios-install 
2016-02-15 開始撰寫。

目的: Install Nagios core 4.1.1 on Ubuntu 14.04.3 LTS  
流程:  
環境: VirtualBox 4.3.18, Vagrant 1.7.2, Ubuntu 14.04.3 LTS, Java 1.7.0_80-64bit, MongoDB 3.0.8  
軟體: nagios-core 4.1.1, nagios-plugins 2.1.1  
成果:  
附檔:  
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
監控遠端 services時,  
nagios-server (伺服器端 = 監控端)會安裝 : nagios4, nagios-plugins, nagios-nrpe-plugin  
nagios-client (用戶端 = 被監控端)會安裝 : nagios-plugins, nagios-nrpe-server  
註：core-4 好像沒有下載 xinetd 服務 (用來交握)  

安裝方式  (注意 : 預設安裝路徑可能不同)   
1. 下載編譯後安裝(手動)  
2. 套件安裝(apt-get)  

>
(apt-get 沒有看到 nagios4)  
nagios3                    
nagios3-cgi                
nagios3-common             
nagios3-core
nagios3-dbg 
nagios3-doc
nagios-images     
nagios-nrpe-plugin
nagios-nrpe-server
nagios-plugin-check-multi  
nagios-plugins             
nagios-plugins-basic
nagios-plugins-common          
nagios-plugins-contrib     
nagios-plugins-extra  
nagios-plugins-openstack        
nagios-plugins-standard
nagios-snmp-plugins  


* #### Nagios server  
Operating system : Ubuntu 14.04.3 Server  
Description : 筆電的 VM (自行建立)  
IP Address : 192.168.1.103  
Hostname : nagios-server    


* #### Nagios client  
Operating System : Ubuntu 14.04.3 Server  
Description : 筆電的 VM (自行建立)  
IP Address : 192.168.1.104  
Hostname : nagios-client  


## Prerequisites  (nagios-server)  
nagios-server 需要安裝 LAMP ，否則會出錯。(有參考其他的blog)
```
sudo apt-get update
sudo apt-get -y install wget build-essential apache2 apache2-utils unzip php5 openssl perl make php5-gd wget libgd2-xpm-dev libapache2-mod-php5 libperl-dev libssl-dev daemon 
```

## Create Nagios User And Group  (nagios-server)  
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

## Download Nagios And Plugins  (nagios-server)  
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

## Install Nagios And Plugin  (nagios-server)  

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

## Configure Nagios  (nagios-server)  
(可選) Nagios 的配置文件樣本，位於 /usr/local/nagios/etc 目錄下。 這些配置文件應該可以正常工作了。
然而，你可以設定 /usr/local/nagios/etc/objects/contacts.cfg 裡面的 email address，方便接收到示警。  

      sudo nano /usr/local/nagios/etc/objects/contacts.cfg


(可選) 若想限制登入nagios console的 IP，可以在 /etc/apache2/sites-enabled/nagios.conf 裡面設定

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

## Access Nagios Web Interface    (nagios-server)  
打開瀏覽器，並導向到 http://[nagios-server-ip]/nagios  
帳號 : nagiosadmin  
密碼 : (之前步驟所建立的)  

* 登入 nagios UI  
![Image1.png](https://github.com/sethest/Nagios-mongo/blob/master/Nagios-install/Image1.png "帳號 nagiosadmin")
  
* 登入畫面 (可以點進選單 Hosts、Services 看看)
![Image2.png](https://github.com/sethest/Nagios-mongo/blob/master/Nagios-install/Image2.png "nagios UI")



## Add Monitoring targets to Nagios server  (nagios-client)  
加入被監控對象 nagios-client  
nagios-client 需要安裝 nrpe and nagios-plugins (這裡用 apt-get 安裝，而非下載後編譯安裝。)

```
sudo apt-get update
sudo apt-get -y install nagios-nrpe-server nagios-plugins
```

## Configure Monitoring targets  (nagios-client)  

在 nrpe.cfg 裡，設定 nagios-server的IP。

    sudo nano /etc/nagios/nrpe.cfg


>
```
[...]
## Find the following line and add the Nagios server IP ##
allowed_hosts=127.0.0.1 192.168.1.103
[...]
```

啟動 nrpe

    sudo /etc/init.d/nagios-nrpe-server restart
    
    
    
## Configure Monitoring targets (nagios-server)  

讓 server目錄下的 cfg ，都是會被 nagios-server 監控的名單。

    sudo mkdir /usr/local/nagios/etc/servers    
    sudo nano /usr/local/nagios/etc/nagios.cfg
>
    ## Find and uncomment the following line ##
    cfg_dir=/usr/local/nagios/etc/servers

新增 clients.cfg (檔名可換)

    sudo nano /usr/local/nagios/etc/servers/clients.cfg
>
    define host{
    use                             linux-server
    host_name                       nagios-client
    alias                           server
    address                         192.168.1.104
    max_check_attempts              5
    check_period                    24x7
    notification_interval           30
    notification_period             24x7
    }

重啟 nagios

    sudo service nagios restart

host_name 是顯示在 UI 的名字，所以也可以命名成 client-001。  

* 透過 UI 可以發現 Hosts 選單，多了一台 nagios-client。
![Image3.png](https://github.com/sethest/Nagios-mongo/blob/master/Nagios-install/Image3.png "Hosts")

## Define services   (nagios-server)  
設定 nagios-client 要被監控的服務。
假設是 SSHappend 下列內容到 clients.cfg。

    sudo nano /usr/local/nagios/etc/servers/clients.cfg
>
```
define service {                                    
use                             generic-service     
host_name                       nagios-client
service_description             SSH                 
check_command                   check_ssh           
notifications_enabled           0                   
}                                                   
```

重啟 nagios

      sudo service nagios restart

* 透過 UI 可以發現 Services選單，nagios-client 多了一個 SSH 的監控服務。
![Image4.png](https://github.com/sethest/Nagios-mongo/blob/master/Nagios-install/Image4.png "Services")

PENDING 狀態，預計要等 90秒，才會改成 OK 狀態。

* 加快狀態確認 (Re-schedule the next check of this service) & 啟動通知 (Enable notifications for this service) 
![Image5.png](https://github.com/sethest/Nagios-mongo/blob/master/Nagios-install/Image5.png "Speed")
進入後點擊 Commit

* 最後呈現
![Image6.png](https://github.com/sethest/Nagios-mongo/blob/master/Nagios-install/Image6.png "Final")


## Install nrpe-plugin manually  (nagios-server)
若沒有安裝 nrpe-plugin，會導致沒有 check_nrpe 執行檔  
比較不同安裝方式的執行檔路徑  

* 手動編譯安裝
```
wget http://sourceforge.net/projects/nagios/files/nrpe-2.x/nrpe-2.15/nrpe-2.15.tar.gz
tar zxvf nrpe-2.15.tar.gz
rm nrpe-2.15.tar.gz
cd nrpe-2.15
sudo apt-get install libssl-dev
./configure --prefix=/usr/local/nagios/ --with-ssl=/usr/bin/openssl --with-ssl-lib=/usr/lib/x86_64-linux-gnu   
make all
sudo make install-plugin
```

* apt-get 安裝
```
      sudo apt-get install nagios-nrpe-plugin
```

執行檔路徑   
1. 手動編譯安裝 /usr/local/nagios/libexec/check_nrpe  
2. apt-get 安裝 /usr/lib/nagios/plugins/check_nrpe  

## Reference
http://www.unixmen.com/how-to-install-nagios-core-4-1-1-in-ubuntu-15-10/  (英文 blog)  
http://sites.box293.com/nagios/guides/installing-nagios/4-0-x/ubuntu-14-04       
http://askubuntu.com/questions/329323/problem-with-restarting-apache2  

