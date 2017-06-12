# gRPC-Java 快速上手 #

## gRPC ##

[gRPC](http://grpc.io)是Google與Square公司開發的基於[HTTP/2](https://http2.github.io/)為傳送媒介的開源RPC(Remote Procedure Call)函式庫，
其資料傳輸的序列化方法目前官方提供的預設是以Google的[Protocol Buffers](https://developers.google.com/protocol-buffers/) 這種二進位方式編碼，速度較傳統以REST API格式傳送JSON來得快。

## gRPC 術語及其概念解釋 ##

gRPC server/client端官方函式庫支援多種程式語言，雖然各種程式語言的寫法會有所不同，但在函式庫以及文件之中會有幾種概念或術語，以下列出常見術語及其概念解釋(粗體字為官方文件中有提及的定義)：

* **Channel** ：一層更高階包裝 **連線到gRPC Server** 的物件，可設定連線各種設定，在跑單元測試時有些語言的函式庫有提供"*in-Process* channel"以方便寫&執行測試。  
[http://www.grpc.io/docs/guides/concepts.html#channels](http://www.grpc.io/docs/guides/concepts.html#channels)

* **Deadline/Timeout** ：定義gRPC服務呼叫可容許執行的時間，**Deadline** 表示總共可容許執行的時間完畢後的**時刻** ，**Timeout** 表示總共可容許執行多少**時間**。  
[http://www.grpc.io/docs/guides/concepts.html#deadlinestimeouts](http://www.grpc.io/docs/guides/concepts.html#deadlinestimeouts)  
[敘述Deadline/Timeout概念的投影片](https://www.slideshare.net/borisovalex/enabling-googley-microservices-with-http2-and-grpc/132?src=clipshare)

* **Metadata**/Context ：一種伴隨在gRPC呼叫中夾帶的key-value資料結構，常用來做儲存gRPC呼叫時使用者驗證資料，或是跨不同gRPC方法呼叫時傳遞額外資料。  
[http://www.grpc.io/docs/guides/concepts.html#metadata](http://www.grpc.io/docs/guides/concepts.html#metadata)

* **Streaming** : gRPC和HTTP REST API呼叫不同的獨特新功能，可由server端或client端發出連續的資料流讓另一端接收，也可同時雙向傳送。  
[http://www.grpc.io/docs/guides/concepts.html#server-streaming-rpc](http://www.grpc.io/docs/guides/concepts.html#server-streaming-rpc)  
[gRPC的那些事 - streaming](http://colobu.com/2017/04/06/dive-into-gRPC-streaming/)

* Interceptor ：類似一些程式語言的Web API框架中的 *filter* / *middleware* 的概念，可在client端發送gRPC呼叫前，或是server端真正執行gRPC定義的服務實做前執行客製程式碼，可用於功能：使用者認證、傳輸資料壓縮加解密。  
[gRPC的那些事 - interceptor](http://colobu.com/2017/04/17/dive-into-gRPC-interceptor/)  
Java的Interceptor實作範例：  
[client interceptor](http://www.programcreek.com/java-api-examples/index.php?api=io.grpc.ClientInterceptor)  
[server interceptor](http://www.programcreek.com/java-api-examples/index.php?api=io.grpc.ServerInterceptor)

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
    1. 實作前一個步驟產生的gRPC服務介面定義方法/繼承Abstract Class複寫宣告gRPC服務的函式。
    2. 建立server物件在專案中啟動的方法。

4. Client端gRPC呼叫
    1. 在呼叫服務前可能需要做一些呼叫前的設定和參數組態物件的建立，由gRPC在該程式語言所提供的函式庫資源而定。
    2. 使用步驟3.產生的gRPC呼叫用Proxy物件或Stub方法，呼叫gRPC服務，

以下用Java的[gRPC-Java](http://github.com/grpc/grpc-java)做詳細介紹。

## gRPC-Java ##

Google官方提供的Java程式語言使用gRPC函式庫，支援Oracle Java及Android client端Java使用：  
[http://github.com/grpc/grpc-java](http://github.com/grpc/grpc-java)

Javadoc: <http://www.grpc.io/grpc-java/javadoc/>

專案整合Gradle建置時需要用這個protocol buffer的plugin:  
<https://github.com/google/protobuf-gradle-plugin>

並建議加上這個設定，以便讓eclipse不會去對於protocol buffer產生的Java程式碼做太多無意義的語法檢查(目前protocol buffer產生的Java Code在JDK 8編譯的環境會有很多compile warning)：

```Gradle
apply plugin: 'eclipse'
//remove redundant compile check on generated code.
eclipse {
  classpath {
    file {
      whenMerged {
        entries.each {
          source ->
          if (source.kind == 'src' && source.path.contains('protoGeneratedSrcFolder')) {
            source.entryAttributes['ignore_optional_problems'] = 'true'
          }
        }
      }
    }
  }
}
```

並且加上這個清除產生程式碼的task以便讓gradle clean執行時能確實清除產生的Java程式碼檔案，否則在專案進行過程中修改 **.proto* 檔案時，重新產生的Java Code可能會產生奇怪的編譯錯誤：

```Gradle
task cleanProtoGen{
    doFirst{
      delete("what ever the generated java source path is")
    }
}
clean.dependsOn cleanProtoGen
```

gRPC-Java的軟體架構如下：

![gRPC-Java Architect](https://cdn.rawgit.com/windperson/grpc-springboot-demo/dab11cda/docs/grpc-java-arch.mmd.svg)

## grpc-spring-boot-starter ##

配合Spring Boot的gRPC框架，  
原始碼：[http://github.com/LogNet/grpc-spring-boot-starter](http://github.com/LogNet/grpc-spring-boot-starter)


## 參考資料 ##

gRPC: The Story of Microservices at Square  
[https://www.youtube.com/watch?v=-2sWDr3Z0Wo](https://www.youtube.com/watch?v=-2sWDr3Z0Wo)

gRPC 101 for Java Developers  
[https://www.youtube.com/watch?v=5tmPvSe7xXQ](https://www.youtube.com/watch?v=5tmPvSe7xXQ)

gRPC: A High Performance, Modern RPC System  
[http://www.infoq.com/presentations/grpc](http://www.infoq.com/presentations/grpc)

gRPC Design Principle  
[http://www.grpc.io/blog/principles](http://www.grpc.io/blog/principles)

gRPC using HTTP/2 protocol spec  
[http://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md)
