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
app = FastAPI()
@app.get("/")
async def root():
  return{'message': "Hello World"}

@app.get("/hello/{name}")
async def say_hello(name: str):
  return{'message': f"Hello{name}"}
```
