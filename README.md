# GopherChina 2017


## 4/15

### Go 在大數據開發中的實戰經驗


- Pandora @ 七牛
- 技術 stack 太複雜，只想關注業務的人很困難
- 數據導出會有哪些問題？
	- 本質：數據流動的效率問題
	- 降低延遲
	- 數據模式：Pull(數據已經在數據源上，kafka/hdfs，去啦取即可)、Push (讓用戶打過來)
	- 精細化調度 -> 使用輕量化 goroutine 
	- 沒有人比下游更懂下游 -> 使用插件形式
	- 使用事務形式，只要放到 channel 就 200 ok，不然就 503 fail
- 分佈式系統的一致性問題
	- zookeeper/etcd
	- 自己開發
	- 最終一直性 -> pull + 版本戳記
- 彈性化機器配置
	- 白名單機器配置
- 透過 protobuf 協議和上游通信 <--> 和 json 至少有十倍差異


- Reference
	- [GopherChina講師專訪-七牛雲大數據高級工程師孫健波](https://www.taiwanfansclub.com/article-597406-1.html)

### Go in TiDB

TiDB is a distributed NewSQL database compatible with MySQL protocol.


下一代數據庫需要具備的能力，所以發展 TiDB
- scalability: 儲存、計算的能力
- 高可用 (HA)
- ACID
- SQL

Challenge in building a distributed database
- very complex distributed system
- a lot of rpc work
- high performance (在雲計算時代給 db 的計算時間很少)
- tons of data
- huge amount of oltp query
- very complex olap query

Why Go
- productivity
- concurrency
- great network programming
- GC
- standard library & tools
- good performance
- quick improvement

Goroutine
- start goroutine is easy and cheap
- goroutine come with built it primitive to communicate safely between themselfs
- very easy to write concurrent program
- make full use of multi-core procesors

Goroutine Leak
- write to a chan with no reader
- read from a chan with no wirter
- how to solve
	- block profile
	- timeout
	- context

goroutine leak test
- https://github.com/pingcap/tidb/tree/master/util/testleak

Go 語言的優勢對 TiDB 是可以用並行的思維來設計一個資料庫

經驗分享
- 要想辦法減少內存分配壓力
- reuse object
	- share a stack for all query in one session
- sync.Pool
	- thread safe
	- reuse object to relieve pressure on the gc
- protobuf
	- golang 的 protobuf 有點小問題，後來用了 gogo/protobuf 

TiDB 使用了 gogo/protobuf，而不是官方的 protobuf

NEw in Go1.8
- better gc


- Reference
	- [tidb](https://github.com/pingcap/tidb)
	- [分佈式數據庫 TiDB 過去現在和未來](https://gocn.io/article/14)

### Go coding in Go style




- Reference
	- [GopherChina講師專訪-東軟雲科技架構師白明](http://chuansong.me/n/1741471551626)

### Understanding Go Interfaces


#### In go, two types
- abstract types
	- they describe behavior
	- sets of methods, without specifying receiver
- concrect types
	- they describe a memory layout
	- ex: int

"interface{} says nothing"

"The bigger the interface, the weaker the abstraction"

Retrun concrect type, receive interface type



### NSQ 重塑之路

- Reference
	- [GopherChina講師專訪-杭州有贊技術專家李文](http://chuansong.me/n/1717421851726)

### 基於 Go 的微服務架構 (Spring 開發者視角)

ali 基本上是 Java 起家，也有 C/C++，Go 還在增長中，增長速度也是挺可觀。

Micro-service complexity in Ali
- team is too large
- produtivity velocity

Many microservice inferited problem are solved with Dubbo and Spring framework, however:
- testing is srill hard
- devops culture
- security


- Reference


### 用 Go 搭建 Kubernetes Operators

k8s 的 operator 是用來管理和部署 cluster

- Reference


### 嗶哩嗶哩的 Go 微服務實戰

一開始整個代碼是一大包

如果改為微服務架構
1. 梳理業務邊界
	- API 要兼容
	- 先從非核心服務開始
2. 資源隔離部署
	- 買一台新機器放心的代碼
	- 舊的代碼沒有文件，不動
3. 內外網服務隔離
	- 以前內網的服務也可以被外部訪問（只要有權限）
	- 定義某些不能對外的 API 要放到內網
4. RPC 框架
	- 序列化(GOB)
	- 上下文管理(超時控制)
	- 攔截器(統計、限流、授權)
	- 服務註冊(zookeeper)
	- 負載均衡(客戶端)
5. API Gateway



- Reference


## 4/16

### 純 Go 打造億級實時分佈式平台

#### why go at grab

- 簡潔的語言規範
- 完整的工具鍊
- 方便的部署流程
- 優秀的性能

#### Grab 用 Go 在做什麼
- Machine learning -> 怎麼分配一個司機給顧客

#### Grab at Go
- 所有的 Go 代碼都在一個 repo
- 一致的版本，open source of truth
- 極致的代碼分享，共用
- 簡單的依賴管理
- 原子化的代碼變更
- 更好的支撐大規模的重構，代碼庫的更新
- 團隊之間更好的合作
- 靈活的團隊界線以及代碼歸屬權
- 最大化的代碼透明度，以及自然而然按團隊團隊的命名空間

#### 缺點
- 市面上的 CI 系統沒有為上面這樣的管理方式做優化
- 傳統的 CI 需要 40 mins
- 需要有強大的工具鍊來支撐這樣的方式

#### Distributed Tracing
單體應用 -> 大規模的微服務架構
- 函數變成了服務
- 越來越多的 vendor libs
場景
- 一個請求要 3s，這個請求可能流過好幾個服務，到底哪個服務出問題
- 怎麼定義 single point of failure
- 如何檢測避免循環依賴關係
- 怎麼定位 fan in、fan out
實現原理
- 當一個請求進入系統時，是進入一個 api gateway，這時候生成一個全局唯一的 trace id，並加入 header 中
- 每個系統中再生成一個 key，把 traceid+key 用來定位每個服務的耗時


#### Testing
- 單元測試
- 端到端測試
	- 要有真實的環境
	- Postman API
	- Go Test 

#### Code Quality Control
- Code Review 非常重要
- 但重要性很常被忽略
- 好的工具能夠提高 Code Review 效率
- 我們使用的工具
	- Phabricator
	- Jenkins
	- Slackbot
- 當代碼請求 review 前 (check-in 後)
	- Phabricator + Arcanist
	- 跑 Linter + 單元測試引擎(自己開發)
- Build Pass
	- 當 CI Build 成功後，開發者會收到 SlackBot 祝賀消息
	- Reviewer 可以開始代碼審查
	- Phabricator 頁面上會清晰標註冊是覆蓋捍衛覆蓋的代碼塊
- Build Fail
	- 開發者會收到 build fail  
	- Phabricator 會 reject diff
	- 會顯示哪個測試沒通過
	- maintainer 會收到 mail 通知
	- build fail 的紀錄會記錄在 server 端，長期程式碼質量很差會收到通知
- Test Coverage
	- 當單元測試覆蓋率低於一定百分比時，CI 會自動 build fail

#### Go 常見的問題
- Nil Pointer
	- make zero value useful 
- DNS Resolution
	- Go net library 沒有完整實現 RFCxxx 標準
	- 造成 DNS round robin 沒有勻稱的附載均衡
	- Go 1.9 will fix
	- [https://github.com/golang/go/issues/18518](https://github.com/golang/go/issues/18518) 


### Go 語言在掃碼支付系統中的成功實踐

#### 全球支付的特點
- 信息流：API Gateway
- 商業對帳服務：批次處理


#### 技術選型
- 業務需求
- 技術需求
	- 每個技術都有它適用的範圍
	- API Gateway、批次處理都是 Golang 擅長的範圍
- 團隊需求
- 團隊背景
	- C, Java, Golang
- 安全性
	- 數據傳輸安全性
	- 防竄改

#### 架構演進
-
-
-
-

### Go 在百萬億級搜索引擎中的應用

自己開發的 Poseidon

#### 設計目標
- 保留 3y 歷史資料，百億條，100PB
- 秒及交互搜索
- 每天 2000億數據灌入
- 原始數據僅存一份
- 自定義分詞策略
- 故障轉移，附載均衡，自動恢復
- HBase 在千萬等級沒問題，百億也是有問題

#### Go 應用場景與挑戰
- 用 ProtoBuffer 描述核心數據結構

#### Go 的坑
- 傳統組件是 c++，c++ -> c -> cgo -> go
- 一直掛，用 gdb 調適，發現進程過多
- 即使精通多個語言，最好不要混用。謹慎引入其他語言的解決方案
- 不要完全相信 recover，他不能恢復 runtime 的一些 panic

#### 總結
- 開發體驗好、性能高、服務穩定，可以滿足大部份場景需求
- Go 開發需要在代碼惡毒性和性能之間做好取捨，應用程序開發模型要在控制之內
- 謹慎與其他語言結合
- 合理引進第三方解決方案，在維運成本和系統維護成本之間卻考量

### The hidden #pragma's of Go

#### Go has pragmas
-
-
-
-

####
-
-
-
-

####
-
-
-
-

####
-
-
-
-

### 跨境電商的 Go 服務治理實踐

#### 背景
- C# to Go，計畫全部換成 Go
- 第一件事情就是從規範開發環境做起
- 這次分享介紹從零開始要怎麼開始切換到 Go

#### 開發環境建構 Goflow
- 開發環境統一化
	- 一個 sh 檔案，source 一下即可用
	- 修改 GOPATH、PATH
	- 在所在環境註冊一些函數
	- 負責基礎庫更新、工具練的編譯 
- 第三方依賴方案
	- 共享依賴
	- 內網緩存、不走小水管
	- 和業務代碼分開
	- 隨意修改第三方包 
- 編譯流程一體化

#### 微服務選型
- gRPC
- 使用 pb 描述接口
	- 包 -> 服務 -> 方法
	- d
- d	s

#### 分佈式追蹤
-
-
-
-
 
#### 跨數據中心
-
-

### 使用 Golang 語言實現 DevOps Orchestration

####
-
-
-
-

####
-
-
-
-

####
-
-
-
-

####
-
-
-
-

### Harbor 開源項目中容器鏡像遠程複製的實現

#### vmware 開發的 docker registry for enterprise usage
-
-
-
-

####
-
-
-
-

####
-
-
-
-

####
-
-
-
-

### Go 在證券行情系統中的應用

#### 證券行情系統特點
- 超低延遲
- 超高併發
- 超高可靠性
- 超嚴格監督

#### 行情開發遇到的挑戰
- 開發語言的選擇
- GC 問題
	- Go 的 GC 使用 CMS 算法，優點是不中斷業務下並行執行，將 STW 時間降低到最小
	- 缺點是更多的同步降低吞吐量，堆的快速增長 
- GC 算法考量
	- 併發
	- 停頓時間
	- 停頓頻率
	- 壓縮
	- 堆內存開銷
	- ＧＣ吞吐量 
-

####
-
-
-
-

####
-
-
-
-
### Go 語言在證券期貨行情系統中的實踐

#### 
-
-
-
-

####
-
-
-
-

####
-
-
-
-

####
-
-
-
-