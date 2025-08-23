---
{"dg-publish":true,"permalink":"/zettel/202305222026mermaid-tutorial/","title":202305222026,"tags":["mermaid","graph","draw","画图","作图"]}
---


Pie
-----

```mermaid
pie
    title 今天晚上吃什么？
    "火锅" : 8
    "外卖" : 60
    "自己煮" : 8
    "海底捞" : 9
    "海鲜" : 5
    "烧烤" : 5
    "不吃" : 5
```

```mermaid
pie
    title 时间分布
    "GRE拨测" : 35
    "NCE需求" : 15
    "NCE/SPU问题单" : 12
    "集采LLT支撑" : 10
    "SA-UI安装/后台组件编译" : 7
    "NCE/SPU网课学习" : 6
    "SIG前后台补丁制作/安装" : 5
    "可信考试" : 5
    "三营/MINI培训" : 4
```

Flow chart
---

```mermaid  
graph LR  
A[矩形] -.->B(圆角矩形) --> C{菱形} ==> D((圆形))   
E([体育场形])--实线文本--> F[[子程序形]]==粗实线文本==>G[(圆柱形)]-.虚线文本.->H{{六角形}}  
I[/平行四边形/]-.-J[\反向平行四边形\]---K[/梯形\]===L[\反向梯形/]
```


```mermaid
graph RL

        User((用户))--1.用户登录-->Login(登录)
        Login --2.查询-->SERVER[服务器]
 subgraph 查询数据库
        SERVER--3.查询数据-->DB[(数据库)]
        DB--4.返回结果-->SERVER
 end
        SERVER--5.校验数据-->Condition{判断}
        Condition -->|校验成功| OK[登录成功]
        Condition -->|校验失败| ERR[登录失败]
        OK-->SYS[进入系统]

        ERR -->|返回登录页面,重新登录| Login
```

Sequence diagram
---

```mermaid
sequenceDiagram
Alice->>+John: Hello John, how are you?
Alice->>+John: John, can you hear me?
John-->>-Alice: Hi Alice, I can hear you!
John-->>-Alice: I feel great!
```


```mermaid
sequenceDiagram
    autonumber
    participant Z as 张三
    participant L as 李四
    participant W as 王五

    Note over Z,W: 张三,李四,王五, 小时候是最好的玩伴,现在80年过去了...
    Z->>W: 老王最近还好吗？
    Note left of Z: 除了老张,当过兵,身体比较好之外,其他两人都不太行了
    loop 健康检查
        W->>W: 与疾病战斗
    end
    Note right of W: 合理进食   看医生  打点滴...
    W-->>Z: 还行,老了走不动了 !!
    L->>Z: 老张,你呢怎样了
    alt 健康#9829;
    Z-->>L: 很好!
    else 去世
    Z-->>L: 对不起,老张已经走了!!!
end
```


Gantt
---

```mermaid
gantt  
  
dateFormat YYYY-MM-DD  
title 开发计划  
  
section 需求文档  
	登录注册:done,login,2021-06-25,2021-06-28  
	添加好友与分组:add, 2021-06-29,3d  
	单聊 :active ,chat, 2021-07-01,2d  
	群聊 :groupChat,after chat,5d  
	朋友圈 :crit,5d  
	其他:3d  
section 开发  
	开发登录注册:done,d-login,2021-06-25,24h  
	开发添加好友与分组:active,d-add,after d-login,5d  
	开发单聊与群聊:crit,d-chat,after d-add,7d  
	开发朋友圈:d-friend,after d-chat,7d  
	  
section 测试  
	测试用例与玩耍:active,test,2021-06-25,10d  
	开始测试部分接口:crit,start-test,after test,11d
```

Class diagram
---

```mermaid
classDiagram  
  
class Serializable{  
<<interface>>  
}  
class Throwable{  
<<class>>  
 String detailMessage;  
 Object backtrace;  
 Throwable();  
}  
  
class Exception{  
<<class>>  
Exception()  
Exception(String message)  
}  
class IOException{  
	<<class>>  
}  
  
class SocketException{  
	  
}  
  
class RuntimeException{  
<<class>>  
}  
  
class IndexOutOfBoundsException{  
	  
}  
  
class ArithmeticException{  
	  
}  
  
Serializable <|.. Throwable:序列化接口  
Throwable *--Exception  
Exception <-- RuntimeException:运行时异常  
Exception <-- IOException:io流相关异常  
IOException o-- SocketException  
RuntimeException o-- IndexOutOfBoundsException  
RuntimeException o-- ArithmeticException
```


State diagram
---

```mermaid
stateDiagram-v2  
  
    [*] --> 静止  
    静止 --> [*]  
    静止 --> 走  
    走 --> 静止  
    走 --> 跑  
    跑 --> 走:跑可以停下来就静止，也可以慢下来：走  
    跑 --> [*]  
  
  
    state if_state <<choice>>  
    [*] --> IsPositive  
    IsPositive --> if_state  
    if_state --> False: if n < 0  
    if_state --> True : if n >= 0  
  
  
    state fork_state <<fork>>  
      [*] --> fork_state  
      fork_state --> State2  
      fork_state --> State3  
  
      state join_state <<join>>  
      State2 --> join_state  
      State3 --> join_state  
      join_state --> State4  
      State4 --> [*]  
  
    [*] --> First  
    First --> Second  
    First --> Third  
  
    state First {  
        [*] --> fir  
        fir --> [*]  
    }  
    state Second {  
        [*] --> sec  
        sec --> [*]  
    }  
    state Third {  
        [*] --> thi  
        thi --> [*]  
    }
```


```mermaid
classDiagram

class RefCounted {
    <<interface>>
}

class IOManager {
    <<interface>>
    +SetWorkersProxy(IOManager*)
    +Shutdown()
    +RegisterMessage(uint8_t, uint16_t, uint16_t)
    +AsyncPublish(MDLMessage*)
    +SyncPublish(MDLMessage*)
    +AsyncResponse(Publisher*, RefCounted*, MDLMessage*)
    +AsyncRebuildResponse(Publisher*, RefCounted*, MDLMessage*)
    +CreateAutoSubscriber(MessageHandlerBase*, bool)
}

class ASIOThread {
    #OnAccept(ASIOListenerPtr, ASIOSocketPtr)
    #OnConnect(ASIOSocketPtr, error_code)
    #OnOpen(ASIOSocketPtr) 
    #OnClose(ASIOSocketPtr, error_code)
    #OnRead(ASIOSocketPtr)
    #OnWritten(ASIOSocketPtr)
    #OnTimer()
    #OnThreadInitialize(char*)
}

class IOManagerImpl {
    -m_ClientSockets: vector~SubscriberImplPtr~
    -m_Publishers: map~int,PublisherImplBasePtr~
    -work_threads_: vector~_WorkThreadItem*~
    -thread_pool_: shared_ptr~thread_pool~
    +Open(bool, int, int)
    +Close()
    +Post(int, RefCountedPtr)
    +RegisterSubscriber(SubscriberImplPtr)
    +UnRegisterSubscriber(SubscriberImplPtr)
    +SetRequestHandler(RequestHandler*)
    +GetCodecManager()
    +GetSubscriberIOStat()
    +GetPublisherIOStat()
}

RefCounted <|-- IOManager
IOManager <|-- IOManagerImpl
ASIOThread <|-- IOManagerImpl
```

References
---

1. [Mermaid doc][1]
2. https://rstyro.github.io/blog/2021/06/28/Markdown%E6%B5%81%E7%A8%8B%E5%9B%BE%E8%AF%AD%E6%B3%95%E7%A4%BA%E4%BE%8B/
3. https://www.drawio.com/blog/mermaid-diagrams


[1]:https://mermaid.js.org/intro/n00b-syntaxReference.html
[2]:https://rstyro.github.io/blog/2021/06/28/Markdown%E6%B5%81%E7%A8%8B%E5%9B%BE%E8%AF%AD%E6%B3%95%E7%A4%BA%E4%BE%8B/