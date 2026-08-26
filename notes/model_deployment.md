# 模型部署
把训练好的模型权重，从训练环境转化成可对外提供推理服务的可执行程序，满足低延迟、高吞吐、低显存、高可用、成本可控的线上业务需求
部署关注延迟、吞吐量、显存/内存占用、QPS、可靠性、功耗、版本管理。
- 推理：输入数据，模型计算结果
- 部署：把推理能力封装、上线、运维，给业务调用
  ## 模型部署流程
    训练导出模型文件->模型转换与优化->推理引擎加载->服务封装API->上线调度->监控观测->版本迭代
    - 1、模型导出：训练框架权重转通用存储格式
    - 2、模型压缩&优化：量化、剪枝、蒸馏、算子融合
    - 3、推理后端选型：不同硬件匹配不同推理引擎
    - 4、服务化封装：HTTP/gRPC接口，批处理，流式输出
    - 5、资源编排：容器，K8s，弹性扩缩容
    - 6、业务集成：业务后端调用，鉴权，限流
    - 7、可观测性：延迟，GPU显存，错误率，token统计
    - 8、运维：灰度，回滚，A/B测试，多模型管理
  ## 模型导出格式
    - 原生格式：1、Pytorch：.pth/.bin,只存权重，不直接部署，依赖python环境，需要完整网络代码，适合调试，不适合生产。
               2、Hugging Face：Pytorch_model.bin+config.json，LLM最常见，训练侧产物
    - 通用中间表示IR格式（部署核心）：1、ONNX：开放式神经网络交换格式，跨架构，CNN、CV模型最主流（大模型Transformer容易出现算子不兼容，部分自定义算法缺失）.
                                   2、TorchScript（Torch IR）：PyTorch内置，脚本化，剥离python依赖，可被LibTorch直接加载，CV，小模型常用
                                   3、TensorRT-ONNX：不是存储格式，是TensorRT读取ONNX做编译
                                   4、GGUF（前沿）：LLaMA.cpp生态，端侧，CPU，低资源大模型部署事实标准，支持不同量化等级Q4_K_M/Q5_K_M/Q8_0，广泛用于本地部署，边缘设备
                                   5、Safetensors：huggingface安全权重格式，替代bin，无恶意代码风险，加载更快，多用于大模型导出，不是推理引擎，只是权重存储（不能直接跑推理，需要推理引擎加载它）
  ## 模型优化技术：
    降低显存占用，提升推理速度，尽可能少损失精度（分为算法层面优化，编译算子优化）
    - 1、量化Quantization（工业界最常用）：把FP32（32位浮点数）权重压缩为更低比特：FP16/BF16：半精度，几乎无损，GPU部署默认2，FP16更适合大模型，对数值益出更友好，A100/A800/H100原生支持。INT8量化：8比特，CV，多模态广泛使用；分PTQ(训练后量化，不用重训)，QAT(量化感知训练，训练模拟量化，精度更高)
    - 2、低比特量化（前沿LLM）：INT4，INT3，INT2：GPTQ（权重仅量化，激活FP16）、AWQ（激活感知权重量化）、SqueezeLLM（基于稀疏的量化）、GGUF-Q4_K_M(llama.cpp的量化，面向cpu)
  ## 知识蒸馏
    大教师模型输出软标签训练小模型，得到小体积学生模型，适合对延迟敏感业务
  ## 剪枝pruning
    移除冗余权重，分为结构化剪枝（删掉整层/通道，硬件友好），非结构化剪枝（零散权重重置，普通GPU很难加速，需要特殊稀疏硬件）
  ## 算子融合Operator Fusion（编译优化，零精度损失）
    把多个连续小算子合并为一个大算子，减少GPU显存读写开销，例如Conv+BN+ReLU合并，Transformer的QKV融合，推理引擎TensorRT，vLLM，TensorRT-LLM会自动做
  ## kV-Cache缓存（大模型推理核心）
    LLM自回归生成每一步都要保存历史token的key、value向量，即KV Cache，没有KV-cache：每轮计算都要计算全部历史，速度极慢，KVCache占用大量显存，是大模型部署显存主要开销
    - 1、PagedAttention：vLLM提出，类比操作系统虚拟内存，碎片化管理KV缓存，大幅度提升并发能力
    - 2、Continuous Batch（连续批处理）：不等待完整请求结束，动态加入新用户请求。提升GPU利用率，替代传统batch
    - 3、KV Cache量化：把KV缓存也做INT4/INT8，进一步降显存；AWQ，SGLang支持
    - 4、KV Cache稀疏，DropKV：丢弃不重要token的KV向量，长文本场景
  ## speculative decoding 投机解码（前沿推理加速）
    小草稿模型快速预生成诺干token，大模型只做验证，如果token正确直接accept，通过计算，整体吞吐提升，适合在线对话业务，代表框架：vLLM，SGLang支持
# 主流推理引擎：
  推理引擎负责加载模型、执行算子、做编译优化：
  - GPU推理引擎（云端，Nvidia GPU为主）：TensorRT:NVIDIA官方，CV，多模态模型标杆，对CNN优化极强，支持ONNX导入，算子编译，对原生Transformer支持一般，衍生出TensorRT-LLM专门做大语言模型
                                        vLLM：当前LLM部署工业界主流，核心创新PagedAttention+Continuous Batching，高并发，支持GPTQ/AWQ，支持OpenAI兼容接口
                                      SGLang：新兴高性能LLM推理，RadixAttention优化KV缓存，多模态LLM，RLHF服务性能很强，支持投机解码
                                      TensorRT-LLM：英伟达官方大模型推理库，底层CUDA内核深度优化，H100/A800硬件上性能天花板，配置复杂
                                      Text Generation Inference（TGI）：HuggingFace官方推理服务，稳定成熟，早期LLM，并发弱于vLLM
  - CPU部署引擎：llama.cpp:GGUF模型，纯c++实现，CPU推理标杆，也支持部分GPU加速，本地PC，边缘设备
                ONNX Runtime：微软，跨平台CPU/GPU，CV，小模型。windows服务友好
  - 端侧推理（手机，嵌入式板卡）：ONNX Runtime Mobile、TensorFlow Lite、MNN（阿里）、NCNN（腾讯）、移动端CV/小LLM、QNN：高通芯片端侧大模型推理
  - 国产硬件适配：昇腾 CANN、寒武纪 MagicMind：对应国产 NPU，需要专门转换工具链，不能直接跑 ONNX/TensorRT。
# 服务化部署模式：
  ## 1、Python简单封装（测试原型，禁止上生产）
    直接使用PyTorch，FastAPI封装，简单写接口(没有批处理优化，显存无法复用，并发极低，稳定性差，只适合原型验证)
  ## 2、专用推理引擎+HTTP/gRPC服务（标准线上部署）
    vLLM/SGLang/TGI本身内置web服务，输出OpenAI兼容接口，业务直接调用(引擎内部已经实现连续批处理，KV缓存管理；对外提供REST/gRPC)
  ## 3、前后端分层架构
    业务网关层（Nginx/APISIX）->推理服务集群（vLLM多实例）->K8s调度资源
  网关做：鉴权、限流、负载均衡、请求路由、多模型 A/B、日志；后端只负责 GPU 推理。
  ## 4、模型即服务Maas
    云厂商封装好，不用管GPU，调用API：阿里云 PAI、腾讯云 TI、OpenAI API；企业自建 MaaS 平台会封装模型版本管理、镜像、弹性调度。
# 大模型特有部署关键能力
  - 1、流式输出Streaming：token逐字返回，聊天界面打字效果，推理引擎原生支持SSE协议
  - 2、多模型部署：单GPU同时加载多个模型，或者多GPU分布部署不同模型，支持模型热加载，不需要重启服务
  - 3、动态批处理Continuous Batching：区别静态batch；传统静态batch必须等一批请求全部到齐才开始计算，动态批处理请求可以随时加入，GPU利用率显著提升
  - 4、前缀缓存（Prefix Caching/RadixAttention）:多个用户共享相同prompt前缀，缓存KV，知识库RAG场景所以巨大，SGLang重点特性
  - 5、上下文长部署：需要优化KV内存，开启滑动窗口，支持128k上下文
# 资源调度：容器&Kubernetes（K8s）
  生产环境模型全部容器打包，镜像包含推理引擎，CUDA，模型权重
  - GPU调度依赖K8s device-plugin、分配GPU卡给容器
  - 弹性扩缩容，根据队列长度/QPS自动拉起/销毁推理pod；GPU资源昂贵，闲时缩容节省成本
  - 大模型pod启动慢（加载模型权重需要几十秒到几分钟），冷启动延迟很高，无法像普通web服务秒级扩缩容。所以LLM一般采用预留常驻实例+有限扩容策略
# 部署性能指标：
  - TTFT：Time TO First Token：首token延迟，聊天体验最重要，流式场景
  - TPOT：Time Per Output Token,每个输出token生成耗时
  - QPS：每秒请求数
  - 吞吐量Theroughput：token/s，每秒总生成token，衡量GPU算力利用率
  - GPU显存占用，权重显存+KV-Cache显存总和，大模型瓶颈
  - P95/P99延迟：95%，99%请求的最大延迟，线上不能看平均延迟
