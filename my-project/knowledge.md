# 一些基础知识
- 1、多进程创建的三种方式：multiprocessing.set_start_method("xxx")
  - spawn:全新干净进程，不继承父进程内存，安全但启动慢，全平台可用
  - fork：复制父进程内存快照，启动快，但会把锁/文件句柄/GPU上下文也复制过去，仅linux/Mac
  - forkserver：启动一个独立服务器进程，由它来fork子进程，折中方案，仅linux。
  > 用了 PyTorch/CUDA，只能用 spawn，fork 会因为 GPU 上下文被复制而直接崩溃。
- 2、服务启动时绑定的ip地址(监听地址)
  - 0.0.0.0：监听本机所有网络接口，允许任何来源的请求访问中国服务
  - 127.0.0.1(localhost)：只有本地可以访问，外部连不上
  - 192.168.x.x:只能通过该ip访问
