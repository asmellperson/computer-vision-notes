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
  1、生成OpenAPI模式：当使用Python的类型提示（Type Hints）定义API路径、参数和请求体时，FastAPI会在后台自动提取这些信息。并生成一个符合OpenAPI规范的JSON或YAML文件，这个文件是一份关于所有端点的结构化蓝图
  2、渲染成交互式界面：基于这份OpenAPI蓝图，FastAPI内置了两种广受欢迎的用户界面来将其渲染成交互式文档，可以直接在应用地址后加上特定路径来访问他们
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
