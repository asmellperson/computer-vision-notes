# 手机检测前端业务页面组件
- 1、unified_frontend/src/sections/PhoneDetectionSection.tsx：页面
- 2、api/compat.py:接收手机配置API
- 3、PhoneConfigSchema：检查参数
- 4、FeatureConfigService：保存配置
- 5、BaseFunctionConsumer实时检测通用框架
- 6、PhoneConsumer:手机业务判断
- 7、BaseModelWorker：模型进程通用框架
- 8、YOloDetectorWorker:真正model.predict()
## React：
- usestate：解决页面运行过程中，需要保存一些会变化的，React响应式UI的核心
- useeffect：页面第一次打开时加载一次指定的东西（useEffect(() => {loadCameras()}, [])）（加载摄像头）
- lazy：const AiVisionRoiEditor = lazy(() =>import('@/components/AiVisionRoiEditor')) ：ROI编辑器不要页面一打开就加载，真正需要的时候的再加载（懒加载）
- Suspense：配合lazy：ROI组件还没加载完，先显示“正在加载ROI组件”，组件加载完成，显示真正的ROI编辑器
## TypeScript（javaScript+类型系统）
  - type PhoneConfig={...}：规定数据应该长什么样：boolen表示只能是true或false，number表示只能是数字，？表示这个字段可以不存在
  - type PhoneConfigResponse={...}:Python后端给前端返回的Json长什么样，整个流程是Python，JSON，HTTP，浏览器，PhoneConfigResponse(把后端接口的数据结构明确写出来)
  - const：常量，不能重新赋值，let 变量，可以重新赋值
  > defaultForm是字符串：因为HTML输入框里面的值本质上都是字符串，页面编辑阶段是字符串，真正发送后端变成数字（UI 表单数据类型 → 业务数据类型转换。）
>   defaultPhoneConfig是数字，因为他不是输入框里的文字，而是真正的业务配置
>   toForm函数就是在做后端到前端的字符串转换：String（）：python后端 number 0.7->Json->String->React输入框‘0.7’。用户修改 “0.8”->Number()->0.8->Json->Python后端
>   ？？：空值合并运算符 Nullish Coalescing。CONF_THRESHOLD ?? 0.7，表示没有值就使用0.7，0.5 ?? 0.7结果0.5
 - async function requestPhoneConfig(....):函数内部会做耗时的异步操作(这里的耗时操作就是HTTP网络请求)，async标记函数是异步函数，函数返回永远是promise对象
- const response = await fetch(...)：fetch（）是浏览器提供的HTTP 后端，await等待异步操作完成，暂停代码往下跑，拿到结果再继续，await不能直接写在普通函数/顶层 ，必须包裹在async函数里面
- fetch(/api/ai/phone-config/${encodeURIComponent(shmName)}：GET：GET /api/ai/phone-config/shmName对应的值：React,HTTP,Vite Proxy,其中Vite 开发服务器的代理功能，前端请求发给 Vite 服务器，Vite 代替你去请求后端，再把结果拿回给浏览器。
- encodeURIComponent()：URL参数转义：安全编码为URL可以传输的形式
- credentials: 'include'：HTTP请求把登录Cookie带过去
- HTTP状态码：200：成功，400：你的请求有问题，401：没登录，403：登录了，但是没有权限，404：找不到，500：后端炸了，502：网关后面的服务炸了，504：上游超时
- if (!response.ok) {...}:正常路径：try->await fetch->成功->显示数据，异常路径：fetch失败->throw Error->catch->显示读取失败
- response.json():将Json格式的数据转换为javaScript对象
- export function PhoneDetectionSection() {...return(....)}:export是js/ts模块导出关键字，命名导出，导入这个函数时必须{xxx},名字严格匹配，可以多个。这个函数是整个页面，function XXX() {return (...)}这个是函数组件Function Component，最终返回JSX：<div>...</div>，React把这些东西渲染成网页
- const [form, setForm] = useState(defaultForm):用户当前输入了什么？
- const [phoneConfig, setPhoneConfig] = useState<PhoneConfig>(defaultPhoneConfig)：后端真实配置
- const [cameras, setCameras] = useState<AiVisionCamera[]>([])：当前摄像头列表
- const [loading, setLoading] = useState(false)：正在读取配置吗
- const [saving, setSaving] = useState(false)：正在保存吗     以上都是前端状态State
  > 总结：import 导入别人的模块，type：规定数据结构，useState：保存页面状态，async/await：异步操作，fetch：发送HTTP请求，JSON：前后端传输数据，try/catch：异常处理，HTTP状态码：200/401/404/500....
- shmName.trim():去掉字符串的前后空格
- source.find((camera) => ...)：.find()从数组里找第一个满足条件的元素
- source.find(...)?.name：？.name的意思是找到了就读取 .name，没找到就返回 undefined，别报错。
- ?.name || fallback || key：如果找到了name，用name，否则，如果fallback有值用fallback，否则直接用shmName
- const loadConfig = async（cameName = form.cmeraName,source:AiVisionCamera[]=cameras）:根据摄像头，去后端读取该摄像头的手机检测配置，cameraName=form.cameraName默认参数
> JS里面一共8个falsy（假值）：false： 布尔假，0：数字0，-0：负0，On：BigInt零，""：空字符串，null，underfined，NaN，只要上面任意，if（值）条件就不进入，视为不成立
- setLoading(true)：进入loading状态，后面是setMessage('正在读取当前配置...')，网页从读取配置变成正在读取当前配置：这就是：业务状态->React State->UI变化
- try {const payload = await requestPhoneConfig(shmName)：真正调用fetch，完整关系是：loadConfig（）->requestPhoneConfig()->fetch()->HTTP GET->/api/ai/pjone-config/{shm_name},后端返回的JSON文件得到payload，setPhoneConfig(payload.detect_phone)：保存后端原始业务配置
- 为什么 phoneConfig 和 form 两份数据都要留：原因是phoneConfig可能有很多页面没有修改的字段，用户修改一个值时其他配置全部没有了，这个的原因就是保留没有动过的数据，只修改优化真正动过的数据
- error instanceof Error：这个错误是不是标准的JavaScript Error对象，? error.message：如果是得到原始错误信息，: String(error)：如果不是，强制转换字符串
- finally {setLoading(false)}：finally：无论成功或者失败，都一定执行
- const loadCameras = async () => {}：函数内部getAiVisionCameras()这个函数在src/lib/aivision.ts里面封装好的，他最终发送：GET /api/aivision/cameras，const items = await getAiVisionCameras()
- setCameras(items)：保存当前系统所有摄像头
- const enabledCamera =items.find(camera=> isAivisionFeatureEnabled(camera,'detect_phone')):从所有摄像头里找到第一个已经开启detect_phone的摄像头，isAivisionFeatureEnabled这个函数定义在aivision.ts里，看这台摄像头有没有开启这个功能，所以：camera->functions->detect_phone->true/false(典型的前端API工具函数封装)
- ？.:enabledCamera?.shm_name || '':找到了shm_name返回shm_name,没找到就是underfined，因为使用了?.，所以没找到不会报错
- if (items.length === 0){}：后端读取摄像头列表，如果没有摄像头，告诉用户先加摄像头
- if (!nextCamera) ：系统有摄像头，但是没有一个开启了手机检测，setMessage('当前手机使用检测未添加摄像头...')
- setForm((prev) => ({...prev,cameraName：''})):...prev:把原来所有字段复制过来，cameraName: ''覆盖旧值
- ...prev:展开运算符 Spread Operator
> 为什么不直接form.cameraName='',因为react通常要求不直接修改state原对象，而是创建一个新对象然后交给setForm（不可变状态更新 Immutable Update）
- await loadConfig(nextCamera, items)：获取当前shm_name摄像头的配置
- useEffect(()=>{....},[])：[]加依赖数组：Dependency Array：空数组表示组件第一次挂载时执行一次：用户进入手机检测页面->PhoneDetectionSection 创建->useEffect执行->loadCameras():页面一打开就自动有摄像头和参数
  > 完整首屏启动：用户点击"手机识别检测"->React创建PhoneDetectionSection->useEffect()->loadcameras()->GET /api/aivision/cameras->找到detect_phone=true摄像头->nextCamera={shm_name}->loadConfig({shm_name})->requestPhoneConfig(shm_name)->GET /api/ai/phone-config/{shm_name}->后端返回手机检测配置->setPhoneConfig()->toForm()->setForm()->React重新渲染->用户看到：摄像头、ROI、置信度、IoU...
- window.addEventListener('aivision:cameras-changed',refreshCameras):浏览器全局监听一个叫aivision：cameras-changes的事件
- window.dispatchEvent(new Event('aivision:cameras-changed'))：广播：摄像头发生变化了，然后立即refreshCameras()，最终loadCameras()
  > 假如在另外一个组件：新增摄像头，成功之后dispatchEvent"aivision:cameras-changed"，然后：手机检测界面->收到事件->重新获取摄像头：这个叫发布订阅Publish-Subscribe（事件驱动思想）
- return () =>window.removeEventListener(...)：cleanup清理函数，组件创建：return () =>window.removeEventListener(...)，组件销毁：removeEventListener（资源生命周期管理）
- const addCameraToFeature = async (cameraName: string) => {：添加摄像头到手机检测，setCameraAction(true)：告诉页面正在操作
- const payload = await requestPhoneConfig(cameraName)：先把旧配置读取回来
-  method: 'PUT'：之前requestPhoneConfig(cameraName)没有写method，fetch默认GET读取配置，现在加上method：'PUT'，表示更新配置（GET：查询，PUT：修改，POST：创建，DeLETE：删除）
-  'Content-Type': 'application/json'：告诉后端，我发给你的Body是JSON
-  body: JSON.stringify(...)：HTTP的body是js对象，经过stringify转化为Json文本
-  { ...payload.detect_phone, enable: true}:...payload.detect_phone保留所有东西，enable: true：只把enable改为true：所以这个把添加摄像头到手机检测的本质不是新建东西，而是把该摄像头的detect_phone.enable改成true
-  await refreshCameraList() await loadConfig(cameraName)：PUT修改服务器列表->重新GET摄像头列表->重新GET手机配置->用服务器真实数据刷新页面（而不是我PUT成功了。那本地肯定也是正确的）
-  body: JSON.stringify({ ...payload.detect_phone, enable: false})：停用这台摄像头的手机检测功能，摄像头本身仍然存在（项目核心数据模型是一个摄像头多个检测算法的开关）
- saveconfig（）：用户点击保存时调用的是这个函数。整个saveconfig的流水线：用户修改输入框->form保存用户输入->点击'保存'->saveConfig()->检查摄像头->检查权限->解析ROI JSON->字符串转数字->检查数字是否合法->组装nextConfig->JSON.stringify()->HTTP PUT->后端->保存成功->重新GET一遍->页面刷新成服务器的真实配置
- if (!shmName) {..}:前置校验
- if (!isAdmin) {..}:权限校验，const { isAdmin } = useAuth()来自登录系统。典型的前端权限控制(前端权限控制只能改善用户体验，真正的权限校验还必须由后端做。因为懂技术的人可以绕开网页，自己发送：PUT /api/ai/phone-config/camera_01)
- form.roiPos：这个还是字符串，roiPos = JSON.parse(form.roiPos || '[]')通过json.parse()转化成javaScript数组：JSON.parse():字符串->JS对象，JSON.stringify():JS对象->JSON字符串
- let roiPos: unknown:在JSON.parse()之前，程序不知道用户输入的是什么，所以TypeScript这里先说，我不知道这个东西是什么，这是比any更安全的写法
- 对于ROI输入，程序写了两层校验：1、是不是合法JSON，2、是不是数组。ROI数组格式：[][][],第一个中括号是多个ROI，第二个是一个ROI的多个点，第三个是一个点的[x,y]
- normalizePhoneRoiPos():兼容旧写法，保证其是三位数组（数据规范化）Normalization
- const xxx =Number(form.xxx)：字符串转数字
- Number.isFinite():检查是不是真正有效的数字。还有就是我们是前后端双重校验：前端只检查是不是数字，python后端的Pydantic还要检查范围贵不规范（全栈开发的重要思想：永远不要完全相信客户端的输入）
- nextConfig：即将发送给Python后端的最终配置对象，form是用户输入状态。nextConfig是整理，转换，校验完成后的业务对象：form（页面数据）-(转换)>nextConfig(接口数据)-(json)>Python
- value: {...phoneConfig.value,CONF_THRESHOLD:confidenceThreshold}:第二层spread：后端的有些字段页面没有输入框，直接替换值会把其他值丢掉，...phoneConfig.value先保留，CONF_THRESHOLD: confidenceThreshold只覆盖新值
  > 旧配置：字段a,b,c，用户只改b，复制旧配置，覆盖b（Merge / 合并配置。）
  > 一些配置值：MIN_CONSECUTIVE_HITS：连续命中几帧后才确认，REQUIRE_PERSON_ASSOCIATION：true：手机不能单独出现，最好还得和人相关联（误报过滤）
- setSaving(true)：进入保存状态，保存按钮可以被禁用（防重复提交。）
- await requestPhoneConfig(shmName, {method: 'PUT',headers: {'Content-Type':'application/json'},body:JSON.stringify(nextConfig),})：前端保存动作的终点，HTTP Request Body。
- const reloaded = await requestPhoneConfig(shmName)：保存完成后再GET一次服务器，后端可能做校验，默认值补全，数据规范化，数据库转换，字段修正，所以需要重新读取，服务器保存的最终结果才是真实的结果
- setPhoneConfig(reloaded.detect_phone)setForm(toForm(reloaded))：重新覆盖前端状态，服务器真实配置->toForm()->form->输入框
- const items = await refreshCameraList()：重新刷新摄像头列表
   > 完整链路：用户修改Input->onChange->setForm()->form保存字符串->用户点击保存->onSave={saveConfig}->检查cameraName->检查isAdmin->JSON.parse ROI->Number()字符串转数字->Number.isFinite()检查数字->构造nextConfig->JSON.stringify()->PUT /api/ai/phone-config/{shm_name}->后端—>成功->重新GET->setPhoneConfig() setFrom->React重新渲染
   > 前端发到/api/ai/phone-config/{shm_name}，Vite会把他转到：unified_backend :5000
# Python后端
## unified_backend
- 前端发送的/api/ai/phone-config/{shm_name},Vite会把他转给unified_backend :5000。
- 统一后端的通配代理路由：@app.api_route("/api/ai/{path:path}",methods=["GET","POST","PUT","PATCH","DELETE","OPTIONS"])：{path:path}表示后面不管跟什么路径，都接住，然后转发给Aivision
- body = await request.body()：HTTP请求到python后，python就在这里读取，await proxy_client.send(...)：发送上游HTTP
## AiVision：7862
- AiVision本身是FastAPI(...),它注册了：from api.compat import router as compat_router; app.include_router(compat_router),所以真正的PUT接口在aivision/api/compat.py里面
- @router.put("/api/ai/phone-config/{shm_name}") def put_phone_config(...):
  - @router.put()是FastAPI的路由装饰器，意思是：只要收到这个 URL 的 PUT 请求，就执行下面这个函数。（路由Routing）
  - {shm_name}自动变成Python参数（这个叫路径参数Path Parameter）
- payload: dict[str, Any]：自动把前端JSON解析成Python字典
- _user: dict = Depends(require_admin) ： 前端权限不可行，后端必须在检查(依赖注入 Dependency Injection)
- Depends(xxx):FastAPI非常核心的东西，表示当前函数需要一个“xxx服务”，FastAPI 帮我拿过来。
- def _put_legacy_config（legacy_name,shm_name,payload,feature_servuce）:正式处理手机配置：legacy_name = "phone"："phone":"detect_phone",旧名字phone——（映射新架构名字）>detect_phone
- spec = get_spec("detect_phone"),此时SPEC== FeatureSpec(name="detect_phone",consumer_cls=PhoneConsumer,orm_class=PhoneConfigORM,config_schema=PhoneConfigSchema,required_models=("yolo_phone",),)，spec里面其实带着整套信息：detect_phone:phoneConsumer,phoneConfigORM,PhoneConfigSchema,yolo_phone(features-first架构一个关键的地方)
- normalized =normalize_legacy_payload(spec,payload):项目以前可能有不同的格式，兼容层这里统一成{"enable": true,"value": {...},"alarm": {...}}
- spec.config_schema：对手机检测来说是PhoneConfigSchema，PhoneConfigSchema.model_validate(normalized)
- CONF_THRESHOLD: float = Field(0.7, ge=0, le=1)：Pydantic数据校验，后端的schema
- result = feature_service.save_config(spec,shm_name,validated)：校验通过后进入Service层，进入AiVision/features/_base/service.py（API层，负责HTTP，service层，负责业务操作，ORM/Database负责数据库，不要把所有东西写在一个FastAPI函数里）
- save_config():
    - 先找摄像头camera = self._get_camera_by_key(session,shm_name)
    - 然后查询手机配置检测表(ORM 对象关系映射)：cfg = (session.query(spec.orm_class).filter_by(camera_id=camera.id).first())：spec.orm_class就是PhoneConfigORM（SELECT*FROM 手机检测配置表 WHERE camera_id=?）
      - Python不直接手写SQL，而是session.query(...),背后由SQLAIchemy生成SQL
- 数据库没有记录就创建一条cfg = spec.orm_class(camera_id=camera.id)     session.add(cfg)，属于upsert风格的处理：查，有就更新，没有就创建
- 把配置分别保存到ORM对象：new_enable = bool(payload.get("enable", False))，cfg.apply_value_dict(payload.get("value") or {})，cfg.apply_alarm_dict(payload.get("alarm") or {})
- session.flush()：把当前ORM的变更同步给数据库事务，with session_scope() as session:退出上下文后，session_scope会完成commit，所以：Python ORM——>flush——>数据库事务——>commit——>真正保存
- config_cache.reload()：数据库变了，把运行时配置缓存重新加载（因为通常是数据库->Cache->实时算法）
- config_event_bus.publish(ConfigChangeEvent(change_type=f"{spec.name}_config_updated",..)):手机检测变成detect_phone_config_updated，这时候已经进入事件驱动架构
- 项目是一个发布/订阅系统（publisher发布者->Event Bus->subscriber订阅者）保存配置的时候FetureConfigService->publish->detect_phone_config_update
- 谁订阅了这个事件？Orchestrator  config_event_bus.subscribe_wildcard(self._on_config_change)：所有配置变更事件我都监听，所以detect_phone_config_update()->Orchestrator._on_config_change()
- Orchestrator根据feature找Consumer（数据处理模块）：事件detect_phone_config_updated被拆成feature = detect_phone，action  = config_updated，代码最终：consumer.update_config(cam_id)
- PhoneConsumer为什么可以立即拿到新参数：因为它继承了：BaseFunctionConsumer，里面有：def update_config
   >热更新Hot Reload： 数据库保存->全局Config_cache刷新->发布事件->Orchestrator->PhoneConsumer.update_config->consumer自己的config_cache更新，这样就不需要重启AiVision
  > 保存配置的完整闭环：
      - React Input：页面输入框，用户修改参数
      - form ： 前端表单对象，JS对象，存着页面上填好的全部配置
      - saveConfig()：点击保存按钮触发的函数，前端业务函数
      - JSON.stringify()：JSON对象转JSON字符串，HTTP接口只能传文本JSON
      - HTTP PUT ： 发起PUT请求，把配置提交出去
      - Vite:3000：前端页面地址（Vite代理转发）
      - unified_backend:5000：统一网关服务
      - _proxy()：网关内部的代理转发函数，把请求转发给AI视觉服务（类似 Vite Proxy），这是后端网关的代理，不能前端开发代理，网关做请求中转
      - HTTP PUT：网关重新发起一起PUT请求，送到真正的业务服务
      - AiVision:7862
      - auth middleware：鉴权中间件，请求先走到中间件，校验token，判断用户是否登录
      - require_admin ： 权限校验，判断这个用户是不是管理员，middleware、require_admin：只做拦截校验，不改业务数据，校验通过才放行往下走
      - put_phone_config()：接口路由处理函数，真正接收这个配置请求的入口handler
      _ normalize_legacy_payload()：老数据兼容函数，把前端传过来的数据做格式调整，比如老版本接口传的字段格式不对，部分字段缺失，在这里统一整理成规范格式，适配新旧数据
      - PhoneConfigSchema+Pydantic校验:Schema:数据结构定义；Pydantic做强校验：字段类型、范围、必填项检查
      - FeatureConfigService.save_config()：业务服务层，封装保存配置逻辑
      - SQLAlchemy ORM：Python ORM框架，不用手写SQL，把对象转成数据库语句
      - MySQL：数据真正落地
      - config_cache.reload()：刷新全局共享参数，把MySQL最新配置加载进内存
      - config_event_bus.publish()：推送配置变更事件，通知系统内所有模块
      - Orchestrator：编排调度器，接收事件，分发通知给对应业务模块
      - PhoneConsumer.update_config()：手机检测模块，收到事件通知，更新模块自己的本地化部署。下一帧直接使用新参数
  > api/compat.py:负责：HTTP、权限、参数入口
  > Schema PhoneConfigSchema:负责：数据格式，范围校验
  > Service FeatureConfigService:负责业务读写
  > ORM PhoneConfigORM：负责Python与数据库表交互
  > Database MySQL
  > Cache+Event:负责 让实时系统立即拿到新配置
  > Consumer PhoneConsumer:负责真正的视觉业务逻辑（PhoneConsumer手机业务处理器）
## PhoneConsumer手机检测业务处理器
  AiVision/features/detect_phone/consumer.py
  - 实时链路：摄像头采集进程->最新完整画面->BaseFunctionConsumer主循环->裁剪ROI->PhoneConsumer._process()->手机模型+人体模型->空间关联->简单跟踪->连续次数->持续时间->是否报警
  - 父类负责拿帧，裁ROI，控制处理频率，最后才调用子类的_process(),PhoneConsumer接收到的是处理好的roi_list
  - 父类实现了：读取最新帧、ROI剪裁、线程主循环、推理任务调度、告警触发、结果画面缓存、配置热更新
  - self._tracks: dict[int, dict[int, dict]] = {}：手机检测自己多维护两个状态：保存跨帧的手机跟踪状态，大致结构就是：摄像头1下有手机轨迹1，手机轨迹2，每个手机轨迹下又有第一次出现的时间，最近出现的时间，bbox，已连续识别次数
  - self._next_track_id: dict[int, int] = {}：负责给手机对象编号，这里不是ByteTrack，是自己写的一套非常轻量的跨帧匹配逻辑（通过IoU或者中心点距离，判断两帧是不是一个手机）
  - def _process(self,roi_list,camera_id):给一台摄像头当前所有的ROI，返回每个ROI是否应该报警，这里的ROI是由父类的_get_roi()函数得到（职责分离）
  - cfg = self._value_config( self.config_cache.get(camera_id, {}):读取运行配置（之前有前端->MySQL->config_cache.reload()->PhoneConsumer.update_config()）
  - self.dispatch_inference_sync(...):将ROI交给yolo_phone模型去检测，等结果回来：dispatch：把任务推到推理模块，inference：模型推理，sync：当前业务逻辑等待结果返回
  - PhoneConsumer自己没有执行YOLO,而是：PhoneConsumer线程->提交推理任务->模型推理进程->YOLO->结果返回
  - task_meta={}:告诉模型进程，推理置信度和IOU
  - 手机检测完还要跑一次人体检测self.dispatch_inference_sync(...)里面传人体检测模型，其中timeout=0.6：表示最多等待0.6秒
  - _results_by_roi()：整理模型返回值，做list-dict索引化，后续按ROI编号快速拿结果：phone_results.get(roi_index)
  - roi_offsets：将ROI坐标映射回原图，因为roi在原图上剪裁了之后，检测到的手机是相对ROI内部的坐标，所以需要offset=(ox,oy)
  - candidates = self._qualified_phone_detections(...):手机检测框，1、判断是不是靠近人体区域，2、和上一帧做匹配，3、判断连续次数和持续时间，这里是真正的业务判断逻辑
  - 最终报警条件：只要有一个ROI需要报警就报警
  - self._prune_tracks():清理过期轨迹，超过TTL，将当前手机轨迹从_tracks中删除。
  总结：
    收到当前摄像头的ROI->读取这台摄像头的配置->调用yolo_phone得到手机框->如果开启人体关联，调用yolo_human得到人体框->遍历每个ROI->把手机框和人体框拿出来->过滤不合理的手机框->和历史手机轨迹进行匹配->统计出现次数和持续时间->只要有一个手机被 confirmed->这个 ROI = 报警->清理消失太久的手机轨迹->返回结果
## dispatch_inference_sync
  - 给本次推理任务生成唯一编号
  - 一个ROI会被包装成一个任务：ROI图片、摄像头是谁、哪个业务请求、哪个模型、模型参数全部包装到一个Python dict里
  - self.infer_dict[model_name].put(task,timeout=0.02):进入多进程队列：multiprocessing.Queue，也就是跨进程任务通道，就是PhoneConsumer线程，包装成任务，给到yolo_phone队列，得到这样一个YOLO模型进程
  - IPC：进程间通信：不同进程拥有独立内存空间：multiprocessing.Queue进行进程间通信
  - AiVision/platform/inference/base_worker.py：模型进程一直在运行，有没有新任务?有就拿出来执行YOLO，没有就继续等。所以：PhoneConsumer-put()>任务队列-get()>YOLO进程
  - platform/inference/workers/yolo_detector.py：真正执行YOLO的地方，完整关系：原图，裁ROI，Queue，YOLO：这样既可以降低计算量，也可以减少无关区域误检
  - target_queue.put(result,timeout=0.5)：PhoneConsumer线程->infer_queue->YOLO模型进程->model.predict()->result_queue->PhoneConsumer线程。所以有两条Queue，一条是infer_queue,一条是result_queue
  - outcome = self.result_queue.get(...):PhoneConsumer在等结果
  - - 会检查request_id
> 总结PhoneConsumer：拿到ROI->包装成task->放进YOLO_Phone Queue->YOLO模型进程->Queue取任务->model.predict()->整理bbox/confidence->放入result_queue,  ->PhoneConsumer线程——>result_queue取结果——>检查request_id——>拿到手机框
> 线程像是一个AiVision主程序里的不同工作流，进程则是隔离出来的独立运行单元（中间不能随便共享普通Python变量），使用multiprocessing.Queue来传任务喝结果
## 检测逻辑
  - 先检查bbox是不是完整的[x1,y1,x2,y2]
  - ROI坐标还原成原图坐标
  - 做人体关联：手机框附近没有人会被过滤掉，人体左右扩展约18%，手机中心点在人体的扩展区域就认为手机和这个人有关联（这是第一种判断）
  - 第二种判断，框有明显重叠，计算交集面积/手机框的面积，大于0.08就认为相关（计算手机有多少比例落进人体区间）
  - 通过以上判断后开始跟踪，判断当前手机是不是上一轮看到的手机，两个判断条件：
    - 1、两个框重叠程度够高：iou大于最小iou
    - 2、两个框的中心距离<=0.6
  - 匹配不到就说明出现了一个新的手机目标，中间消失太久，也会重新计算，不能把相隔很久的两个检测硬当成连续行为。
  - 检测次数hits>=3,并且持续时间duration>=1.2秒才会确认
  - 最后把全部信息塞回结果
>三层检测逻辑：1、YOLO识别到手机，2、几何关系：手机中心点是否在人体扩展范围内，以及手机落在人体区域的比例，3、时间逻辑：判断是否连续出现以及持续的时间
  - 过期轨迹删掉：_prune_tracks
## 报警逻辑
  - PhoneConsumer._process()得到检测判断，post_process()画检测框，_handle_alarm()做告警处理，以上都是父类做的
  - post_process():负责把结果画回视频，先拿完整视频帧，遍历刚才检测出来的手机，把ROI坐标重新变回原图坐标，OpenCV画框，画完以后保存这台摄像头最新的检测后画面，之后API和前端监控页可以取这里的最新结果画面。
  - _handle_alarm():父类统一提供：
## 知识：
  - .find():数组里找元素，
  - ？.:安全访问可能不存在的对象
  - ...prev:展开/复制对象
  - setState(prev=>...)：React更新旧状态
  - useEffect(...,[]):页面第一次加载执行
  - addEventListener:监听事件
  - cleanup：页面销毁时释放资源
  - GET：查询
  - PUT：修改
  - JSON.stringify:js对象转json
  - Content-Type：告诉服务器数据格式
  - try/catch/finally:异常与资源状态处理
## 串通
页面打开：
- react
- HTTP
- 后端
- JSON返回
- React State
- 页面更新
