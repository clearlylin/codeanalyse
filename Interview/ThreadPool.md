# Thread Pool

## 概念

线程池使用线程的模式，线程池维护着多个线程，等待着监督管理者分配可并发执行的任务。
这避免了在处理短时间任务时创建与销毁线程的代价。
线程池不仅能够保证内核的充分利用，还能防止过分调度。

## 线程池的设计