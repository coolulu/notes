# NewProc
>**支持功能 :**
> * 消息任务
> * 超时任务
> * 定时任务
> * 任务分发(汇总)

---

## 1.通用模式 (CPU密集型的程序)
![](https://raw.githubusercontent.com/coolulu/notes/master/_image/NewProc/model_normal.png)

### 1.1 支持功能
* [x]   消息任务
* [x]   超时任务
* [x]   定时任务
* [ ]   任务分发(汇总)

### 1.2 线程概述
* 主线程 x 1
* 工作线程 x 12 (假设12个核)
* 超时检查线程 x 1
* 异步资源链接操作线程 x 1

最低消费**15**线程

#### 1.3 功能说明

##### 1.3.1 消息任务
![](https://raw.githubusercontent.com/coolulu/notes/master/_image/NewProc/task_msg.png)

##### 1.3.2 超时任务
![](https://raw.githubusercontent.com/coolulu/notes/master/_image/NewProc/task_timeout.png)

##### 1.3.3 定时任务
![](https://raw.githubusercontent.com/coolulu/notes/master/_image/NewProc/task_clock.png)




## 2.任务分发模式 (IO密集型或并行计算的程序)
![](https://raw.githubusercontent.com/coolulu/notes/master/_image/NewProc/model_cut.png)

### 2.1 支持功能
* [ ]   消息任务
* [ ]   超时任务
* [ ]   定时任务
* [x]   任务分发(汇总)

### 2.2 线程概述
* 主线程 x 1
* 工作线程 x 24 (假设12个核)

最低消费**25**线程

#### 2.3 功能说明

##### 2.3.1 任务分发(汇总)

![](https://raw.githubusercontent.com/coolulu/notes/master/_image/NewProc/task_cut_1.png)
TaskCutBase是分发任务, TaskCutCollectBase是汇总任务

![](https://raw.githubusercontent.com/coolulu/notes/master/_image/NewProc/task_cut_2.png)
