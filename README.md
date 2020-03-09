kafka version: kafka_2.11-2.2.0 <br />
zookeeper version: zookeeper-3.4.14 <br />

Kafka是一種高吞吐量的分佈式發布訂閱消息系統,我們先了解一下Kafka中涉及的相關術語：<br />  

1、Broker——Kafka集群包含一個或多個服務器，這種服務器被稱為broker;<br />
2、Topic——每條發佈到Kafka集群的消息都有一個類別，這個類別被稱為Topic.<br />
3、Partition——Partition是物理上的概念，每個Topic包含一個或多個Partition.<br />
4、Producer——負責發布消息到Kafka broker.<br />
5、Consumer——消息消費者，向Kafka broker讀取消息的客戶端.<br />
6、Consumer Group——每個Consumer屬於一個特定的Consumer Group（可為每個Consumer指定group name，若不指定group name則屬於默認的group. <br />


Kafka架構：Topic、Partition、Producer、Consumer<br />
一個普通的工作流程是Kafka的producer向topic寫入消息，consumer從topic中讀取消息。<br />

Hostname	      IP	<br />
ka-node1	10.205.35.242<br />	
ka-node2	10.205.35.243<br />
ka-node3	10.205.35.244<br />

***
系統環境準備<br />
java環境準備，並需要配置環境變量<br />

***
ZooKeeper集群部署<br />
#創建數據與日誌文件目錄<br />
mkdir -p /data/zookeeper/{data,logs}<br />

wget -P /usr/local/ https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz && tar -zxf zookeeper-3.4.14.tar.gz && mv zookeeper-3.4.14 zookeeper-cluster<br />
#創建用戶與目錄授權<br />
useradd -s /sbin/nologin -M -U zookeeper && chown -R zookeeper:zookeeper /usr/local/zookeeper-cluster /data/zookeeper/ <br />

#配置環境變量 <br />
export ZK_LOGS=/data/zookeeper/logs/ <br />
export ZK_HOME=/usr/local/zookeeper-cluster <br />
export PATH=$ZK_HOME/bin:$PATH <br />


日誌輸出標準化設置 <br />
#zkEnv.sh <br />
將ZOO_LOG_DIR="." <br />
修改為：ZOO_LOG_DIR="$ZK_LOGS" <br />

將ZOO_LOG4J_PROP="INFO,CONSOLE" <br />
修改為：ZOO_LOG4J_PROP="INFO,ROLLINGFILE" <br />

#log4j.properties <br />
將log4j.appender.ROLLINGFILE=org.apache.log4j.RollingFileAppender <br />
修改为：log4j.appender.ROLLINGFILE=org.apache.log4j.DailyRollingFileAppender <br />
 
其它可配置参数： <br />
log4j.appender.ROLLINGFILE.File=zookeeper.log <br />
log4j.appender.ROLLINGFILE.DataPattern='.'yyyy-MM-dd-HH-mm <br />
log4j.appender.ROLLINGFILE.Threshold=debug <br />
log4j.appender.ROLLINGFILE.encoding=UTF-8 <br />
log4j.appender.ROLLINGFILE.Append=false <br />
log4j.appender.ROLLINGFILE.layout=org.apache.log4j.PatternLayout <br />
log4j.appender.ROLLINGFILE.layout.ConversionPattern= [%d{yyyy-MM-dd HH\:mm\:ss}]%-5p %c(line\:%L) %x-%m%n <br />

zookeeper 配置 <br />
所有節點配置文件如下 zoo.cfg <br />

***
#The number of milliseconds of each tick <br />
tickTime=2000 <br />

#The number of ticks that the initial  <br />
#synchronization phase can take <br />
initLimit=10 <br />

#The number of ticks that can pass between  <br />
#sending a request and getting an acknowledgement <br />
syncLimit=5 <br />

#the directory where the snapshot is stored. <br />
#do not use /tmp for storage, /tmp here is just  <br />
#example sakes. <br />

dataDir=/data/zookeeper/data <br />
dataLogDir=/data/zookeeper/data/logs <br />
snapCount=2 <br />

#the port at which the clients will connect <br />
clientPort=2181 <br />
autopurge.snapRetainCount=50 <br />
autopurge.purgeInterval=1 <br />
server.0 = 10.205.35.242:2888:3888 <br />
server.1 = 10.205.35.243:2888:3888 <br />
server.2 = 10.205.35.244:2888:3888 <br />

***

#對應節點主機創建標識文件，標識文件內容對應zoo.cfg中server.x；x標識節點標識號，具有唯一性。 <br />
10.205.35.242節點創建 <br />
echo "0" >/data/zookeeper/data/myid <br />

10.205.35.243節點創建 <br />
echo "1" >/data/zookeeper/data/myid <br />

10.205.35.244節點創建 <br />
echo "2" >/data/zookeeper/data/myid <br />

***
啟動 <br />
./zkServer.sh start <br />

集群啟動成功標識 <br />
./zkServer.sh status <br />
 
ZooKeeper JMX enabled by default <br />
Using config: /usr/local/zookeeper-cluster/bin/../conf/zoo.cfg <br />
Mode: leader<br /> 

***
zookeeper常用命令<br />

<p>
1.連接客戶端 <br />
打開ZooKeeper的文件目錄\bin運行./zkCli.sh。
b.進入到ZooKeeper安裝目錄，運行./bin/zkCli.sh -server 10.205.35.243:2181連接到ZooKeeper客戶端。

2.簡單命令：


*  顯示根目錄下、文件： ls / 使用 ls 命令來查看當前 ZooKeeper 中所包含的內容

*  顯示根目錄下、文件： ls2 / 查看當前節點數據並能看到更新次數等數據

*  創建文件，並設置初始內容： create /zk "test" 創建一個新的 znode節點“ zk ”以及與它關聯的字符串

*  獲取文件內容： get /zk 確認 znode 是否包含我們所創建的字符串

*  修改文件內容： set /zk "zkbak" 對 zk 所關聯的字符串進行設置

*  刪除文件： delete /zk 將剛才創建的 znode 刪除

*  退出客戶端： quit

*  幫助命令： help

</p>

3.Zookeeper的功能描述 <br />

conf <br />
輸出相關服務配置的詳細信息。<br />
cons
列出所有連接到服務器的客戶端的完全的連接會話的詳細信息。包括“接受發送”的包數量、會話 id 、操作延遲、最後的操作執行等等信息。<br />
dump <br />
列出未經處理的會話和臨時節點。 <br />
envi <br />
輸出關於服務環境的詳細信息（區別於 conf 命令）。 <br />
reqs <br />
列出未經處理的請求 <br />
ruok <br />
測試服務是否處於正確狀態。如果確實如此，那麼服務返回“imok ”，否則不做任何相應。 <br />
stat <br />
輸出關於性能和連接的客戶端的列表。 <br />
wchs <br />
列出服務器 watch 的詳細信息。 <br />
wchc <br />
通過 session 列出服務器 watch 的詳細信息，它的輸出是一個與watch 相關的會話的列表。 <br />
wchp <br />
通過路徑列出服務器watch 的詳細信息。它輸出一個與 session相關的路徑。<br />

<p>
使用示例： <br />
可以通過命令： <br />
echo stat | nc 127.0.0.1 2181 來查看哪個節點被選擇作為follower或者leader <br />
echo ruok | nc 127.0.0.1 2181 測試是否啟動了該Server，若回复imok表示已經啟動。 <br />
echo dump | nc 127.0.0.1 2181 ,列出未經處理的會話和臨時節點。 <br />
echo kill | nc 127.0.0.1 2181 ,關掉server <br />
echo conf | nc 127.0.0.1 2181 ,輸出相關服務配置的詳細信息。<br />
echo cons | nc 127.0.0.1 2181 ,列出所有連接到服務器的客戶端的完全的連接會話的詳細信息。<br />
echo envi |nc 127.0.0.1 2181 ,輸出關於服務環境的詳細信息（區別於 conf 命令）。<br />
echo reqs | nc 127.0.0.1 2181 ,列出未經處理的請求。<br />
echo wchs | nc 127.0.0.1 2181 ,列出服務器 watch 的詳細信息。<br />
echo wchc | nc 127.0.0.1 2181 ,通過 session 列出服務器 watch 的詳細信息，它的輸出是一個與 watch 相關的會話的列表。<br />
echo wchp | nc 127.0.0.1 2181 ,通過路徑列出服務器 watch 的詳細信息。它輸出一個與 session 相關的路徑。<br />
</p>

***
zookeeper配置文件說明<br />

*  客戶端連接server的端口，即對外服務端口，一般設置為2181吧 <br />
clientPort <br />


*  存儲快照文件snapshot的目錄。默認情況下，事務日誌也會存儲在這裡。建議同時配置參數dataLogDir, 事務日誌的寫性能直接影響zk性能 <br />
dataDir <br />


*  ZK中的一個時間單元。 ZK中所有時間都是以這個時間單元為基礎，進行整數倍配置的。例如，session的最小超時時間是2*tickTime <br />
tickTime <br />


*  事務日誌輸出目錄。盡量給事務日誌的輸出配置單獨的磁盤或是掛載點，這將極大的提升ZK性能（NoJavasystem property）<br />
dataLogDir <br />


*  最大請求堆積數。默認是1000。 ZK運行的時候， 儘管server已經沒有空閒來處理更多的客戶端請求了，但是還是允許客戶端將請求提交到服務器上來，以提高吞吐性能。當然，為了防止Server內存溢出，這個請求堆積數還是需要限制下的(Java system property:zookeeper.globalOutstandingLimit.) <br />
globalOutstandingLimit <br />


*  預先開闢磁盤空間，用於後續寫入事務日誌。默認是64M，每個事務日誌大小就是64M。如果ZK的快照頻率較​​大的話，建議適當減小這個參數(Java system property:zookeeper.preAllocSize) <br />
preAllocSize <br />


*  每進行snapCount次事務日誌輸出後，觸發一次快照(snapshot), 此時，ZK會生成一個snapshot.*文件，同時創建一個新的事務日誌文件log.*。默認是100000.（真正的代碼實現中，會進行一定的隨機數處理，以避免所有服務器在同一時間進行快照而影響性能）(Java system property:zookeeper.snapCount) <br />
snapCount <br />


*  用於記錄所有請求的log，一般調試過程中可以使用，但是生產環境不建議使用，會嚴重影響性能(Java system property:?requestTraceFile) <br />
traceFile <br />


*  單個客戶端與單台服務器之間的連接數的限制，是ip級別的，默認是60，如果設置為0，那麼表明不作任何限制。請注意這個限制的使用範圍，僅僅是單台客戶端機器與單台ZK服務器之間的連接數限制，不是針對指定客戶端IP，也不是ZK集群的連接數限制，也不是單台ZK對所有客戶端的連接數限制。指定客戶端IP的限制策略（No Java system property）<br />
maxClientCnxns <br />


*  對於多網卡的機器，可以為每個IP指定不同的監聽端口。默認情況是所有IP都監聽clientPort指定的端口 new in 3.3.0 <br />
clientPortAddress<br />


*  Session超時時間限制，如果客戶端設置的超時時間不在這個範圍，那麼會被強制設置為最大或最小時間。默認的Session超時時間是在2 *tickTime ~ 20 * tickTime這個範圍New in 3.3.0 <br />
minSessionTimeoutmaxSessionTimeout <br />


*  事務日誌輸出時，如果調用fsync方法超過指定的超時時間，那麼會在日誌中輸出警告信息。默認是1000ms(Java system property:fsync.warningthresholdms)New in 3.3.4 <br />
fsync.warningthresholdms <br />


*  在上文中已經提到，3.4.0及之後版本，ZK提供了自動清理事務日誌和快照文件的功能，這個參數指定了清理頻率，單位是小時，需要配置一個1或更大的整數，默認是0，表示不開啟自動清理功能(No Java system property)New in 3.4.0 <br />
autopurge.purgeInterval <br />


*  這個參數和上面的參數搭配使用，這個參數指定了需要保留的文件數目。默認是保留3個(No Java system property)New in 3.4.0 <br />
autopurge.snapRetainCount <br />


*  在之前的版本中， 這個參數配置是允許我們選擇leader選舉算法，但是由於在以後的版本中，只會留下一種“TCP-based version of fast leader election”算法，所以這個參數目前看來沒有用了，這裡也不詳細展開說了(No Java system property) <br />
electionAlg <br />


*  Follower在啟動過程中，會從Leader同步所有最新數據，然後確定自己能夠對外服務的起始狀態。 Leader允許F在initLimit時間內完成這個工作。通常情況下，我們不用太在意這個參數的設置。如果ZK集群的數據量確實很大了，F在啟動的時候，從Leader上同步數據的時間也會相應變長，因此在這種情況下，有必要適當調大這個參數了(No Java system property ) <br />
initLimit <br />


*  在運行過程中，Leader負責與ZK集群中所有機器進行通信，例如通過一些心跳檢測機制，來檢測機器的存活狀態。如果L發出心跳包在syncLimit之後，還沒有從F那裡收到響應，那麼就認為這個F已經不在線了。注意：不要把這個參數設置得過大，否則可能會掩蓋一些問題(No Java system property) <br />
syncLimit <br />


*  默認情況下，Leader是會接受客戶端連接，並提供正常的讀寫服務。但是，如果你想讓Leader專注於集群中機器的協調，那麼可以將這個參數設置為no，這樣一來，會大大提高寫操作的性能(Java system property: zookeeper.leaderServes) <br />
leaderServes <br />


*  這裡的x是一個數字，與myid文件中的id是一致的。右邊可以配置兩個端口，第一個端口用於F和L之間的數據同步和其它通信，第二個端口用於Leader選舉過程中投票通信(No Java system property) <br />
server.x=[hostname]:nnnnn[:nnnnn] <br />


*  對機器分組和權重設置，可以查看官網(No Java system property) <br />
group.x=nnnnn[:nnnnn]weight.x=nnnnn <br />


*  Leader選舉過程中，打開一次連接的超時時間，默認是5s(Java system property: zookeeper.cnxTimeout) <br />
cnxTimeout <br />


*  ZK權限設置相關 <br />
zookeeper.DigestAuthenticationProvider.superDigest <br />


*  對所有客戶端請求都不作ACL檢查。如果之前節點上設置有權限限制，一旦服務器上打開這個開頭，那麼也將失效(Java system property:zookeeper.skipACL) <br />
skipACL <br />


*  這個參數確定了是否需要在事務日誌提交的時候調用FileChannel.force來保證數據完全同步到磁盤(Java system property:zookeeper.forceSync) <br />
forceSync <br />


*  每個節點最大數據量，是默認是1M。這個限制必須在server和client端都進行設置才會生效(Java system property:jute.maxbuffer) <br />
jute.maxbuffer

***
Kafka 集群部署 <br />
wget -P /usr/local/ https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.2.0/kafka_2.11-2.2.0.tgz && tar -zxf kafka_2.11-2.2.0.tgz && mv kafka_2.11-2.2.0 kafka-cluster <br />

創建啟動用戶與目錄授權 <br />
useradd -s /sbin/nologin -M -U kafka && chown -R kafka:kafka /usr/local/kafka-cluster <br />

***
kafka集群配置 <br />

/usr/local/kafka-cluster/config/server.properties <br />

broker.id=0 #ID標識號具有唯一性，同zookeeper標識 <br />
listeners=PLAINTEXT://10.205.35.242:9092 #配置本機網絡地址，可寫域名，只要能解析就行 <br />
num.network.threads=3 <br />
num.io.threads=8 <br />
socket.send.buffer.bytes=102400 <br />
socket.receive.buffer.bytes=102400 <br />
socket.request.max.bytes=104857600 <br />
log.dirs=/tmp/kafka-logs #配置日誌輸出目錄 <br />
num.partitions=1 <br />
num.recovery.threads.per.data.dir=1 <br />
offsets.topic.replication.factor=1 <br />
transaction.state.log.replication.factor=1 <br />
transaction.state.log.min.isr=1 <br />
log.retention.hours=168 <br />
log.segment.bytes=1073741824 <br />
log.retention.check.interval.ms=300000 <br />
zookeeper.connect=10.205.35.242:2181,10.205.35.243:2181,10.205.35.244:2181 #配置zookeeper集群連接參數 <br />
zookeeper.connection.timeout.ms=6000 <br />
group.initial.rebalance.delay.ms=0 <br />

其他節點參照上文配置文件，只需修改listeners地址 <br />
***

啟動Kafka <br />
第一種方式（推薦） <br />
kafka-server-start.sh -daemon ../config/server.properties <br />
第二種方式 <br />
nohup kafka-server-start.sh config/server.properties & <br />
停止 <br />
kafka-server-stop.sh <br />

可觀察進程 <br />
ps aux|grep kafka <br />

***
日誌輸出說明 <br />
server.log: 	kafka系統日誌 <br />
state-change.log:   	leader切換日誌，就是broker宕機副本切換 <br />
controller.log:  	kafka集群中有一台機器是控制器，那麼控制器角色的日誌就記錄在這裡 <br />

***
測試驗證生產者與消費者之間的消息發送 <br />
創建topic <br />
./kafka-topics.sh --create --zookeeper 10.205.35.242:2181 --replication-factor 1 --partitions 1 --topic test123 <br />

生產消息 <br />
./kafka-console-producer.sh --broker-list 10.205.35.242:9092 --topic test123 <br />

消费消息 <br />
./kafka-console-consumer.sh -bootstrap-server 10.205.35.243:9092 --topic test123 --from-beginning <br />

***

Kafka常用命令 <br />
./kafka-topics.sh --create --zookeeper 10.205.35.242:2181/ --replication-factor 2 --partitions 1 --topic test6 <br />

--replication-factor <br />
該主題每個分區的副本數，不能大於broker數量。這個副本用於備份，假設每個分區有2個副本，那麼只有一個是leader負責讀寫，follower只是負責同步內容，對外提供讀寫的也是由leader提供。 Leader宕機follower接管成為新的leader。這裡的數量是包括Leader+Follwer的 <br />
--partitions <br />
分區數量，控制將主題切分成多少個LOG。消費者數量應該和分區數量相等，如果超過則毫無意義。 <br />
--topic主題名稱 <br />

平衡讀寫 <br />
./kafka-preferred-replica-election.sh --zookeeper 10.205.35.242:2181 <br />

線上集群所有主題情況 <br />
./kafka-topics.sh --describe --zookeeper 10.205.35.242:2181/ --topic test6 <br />

***

kafka配置文件說明 <br />

----------------------系統相關---------------------- <br />

*  broker的全局唯一編號，不能重複，和zookeeper的myid是一個意思 <br />
broker.id=0 <br /> 


*  broker監聽IP和端口也可以是域名<br /> 
listeners=PLAINTEXT://10.205.35.242:9092 <br />


*  用於接收請求的線程數量 <br />
num.network.threads=3 <br />


*  用於處理請求的線程數量，包括磁盤IO請求，這個數量和log.dirs配置的目錄數量有關，這裡的數量不能小於log.dirs的數量 <br />
雖然log.dirs是配置日誌存放路徑，但是它可以配置多個目錄後面用逗號分隔 <br />
num.io.threads=8 <br />


*  發送緩衝區大小，也就是說發送消息先發送到緩衝區，當緩衝區滿了之後一起發送出去 <br />
socket.send.buffer.bytes=102400 <br />


*  接收緩衝區大小，同理接收到緩衝區，當到達這個數量時就同步到磁盤 <br />
socket.receive.buffer.bytes=102400 <br />


*  向kafka套接字請求最大字節數量，防止服務器OOM，也就是OutOfMemery，這個數量不要超過JAVA的堆棧大小 <br />
socket.request.max.bytes=104857600 <br />


*  日誌路徑也就是分區日誌存放的地方，你所建立的topic的分區就在這裡面，但是它可以配置多個目錄後面用逗號分隔 <br />
log.dirs=/tmp/kafka-logs <br />


*  消息體（也就是往Kafka發送的單條消息）最大大小，單位是字節，必須小於socket.request.max.bytes值 <br />
message.max.bytes =5000000 <br />


*  自動平衡由於某個broker故障會導致Leader副本遷移到別的broker，當之前的broker恢復後也不會遷移回來，有時候我們需要 <br />
手動進行平衡避免同一個主題不同分區的Leader副本在同一台broker上，下面這個參數就是開啟自動平衡功能 <br />
auto.leader.rebalance.enable=true <br />


*  設置了上面的自動平衡，當故障轉移後，隔300秒（默認）觸發一個定時任務進行平衡操作，而只有代理的不均衡率為10%以上才會執行 <br />
leader.imbalance.check.interval.seconds=300 <br />


*  設置代理的不均衡率，默認是10%  <br />
leader.imbalance.per.broker.percentage=10 <br />


*  ---------------分區相關------------------------- <br />
默認分區數量，當建立Topic時不指定分區數量，默認就1 <br />
num.partitions=1 <br />


*  是否允許自動創建topic ，若是false，就需要通過命令創建topic <br />
auto.create.topics.enable =true <br />


*  一個topic ，默認分區的replication個數 ，不得大於集群中broker的個數 <br />
default.replication.factor =2 <br /> 


---------------日誌相關------------------------- <br />

*  segment文件默認會被保留7天的時間，超時的話就會被清理，那麼清理這件事情就需要有一些線程來做。 <br />
這裡就是用來設置恢復和清理data下數據的線程數量 <br />
num.recovery.threads.per.data.dir=1 <br />


*  日誌文件中每個segment的大小，默認為1G。 topic的分區是以一堆segment文件存儲的，這個控制每個segment的大小，當超過這個大小會建立一個新日誌文件 <br />
這個參數會被topic創建時的指定參數覆蓋，如果你創建Topic的時候指定了這個參數，那麼你以你指定的為準。 <br />
log.segment.bytes=1073741824 <br />


*  數據存儲的最大時間 超過這個時間 會根據log.cleanup.policy設置的策略處理數據，也就是消費端能夠多久去消費數據 <br />
log.retention.bytes和log.retention.minutes|hours任意一個達到要求，都會執行刪除 <br />
如果你創建Topic的時候指定了這個參數，那麼你以你指定的為準 <br />
log.retention.hours|minutes=168 <br />


*  這個參數會在日誌segment沒有達到log.segment.bytes設置的大小默認1G的時候，也會強制新建一個segment會被 <br />
topic創建時的指定參數覆蓋 <br />


*  log.roll.hours=168 <br />
上面的參數設置了每一個segment文件的大小是1G，那麼就需要有一個東西去定期檢查segment文件有沒有達到1G，多長時間去檢查一次， <br />
就需要設置一個週期性檢查文件大小的時間（單位是毫秒）。 <br />
log.retention.check.interval.ms=300000 <br />


*  日誌清理策略 選擇有：delete和compact 主要針對過期數據的處理，或是日誌文件達到限制的額度， <br />
如果你創建Topic的時候指定了這個參數，那麼你以你指定的為準 <br />
log.cleanup.policy = delete <br />


*  是否啟用日誌清理功能，默認是啟用的且清理策略為compact，也就是壓縮。 <br />
log.cleaner.enable=false <br />


*  日誌清理時所使用的緩存空間大小
log.cleaner.dedupe.buffer.size=134217728 <br />


*  log文件"sync"到磁盤之前累積的消息條數，因為磁盤IO操作是一個慢操作,但又是一個"數據可靠性"的必要手段 <br />
所以此參數的設置,需要在"數據可靠性"與"性能"之間做必要的權衡. <br />
如果此值過大,將會導致每次"fsync"的時間較長(IO阻塞) <br />
如果此值過小,將會導致"fsync"的次數較多,這也意味著整體的client請求有一定的延遲. <br />
物理server故障,將會導致沒有fsync的消息丟失. <br />
log.flush.interval.messages=9223372036854775807 <br />


*  檢查是否需要固化到硬盤的時間間隔 <br />
log.flush.scheduler.interval.ms =3000 <br />


*  僅僅通過interval來控制消息的磁盤寫入時機,是不足的. <br />
此參數用於控制"fsync"的時間間隔,如果消息量始終沒有達到閥值,但是離上一次磁盤同步的時間間隔 <br />
達到閥值,也將觸發. <br />
log.flush.interval.ms = None <br />

--------------------------複製(Leader、replicas) 相關---------------- --- <br />

*  partition leader與replicas之間通訊時,socket的超時時間 <br />
controller.socket.timeout.ms =30000 <br />


*  replicas響應partition leader的最長等待時間，若是超過這個時間，就將replicas列入ISR(in-sync replicas)， <br />
並認為它是死的，不會再加入管理中 <br />
replica.lag.time.max.ms =10000 <br />


*  follower與leader之間的socket超時時間 <br />
replica.socket.timeout.ms=300000 <br />


*  leader複製時候的socket緩存大小 <br />
replica.socket.receive.buffer.bytes=65536 <br />


*  replicas每次獲取數據的最大大小 <br />
replica.fetch.max.bytes =1048576 <br />


*  replicas同leader之間通信的最大等待時間，失敗了會重試 <br />
replica.fetch.wait.max.ms =500 <br />


*  fetch的最小數據尺寸,如果leader中尚未同步的數據不足此值,將會阻塞,直到滿足條件 <br />
replica.fetch.min.bytes =1 <br />


*  leader 進行複制的線程數，增大這個數值會增加follower的IO <br />
num.replica.fetchers=1 <br />


*  最小副本數量 <br />
min.insync.replicas = 2 <br />

***

集群的調優方向 <br />

*  調大zookeeper的heap內存，默認是1G，可以根據服務器大小配置其堆內存為2G或者4G <br />
　　

*  修改kafka的副本數，默認的副本數是1，建議修改為2，如果副本數為2，那麼容災能力就是1，如果副本數3，則容災能力就是2，當然副本數越多，可能會導致集群的性能下降，但是可靠性更強，各有利弊，推薦副本數為2； <br />


*  kafka推薦分區數，默認的分區數是1，理論上來說：parition的數量小於core的數量的話，值越大，kafka的吞吐量就越高，但是你必須得考慮你的磁盤IO的瓶頸，因此我不推薦你將分區數這只過大，我建議這個值大於broker的數量，比如我的集群broker的只有5台，我的集群的partition數量是20； <br />


*  kafka的heap內存，默認也是1G，生成環境中建議將它調大，此其並不需要分配太多的堆內存空間)，zookeeper給的是2G，剩下的全部給操作系統預留著 <br />
