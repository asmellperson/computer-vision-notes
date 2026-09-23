# FastAPI
FastAPI是一个现代、高性能的Python Web框架，专为构建API而设计。基于标准的Python类型提示（Type Hints）构建，自2018年底由塞巴斯蒂安·拉米雷斯发布
- 官网：https：//fastapi.tiangolo.com
- 中文：https：//fastapi.tiangolo.com/zh/
解决传统框架Flask、Django在异步支持、开发效率和性能上的痛点，fastAPI的优势：
- 高性能：性能与NodeJS和Go等语言编写框架相当，是Python最快的框架之一，得益于底层基于Starlette（ASGI框架）和Pydantic（数据验证率），在TechEmpower的基准测试中，性能远超Flask和Django
- 极速开发：依赖于高度依赖类型提示的特性，减少了大量样板代码
- 更少错误：借助类型提示和Pydantic的自动数据验证，FastAPI能减少约40%由开发者引起的人为错误
- 自动生成文档：FastAPI最受欢迎的特性之一，基于OpenAPI标准，自动为API生成交互式文档，文档支持Swagger UI和ReDoc两种界面，可以在浏览器直接调用和测试API
- 强大的依赖注入系统：FastAPI包含一个极其易用但功能强大的依赖注入系统，依赖项本身也可以有依赖项，形成一个层次结构或依赖图，这一切由框架自动处理
- 内置安全功能：内置了对OAuth2、JWT（Json Web Tokens）等安全认证机制的支持
核心架构：Starlette + Pydantic
- starlette：一个轻量级的ASGI（异步服务器网关接口）框架，为FastAPI提供了强大的异步Web工具支持
- Pydantic: 一个数据验证和设置管理的库，利用Python类型提示进行数据验证、序列化和反序列化
FastAPI无内置ORM，需自行集成SQLAIchemy等（Django是有的），没有后台管理，Django内置后台管理，增删改查开箱即用
# 编写FastAPI Helloworld项目
```python
from fastapi import FastAPI
# 定义FastAPI实例
app = FastAPI()
@app.get("/")
async def root():
  # 返回json格式数据
  return{'message': "Hello World"}

# name：路径变量，比如我浏览器ip:端口/hello/jack，这个输出就变成Hellojack
@app.get("/hello/{name}")
async def say_hello(name: str):
  return{'message': f"Hello{name}"}
```
uvicorn： 基于ASGI（异步服务器网关接口）规范的Python Web服务器，相当于Python异步Web框架的运行引擎，负责处理底层从网络连接、HTTP请求解析和响应发送
- uvicorn main:app --reload（main是文件，app是实例，--reload保证修改了值后可以自动重启）

# FastAPI生成交互式API文档
  1、生成OpenAPI模式：当使用Python的类型提示（Type Hints）定义API路径、参数和请求体时，FastAPI会在后台自动提取这些信息。并生成一个符合OpenAPI规范的JSON或YAML文件，这个文件是一份关于所有端点的结构化蓝图        2、渲染成交互式界面：基于这份OpenAPI蓝图，FastAPI内置了两种广受欢迎的用户界面来将其渲染成交互式文档，可以直接在应用地址后加上特定路径来访问他们
  两大核心文档界面
  - Swagger UI（/docs）
    提供可视化的、可交互的界面，清晰列出所有API端点，请求方法，参数和响应模型，最大的亮点是支持“try it out”功能，可以直接在浏览器中填写参数并点击执行，向API发送真实请求并查看返回结果
  - ReDoc（/redoc）
    备选的API文档方案，界面风格与Swagger UI不同，更侧重于提供一份结果清晰、易于阅读的文档，非常适合用来作为API的参考手册

# 路由与参数(路径参数、查询参数、请求体参数)
路由：根据请求找对应代码的映射关系。本质是一张查找表，当客户端用方法A访问路径B时，服务器自动执行函数C。在Web开发中可以拆解成三个核心要素的绑定:
- 请求方法（GET、POST、PUT等）
- URL路径
- 处理函数
参数：
· 路径参数： URL中固定占位的动态变量，用于定位具体资源，用于GET、POST、PUT、DElETE等所有方法。例如获取ID为123的用户，修改ID为5的文章
- 查询参数：URL路径？之后的键值对，属于路径末尾的查询字符，用于过滤、排序或分页，主要是GET请求，例如：/user？age=1&page=1
- 请求体参数：放在HTTP请求体（Body）中的完整数据，通常为JSON格式。不在URL中，位于请求报文的独立数据区域。主要是POST、PUT、PATCH，要结合Pydantic。提交复杂数据，注册新用户，或编辑一篇文章的全文
```python
# 路径参数
@app.get("/user/{user_id}")
async def get_user(user_id:int):
  return{'user_id','name':f"用户_{user_id}"}

# 查询参数
@app.get("/items")
async def list_items(skip:int=0,limit:int=10):
  return{"skip":skip,'limit':limit,'items':['item1','item2']}

# 请求体参数
# 1、先要创建类作为请求体参数,要继承自pydantic.BaseModel
class ItemCreate(BaseModel):
  name:str
  price: float
# 2、创建删除一般使用post请求方法
@app.post('/items')
async def create_item(item:ItemCreate)：
  return {'message':'Item created','item':item}
```
# 参数校验（Path、Query、Field）
- gt/ge： 大于/大于等于
- lt/le: 小于/小于等于
- min_length/max_length: 字符串长度范围
- regex：字符串正则表达匹配
- min_items/max_items: 列表元素个数
- ...: 表示该字段必须提供（无默认值）
- default: 设置默认值
- title/；description：用于生成API文档
## 路径参数：使用Path
  默认情况下，FastAPI会根据类型注解进行简单类型转换（如int），可以借助Path增加校验规则：
  - 限定数值范围(gt、ge、lt、le)
  - 设置描述信息(用于OpenAPI文档)
  - 使用正则表达式（regex）校验字符串路径参数
```python
from fastapi import FastAPI,Path
app = FastAPI()
@app.get("/users/{user_id)}")
async def get_user(
  user_id:int = (Path...,title='用户ID'，description=‘必须是整数’，gt=0，le=1000)
):
  return{'user_id':user_id,'name',f'用户_{user_id}'}
```
## 查询参数：使用Query
查询参数是URL中？后面的键值对，通常可选，也可以选默认值，用Query可以：
- 设置默认值
- 校验最大最小，字符串长度
- 使用regex校验字符串格式
- 设置deprecated等
```python
from fastapi import FastAPI Query
app=FastAPI()

async def list_item(
  skip: int=Query(0,title='跳过条数'，ge=0.description='必须>=0'),
  limit: int=Query(10,title="返回条数"，ge=1，le=100，deacription='1~100之间')
):
  return {'skip':skip,'limit':'items'.'items':['item1','item2']}
```
## 请求体参数：使用Pydantic+Field
请求体通常使用Pydantic的BasModel定义结构，并在字段上使用Field添加校验：
- 默认值
- 字符串长度
- 数值范围
- 正则匹配
- 描述信息等
```python
from fastapi import FastAPI
from pydantic import BaseModel,Field

app = FastAPI()

class ItemCreate(BaseModel):
  name: str = Field(...,title='物品名称',min_length=2,max_length=50,description='2~50个字符')
  price: float = Field(...,title='物品名称'，gt=0，le=9999.99，description="必须大于0.最多9999.99")

@app.post('/items')
async def create_item(item: ItemCreate)
  return {'message':'Item created','item':item}
```

# Cookie和Header参数获取
1、Cookie是服务器发送到用户浏览器并保存在本地的一小块数据，服务器每次发起请求时，都会自动带上这张卡，让服务器认出你是谁。浏览器是以特殊的方式处理cookie，并在后台进行操作，因此他们不会轻易允许JavaScript访问这些cookie
- Annotated:把类型（Cookies）和元数据（Cookie()）绑在一起，是FastAPI推荐的写法
```python
class Cookies(BaseModel):
  session_id:str   # 必填
  fatebook_tracker: str | None  =None # 可选，没有则为None
  googall_tracker: str | None = None

@app.get('/cookies/')
async def read_cookies(cookies: Annotated[Cookies,Cookie()]):
  return cookies
```
2、HTTP请求头部（Header）是一组包含请求信息的键值对，用来描述HTTP请求的各种属性和特征
```python
class CommonHeader(BaseModel):
  hostname: str
  save_data: bool
  if_modified_since: str | None = None
  traceparent: str | None = None
  x_tag: list[str] = []
@app.get("/headers/")
async def read_header(headers: Annotated[CommonHeaders,Header()]):
  return headers
```

# 表单数据与模型
当需要接收表单字段而不是JSON时，可以使用Form。使用Form之前需要先安装python-multipart
```
pip install python-multipart --trusted-host pypi.tuna.tinghua.edu.cn 
```
```python
class FormData(BaseModeL):
  username: str
  password: str

@app.post("/login/")
async def login(data: Annotated[FormData,Form()])       # 从From取这个类
  return data
```

# 请求表达与文件
FastAPI支持同时使用File和Form定义文件和表单字段。文件类型有两种，分别是bytes整个文件一次性读进内存，和IploadFile文件流式上传，适合大文件
```python
@app.post('/files')
async def create_file(
  file: Annotated[bytes,File()]          # 整个文件一次性读进内存（bytes）
  fileb: Annotated[UploadFile,File()]    # 文件流式上传，适合大文件（文件夹，视频类的）
  token: Annotated[str,Form()]          
)：
  return {
  'file_size':len(file),      # 获取文件大小
  'token': token,             # 获取表单参数
  'fileb_content_type': fileb.content_type, # 获取文件类型
}
```
# 响应类型-返回文件格式
在FastAPI中，除了默认的JSON响应，还可以通过response_class参数直接返回特定的响应对象，来灵活地返回HTML、文件、纯文本、流媒体等各种类型的内容。主要有两种方式来指定非JSON的响应类型：
- 1、通过response_class参数声明： 在路由装饰器中使用response_class参数，可以声明该接口的响应媒体类型，这会在OpenAI文档中自动生成相应的说明
- 2、直接返回响应对象：在路径操作函数中，直接实例化返回一个具体的响应对象（如FileResponse、
StreamingResponse）。这种方式更加灵活，适用于需要动态决定响应类型的场景

响应类型清单：
  - JSONResponse: 默认响应类型，自动将字典、列表等Python对象转换为JSON格式
  - HTMLResponse：返回HTML内容，浏览器会将其渲染为网页
  - PlainTextResponse: 返回纯文本内容
  - FileResponse: 返回文件供下载或预览，异步读取整个文件，适合大小适中的文件
  - StreamingResponse：流式传输大文件，实时数据等，逐块读取和发送数据，内存占用低，适合大文件或生态生成内容
  - RedirectResponse：将请求重定向到另一个URL
  - ORJSONResponse：一个高性能的JSON响应类，使用orjson库进行序列化
```python
# 1、返回HTML内容实例：
@app.get('/home',response_class=HTMLResponse)
async def get_home():
  return"<h1>Hello,World!</h1>"

# 2、返回图片文件
@app.get('/file')
async def get_file():
  return FileResponse('./files/1.png')

# 3、返回流式文件
@app.get('/stream-file')
async def stream_large_file():
  file_path = './files/1.mp4'
  media_type,_ = mimetypes.guess_type(file_path)     # 获取文件类型

  def iterfile():
    with open(file_path,'rb') as file_like:
      # 每次读取1MB并yield出去
      while chunk := file_like.read(1024 * 1024):     # 1024*1024=1MB
        yield chunk     # yield是python中用来定义生成器（generator）的关键字

  filename = quote(os.path.basename(filepath))   #对文件名进行URL编码
  return StreamingResponse(
    iterfile(),
    media_type=media_type or 'application/octet-stream',
    headers={'Content-Disposition':f"attachment; filename*=UTF-8 {filename}'},

# 4、请求重定向
@app.get('/redirect')
async def redirect():
  return RedirectResponse(url='https://python222.java1234.com/')
)
```

# 响应模型-返回类型
在FastAPI中，响应模型指的是声明的、用于规定API接口返回数据应遵循的格式和结构的Pydantic模型。可以通过在路径操作函数的返回类型注解或装饰器的response_model参数来声明它。它的核心价值在于：
- 数据校验与安全保障：FastAPI会自动校验返回的数据是否符合模型定义，如果数据无效（例如缺少必填字段），FastAPI会返回服务器错误，而不是将错误数据返回给客户端，更重要的是，它会将输出数据限制并过滤模型中多定义的内容，可以避免返回敏感信息，对安全至关重要
- 自动生成API文档：响应模型会为API生成清晰的JSON Schema，并自动在Swagger UI等交互式文档中展示，方便前端或第三方开发者查看
- 数据序列化与过滤：FastAPI会使用Pydantic将返回数据（可以是字典，数据库对象等）自动序列化为符合模型的JSON格式，同时你可以利用模型的exclude，include等参数精细控制哪些字段出现在最终的响应中
```python
class Item(BaseModel):
  name:str
  price: float
  tax: float | None=None
#在装饰器中通过response_model指定
@app.post('/create_item/',response_model=true)
async def create_item(item: Item):
  #假设这里进行了数据库操作，然后返回了一个字典
  # FastAPI会使用Item模型来校验和过滤这个字典
  return {'name': item.name,'price':item.price,'tax':item.tax}
```
```python
# 用户返回信息不带密码，重新定义一个新的用户类UserOut作为返回类型
class UserIn(BaseModel):
  username: str
  password: str
  full_name: str|None=None
class UserOut(BaseModel):
  username: str
  full_name: str | None=None

@app.post('/user/',response_model=UserOut)
async def create_user(user: UserIn):
  return user
```

# 异常错误处理（HTTPException）& 响应状态码（status_code）
在FastAPI中，HTTPException是一个内置的异常类，用于在API处理过程中主动抛出HTTP错误响应，当业务逻辑遇到问题（如资源不存在，权限不足，参数无效等）时，可以抛出HTTPException，FastAPI会捕获它并自动将其转换为符合HTTP规范的JSON错误响应(包含状态码和详细信息)，有以下核心优势：
- 标准化错误响应：返回标准的HTTP状态码和错误信息，方便客户端处理
- 自动生成文档：异常信息会出现在OpenAPI文档中（如果声明了response），提高API的可理解性
- 与依赖注入等机制无缝集成：可以在依赖项、中间件等任何位置抛出
```python
# 参数：status_code: HTTP状态码。detail：错误描述信息。headers：可以添加自定义响应头
@app.get('/items/{item_id}')
async def read_item(item_id:int):
  if item_id < 1:
    # 抛出HTTPException，状态码400，并附带详细信息
    raise HTTPException(status_code=400,detail='id必须大于0')
  if item_id!=1:
    raise HTTPException(status_code=404,detail='Item项不存在')
  return {'item_id': item_id,'name':'Sample Item'}
```

# 中间件
中间件(middleware)是在HTTP请求到达路由处理函数之前，以及响应返回客户端之前，插入的一层可编程的逻辑。可以理解成请求/响应管道中的拦截器：
  - 请求阶段：从外到内依次执行，可对请求做校验，日志，鉴权等
  - 响应阶段：从内到外依次返回，可对响应做压缩，加Header，统一格式等
FastAPI基于Starlette实现，中间件机制与Starlett完全一致。关键概念：
- call_next: 调用下一个中间件或最终路由，必须await
- 洋葱模型：多个中间件向洋葱一样层层包裹，请求由外向内，响应由内向外
- ASGI： FastAPI是ASGI应用，中间件需要异步
注册顺序与执行顺序：后注册的中间件，在请求阶段先执行
- app.add_middleware(MiddlewareA)    第一个注册
- app.add_middleware(MiddlewareB)    第二个注册
- app.add_middleware(MiddlewareC)    第三个注册（最后注册）
使用@app.middleware('http')注解即可实现中间件
```python
@app.middleware('http')
async def my_middleware1(request,call_next):
  print('中间件，执行前')
  response = await call_next(request)          # 执行下一个中间件或执行路由方法
  print('中间件，执行后')

# 后定义的中间件先执行
async def my_middleware2(request,call_next):
  print('中间件，执行前')
  response = await call_next(request)          # 执行下一个中间件或执行路由方法
  print('中间件，执行后')
@app.get('/')
async def root()
  return {'message':'你好，FastAPI 3333'}

@app.get('/helloworld')
async def helloworld():
  return {'message':'你好，helloworld'}

# 浏览器请求http://地址/helloworld
# 控制台输出
    # 中间件，执行前
    # 中间件，执行前
    # 浏览器打印：你好，helloworld
    # 中间件，执行后
    # 中间件，执行后
```

# 依赖注入
依赖注入是一种设计模式，核心思想是将对象的创建与使用分离，通过外部注入的方式为组件提高所需的依赖，而不是在组件内部直接实例化，在FastAPI中，依赖注入系统可以在路径操作函数中声明自己需要声明资源(如数据库连接、用户认证、配置等)，FastAPI框架会自动准备这些资源并注入到函数中，有以下优势：
- 代码复用： 将通用逻辑抽取为可复用的依赖，避免重复代码
- 解耦： 业务逻辑与基础设施分离，降低组件间的耦合度
- 简化测试： 测试时可以轻松Mock依赖项，隔离被测代码，无需实际连接数据库或调用外部服务
- 类型安全： 结合Python类型注解，支持IDE静态检查，避免运行时类型错误
- 自动文档： 依赖参数为自动出现在Swagger API文档中
FastAPI使用Depends实现依赖注入，定义一个依赖函数（可以是普通函数或者是异步函数），然后在路径操作函数的参数中使用Depends(依赖函数)来注入。依赖函数和普通的路径操作函数几乎一样，可以接收路径参数、查询参数、请求体等，也可以使用async/await，它的返回值会通过参数传递给路径操作函数
```python
async def common_parameters(
  q:str|None=None,
  skip: int=0,
  limit: int =100
):
  return {'q':q,'skip':skip,'limit':limit}
@app.get('/items/')
async def read_items(commons: Annotated[dict,Depends(common_parameters)]):
  return commons

@app.get('/users/')
async def read_users():
  return 'user info!' 
```
