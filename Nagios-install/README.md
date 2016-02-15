# Nagios-install 
2016-02-15 開始撰寫。

目的: Install Nagios core 4.1.1 on Ubuntu 14.04.3 LTS  
流程:  
環境: VirtualBox 4.3.18, Vagrant 1.7.2, Ubuntu 14.04.3 LTS, Java 1.7.0_80-64bit, MongoDB 3.0.8  
軟體: Nagios-core 4.1.1  
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

1. 確定你的 nagios-server 安裝了 LAMP 
2.   

    sss
    sudo apt-get -y install build-essential libgd2-xpm-dev apache2-utils unzip
    sudo apt-get -y install wget build-essential apache2 php5 openssl perl make php5-gd wget libgd2-xpm-dev libapache2-mod-php5 libperl-dev libssl-dev daemon  #(參考其他人的 blog ， 因為原本的沒有安裝 apache2 ，會出錯。)








http://sourceforge.net/projects/nagios/files/   (下載 Nagios-core)
http://nagios-plugins.org/download/             (下載 Nagios-plugin)
http://sites.box293.com/nagios/guides/installing-nagios/4-0-x/ubuntu-14-04    (英文教學) 
