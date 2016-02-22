# Nagios-mongo

目的: 如何使用 Nagios的 check_mongodb Python Plugin 監控 MongoDB   
流程:  
環境: 延續 Nagios-install 目錄的內容  
軟體: nagios-plugin-mongodb (Github)  
成果:  
附檔:  
註 : "環境" 的安裝步驟不在此贅述，請自行安裝 。 本範例 ~ 的路徑為: /home/vagrant

當你在生產環境運行 MongoDB ，有必要監控以確保其啟動與運行。  

當你的環境已經使用 Nagios，應優先考慮使用此套件來監控。  

chech_mongodb 是由 Python 撰寫的 Nagios套件。  

以下將解釋如何透過此套件，如何安裝、配置和監控 MongoDB。  


## Download check_mongodb Nagios Plugin (nagios-server)  

套件安裝的位置，可分成伺服器端和用戶端。

伺服器端 :     
blog是把 nagios-plugin-mongodb安裝在伺服器端，  
所以 check_mongodb 要在伺服器端 commands.cfg 做設定。   

用戶端 :   
書本是把 nagios-plugin-mongodb 安裝在用戶端，  
所以 check_mongodb 要在用戶端 nrpe.cfg 做設定。  

另外，套件放在 Github，下載方式可分成  wget 和 git clone。  

本範例採用 "版本一" 和 "wget"  

套件下載網址  https://github.com/mzupan/nagios-plugin-mongodb

```
cd ~
wget --no-check-certificate https://github.com/mzupan/nagios-plugin-mongodb/archive/master.zip
unzip master.zip
rm master.zip
mv nagios-plugin-mongodb-master nagios-plugin-mongodb
```

## Install Plugin in Libexec directory  (nagios-server)
將 "nagios-plugin-mongodb" 目錄移到 nagios 的 libexec 目錄。(所有 plugins 存放的位置)  

    cd ~
    sudo mv nagios-plugin-mongodb /usr/local/nagios/libexec

修改此目錄的權限

    cd /usr/local/nagios/libexec/
    sudo chown -R nagios:nagios nagios-plugin-mongodb/

## Download and Install Mongo Python Driver  
使用 pip 方法安裝 pymongo

    sudo apt-get -y install python-pip
    sudo pip install pymongo==2.8

不要使用 sudo pip install ~~requirements~~
pymongo 不要安裝 3.x 否則會有一堆服務有錯誤訊息 "CRITICAL - General MongoDB Error: can't set attribute"

## Test MongoDB Nagios Plugin from Command Line  (nagios-server)
測試套件 nagios-mongodb 的執行檔 check_mongodb.py 
預設檢查 local:27017 mongod 的連線狀態。

    cd /usr/local/nagios/libexec/nagios-plugin-mongodb/
    ./check_mongodb.py   ( ./check_mongodb.py -H 127.0.0.1 -P 27017 -A connect )
>
```
OK - Connection took 0 seconds
或
CRITICAL - General MongoDB Error: could not connect to 127.0.0.1:27017: (111, 'Connection refused')
```

查詢 check_mongodb.py 的幫助

    cd /usr/local/nagios/libexec/nagios-plugin-mongodb/
    ./check_mongodb.py --help

參數解釋
```
-H                  default='127.0.0.1'     host                                          
-P                  default=27017           port                                          
-u                                          username    
-p                                          passwd    
-W                                          warning    
-C                                          critical    
-A                  default=connect         action                                        
--max-lag                                   複製的延遲上限 (前提:要做RS)    
--mapped-memory                             用"映射記憶體"取代"常駐記憶體" (前提: 常駐記憶體不可讀)
-D                                          開啟輸出效能數據(畫圖用)    
-d                  default='admin'         指定DB                                        
--all-databases                             檢查所有DB(-A database_size)    
-s                                          用ssl安全協議連接    
-r                                          連接到RS
-q                  default='query'         用 (-A queries_per_second)來確認查詢的型態[query|insert|update|delete|getmore|command]   
-c                                          指定collection     default='admin'
-T                                          花費時間去檢查錯誤的頁數     default=1
```

-A    
動作解釋(default=connect) and 會檢查的參數
```
asserts                     ( -H, -W, -C, -D)                                 斷言
chunks_balance              ( -d, -c, -W, -C)                                 塊平衡
collection_indexes          ( -d, -c, -W, -C, -D)                             指定集合的索引大小         
collection_state            ( -d, -c)                                         指定集合的狀態         
collections                 ( -W, -C, -D)                                     集合數         
connect                     ( -H, -P, -W, -C, -D, -u, -p, conn_time)          連線         
connect_primary             ( -W, -C, -D)                                     連結到主節點         
connections                 ( -W, -C, -D)                                     連線百分比         
current_lock                ( -H, -W, -C, -D)                                 當前鎖住
database_indexes            ( -d, -W, -C, -D)                                 指定資料庫的索引大小         
database_size               (全部DB: -W, -C, -D)(指定DB: -d, -W, -C, -D)      指定資料庫的大小         
databases                   ( -W, -C, -D)                                     資料庫數         
flushing                    ( -W, -C, True, -D)                               刷新映射函數mmp的平均時間,單位是秒         
index_miss_ratio            ( -W, -C, -D)                                     索引的錯誤比例         
journal_commits_in_wl       ( -W, -C, -D)                                     日誌提交
journaled                   ( -W, -C, -D)                                     日誌
last_flush_time             ( -W, -C, False, -D)                              刷新映射函數mmp的平均時間,單位是秒         
lock                        ( -W, -C, -D)                                     鎖住時間百分比         
memory                      ( -W, -C, -D, options.mapped_memory)              記憶體使用         
memory_mapped               ( -W, -C, -D)                                     映射記憶體使用         
opcounters                  ( -H, -W, -C, -D)                                 操作日誌計算
oplog                       ( -W, -C, -D)                                     操作日誌
page_faults                 ( sample_time, -W, -C, -D)                        錯誤頁數
queries_per_second          ( query_type, -W, -C, -D)                         每一次節點的數量         
queues                      ( -W, -C, -D)                                     隊列              
replica_primary             ( -H, -W, -C, -D, replicaset)                     複製集的主節點狀態         
replication_lag             ( -H, -P, -W, -C, False, -D, --max_lag, -u, -p)   複製延遲的時間,單位是秒         
replication_lag_percent     ( -H, -P, -W, -C, True, -D, --max_lag, -u, -p)    複製延遲的百分比         
replset_quorum              ( -D)                                             複製集對列              
replset_state               ( -D, -W, -C)                                     複製集狀態         
row_count                   ( -d, -c, -W, -C, -D)                             資料筆數
write_data_files            ( -W, -C, -D)                                     寫資料目錄
```

## Add check_mongodb Command Definition
設定 command.cfg

    sudo nano /usr/local/nagios/etc/objects/commands.cfg

新增下列五種 nagios-mongo CMD  (待修)
```
define command {
command_name check_mongodb
command_line $USER1$/nagios-plugin-mongodb/check_mongodb.py -H $HOSTADDRESS$ -A $ARG1$ -P $ARG2$ -W $ARG3$ -C $ARG4$
}

define command {
command_name check_mongodb_database_size
command_line $USER1$/nagios-plugin-mongodb/check_mongodb.py -H $HOSTADDRESS$ -A $ARG1$ -P $ARG2$ -W $ARG3$ -C $ARG4$ -d $ARG5$
}

define command {
command_name check_mongodb_collection_size
command_line $USER1$/nagios-plugin-mongodb/check_mongodb.py -H $HOSTADDRESS$ -A $ARG1$ -P $ARG2$ -W $ARG3$ -C $ARG4$ -d $ARG5$ -c $ARG6$
}

define command {
command_name check_mongodb_replicaset
command_line $USER1$/nagios-plugin-mongodb/check_mongodb.py -H $HOSTADDRESS$ -A $ARG1$ -P $ARG2$ -W $ARG3$ -C $ARG4$ -r $ARG5$
}

define command {
command_name check_mongodb_query
command_line $USER1$/nagios-plugin-mongodb/check_mongodb.py -H $HOSTADDRESS$ -A $ARG1$ -P $ARG2$ -W $ARG3$ -C $ARG4$ -q $ARG5$
}
```

## Create Nagios Service Definition for MongoDB connect
延續 sethest/Nagios-mongo/Nagios-install 的配置

設定 clients.cfg

    sudo nano /usr/local/nagios/etc/servers/clients.cfg

```
define service {
        use                     generic-service
        host_name               nagios-client
        service_description     check_mongodb
        check_command           check_mongodb!connect!27017!2!4
}
define service {
        use                     generic-service
        host_name               nagios-client
        service_description     check_mongodb_database_size
        check_command           check_mongodb_database!database_size!1024!2048!test
}
define service {
        use                     generic-service
        host_name               nagios-client
        service_description     check_mongodb_collection
        check_command           check_mongodb_collection!
}
define service {
        use                     generic-service
        host_name               nagios-client
        service_description     check_mongodb_replicaset
        check_command           check_mongodb_replicaset!
}
define service {
        use                     generic-service
        host_name               nagios-client
        service_description     check_mongodb_query
        check_command           check_mongodb_query!
}
```

## Reference
http://www.thegeekstuff.com/2013/10/nagios-check-mongodb-plugin/
https://github.com/mzupan/nagios-plugin-mongodb
