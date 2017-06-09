# gRPC-Java 快速上手 #

## gRPC ##

[gRPC](http://grpc.io)是Google與Square公司開發的基於[HTTP/2](https://http2.github.io/)為傳送媒介的開源RPC(Remote Procedure Call)函式庫，
其資料傳輸的序列化方法目前官方提供的預設是以Google的[Protocol Buffers](https://developers.google.com/protocol-buffers/) 這種二進位方式編碼，速度較傳統以REST API格式傳送JSON來得快。

## gRPC 術語及其概念解釋 ##

gRPC server/client端官方函式庫支援多種程式語言，雖然各種程式語言的寫法會有所不同，但在函式庫以及文件之中會有幾種概念或術語，以下列出常見術語及其概念解釋(粗體字為官方文件中有提及的定義)：

* **Channel** ：一層更高階包裝 **連線到gRPC Server** 的物件，可設定連線各種設定，在跑單元測試時有些語言的函式庫有提供"*in-Process* channel"以方便寫&執行測試。  
[http://www.grpc.io/docs/guides/concepts.html#channels](http://www.grpc.io/docs/guides/concepts.html#channels)
* **Deadline/Timeout** ：定義gRPC服務呼叫可容許執行的時間，**Deadline** 表示總共可容許執行的時間完畢後的**時刻** ，**Timeout** 表示總共可容許執行多少**時間**  
[http://www.grpc.io/docs/guides/concepts.html#deadlinestimeouts](http://www.grpc.io/docs/guides/concepts.html#deadlinestimeouts)
* **Metadata** ：
* **Streaming** :
* Context ：
* Interceptor ：

## Work flow ##

使用gRPC函式庫的工作流程如下：

1. 建立敘述gRPC服務的 **.proto* DSL(Domain Specific Language)檔案  
(或是在自己程式碼的資料model物件上加上gRPC函式庫提供該程式語言的annotation/attribute/decorator等等屬性設定)  
撰寫由Protocol Buffer定義的RPC呼叫request/response資料格式和呼叫方法，目前gRPC用的是[Protocol Buffer version 3](https://developers.google.com/protocol-buffers/docs/proto3)
，也請參考Style Guide來做出合乎規範的RPC命名:
[https://developers.google.com/protocol-buffers/docs/style](https://developers.google.com/protocol-buffers/docs/style)

2. 從 **.proto* 檔產生出配合使用的程式語言的原始碼檔  
通常會有下列三種自動產生的程式碼：
    * Request/Resoponse Model：Protocol Buffer定義的訊息格式物件以及其建立和屬性存取方法。
    * gRPC Server Code：gRPC呼叫方法的Interface定義或是Abstract Class
    * gRPC Client Code：可在客戶端使用的gRPC呼叫用Proxy物件或Stub方法

3. Server端實作程式碼  
    1. 實作前一個步驟產生的gRPC服務介面定義方法/繼承Abstract Class並複寫宣告方
    2. 建立server物件在專案中啟動的方法。

4. Client端gRPC呼叫
    1. 在呼叫服務前可能需要做一些呼叫前的設定和參數組態物件的建立，由gRPC在該程式語言所提供的函式庫資源而定。
    2. 使用步驟3.產生的gRPC呼叫用Proxy物件或Stub方法，呼叫gRPC服務，

以下用Java的[gRPC-Java]做詳細介紹。

## gRPC-Java ##



## 參考資料 ##

gRPC: The Story of Microservices at Square  
[https://www.youtube.com/watch?v=-2sWDr3Z0Wo](https://www.youtube.com/watch?v=-2sWDr3Z0Wo)

gRPC 101 for Java Developers  
[https://www.youtube.com/watch?v=5tmPvSe7xXQ](https://www.youtube.com/watch?v=5tmPvSe7xXQ)

gRPC Design Principle  
[http://www.grpc.io/blog/principles](http://www.grpc.io/blog/principles)

gRPC using HTTP/2 protocol spec  
[http://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md)
