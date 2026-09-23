# FastAPI集成SQLAIchemy ORM操作数据库
FastAPI是目前流行的Python异步Web框架，而SQLAIchemy是Python生态中最强大、最成熟的ORM(对象映射关系)工具，将两者结合，可以快速构建出高性能、类型安全且易于维护的Web应用
```
  HTTP请求 -> FastAPI路由 -> Pydantic校验 -> CRUD层 -> SQLAlchemy Session ->MySQL
```
安装
```python
pip install sqlalchemy pymysql cryptography
```
- sqlalchemy:ORM框架
- pymysql：MySQL驱动
- cryptography: pymysql连接加密时需要
# 数据库连接配置
统一管理引擎、Session与依赖注入
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker,DeclarativeBase

# MySQL连接
SQLALCHEMY_DATABASE_URL =(
  'mysql+pymysql://root:123456@127.0.0.1:3308/db_fastapi_pro?charset=utf8mb4'
)

# 创建引擎
engine = create_engine(
  SQLALCHEMY_DATABASE_URL,
  pool_pre_ping=True,         # 连接池自动检测断线重连
  pool_recycle=3600,          # 每小时回收连接，避免MySQL 8小时超时
)

#Session工厂
SessionLocal = sessionmaker(autocommit=False,autoflush=False,bind=engine)

# SQLAlchemy声明式基类
class Base(DeclarativeBase):
  pass

def get_db():
  db = SessionLocal
  try:
    yield db         # 把db交给FastAPI
  finally：
    db.close()
```
- get_db()通过yield实现依赖注入，与Depends用法相同
- 每个HTTP请求使用独立Session，避免线程/协程间共享连接
- yield作用是把get_db()变成一个生成器函数，FastAPI的Depends(get_db)会按先拿到资源然后执行接口再清理资源这个顺序运行

- # 定义ORM模型（建表）
- 定义与t_item表映射的ORM类
```python
from datatime import datetime
from sqlalchemy import String,Float,DateTime,Integer,Text
from sqlalchemy.orm import Mapped,mapped_column
from database import Base

# 物品表ORM模型，对应数据库表t_item
class ItemModel(Base):
  __tablename__='t_item'
  id:Mapped[int] = mapped_column(Integer,primary_key=True,autoincrement=True,commit='主键ID')
  name: Mapped[str] = mapped_column(String(50),nullable=False,comment='物品名称')
  price： Mapped[float] = mapped_column(Float,nullable=False,comment='价格')
  description: Mapped[str | None] = mapped_column(Text,nullable=True,comment='描述')
  create_at: Mapped[datetime] = mapped_column(DateTime,default=datetime.now,comment='创建时间')
  update_at: Mapped[datetime] = mapped_column(Datetime,default=datetime.now,onupdate=datetime.now,comment='更新时间')
```
## 自动建表
  在应用启动时执行Base.metadata.create_all(),SQLAlchemy会根据模型自动创建t_item表
```python
from database import engine,Base
from xxx import ItemModel    #必须导入,否则模型不会注册

# 创建所有表（已存在的表不会重复创建）
Base.metadata.create_all(bind=engine)
```

# Pydantic模型（请求/响应）
字段校验参考fastapi基础.md里的·Itemcreate
```python
from datetime import datetime
from pydantic import BaseModel,Field,ConfigDict

# 创建物品时的请求体
class ItemCreate(BaseModel):
  name：str = Field(...,min_length=2,max_length=50,description='物品名称，2~50个字符')
  price: float = Field(...,gt=0,le=9999.99,description='价格，必须大于0')
  description：str | None = Field(None,max_length=200,description='可选描述')

# 更新物品时的请求体
class ItemUpdate(BaseModel):
  name: str | None =Field(None,min_length=2,max_length=50)
  price: float | None = Field(None,gt=0,le=9999.99)
  description：str | None =Field(None,max_length=200)

# 返回给客户端的数据结构
class ItemOut(BaseModel):
  id:int
  name: str
  price: float
  description: str | None = None
  created_at: datetime
  updated_at: datetime

  # from_attributes=True:允许Pydantic直接从Sqlalchemy ORM对象读取属性，无需手动转字典
  model_config = ConfigDict(from_attributes=True)
```
# CRUD操作
CRUD对应SQL：
- db.add()+db.commit()     增
- db.query().filter().first()  查单条
- db.query().offset(),limit(),all()  查多条
- setattr() + db.commit()  改
- db.delete()+db.commit()   删
```python
from sqlalchemy.orm import Session
from xxx import ItemModel
from xxx import ItemCreate，ItemUpdate

# ------------- 1、增 ---------------
def create_item(db:Session,item:ItemCreate) -> ItemModel:
  db_item = ItemModel(
    name=item.name
    price=item.price
    description=item.description
)
  db.add(db_item)
  db.commit()
  db.refresh(db_item)
  return db_item

# ----------- 2、查 -------------
def get_item(db:Session,item_id:int) -> ItemModel | None:
  return db.query(ItemModel).filter(ItemMoel.id == item_id).first()

def get_items(db:Session,skip:int=0,limit:int=10) -> list[ItemModel]:
  return db.query(ItemModel).offset(skip).limit(limit).all()

# ----------- 3、改 -----------------
def update_item(db: Session,item_id:int, item：ItemUpdate) -> ItemModel | None
  db_item = get_item(db,item_id)
  if not db_item:
    return None
  # model_dump:把模型对象转成字典，exclude_unset=True保证字典只有传过来的字段
  update_data = item.model_dump(exclude_unset=True)
  for field,value in update_data.items():
    setattr(db_item,field,value)
  db.commit()
  db.refresh(db_item)
  return db_item

# ------------- 4、删 ----------------------
def delete_item(db:Session,item_id:int) -> bool:
  db_item = get_item(db,item_id)
  if not db_item:
    return False
  db.delete(db_item)
  db.commit()
  return True
```

# FastAPI路由集成
将CRUD暴露为REST API
```python
from typing import Annotated
from fastapi import APIRouter,Depends,HTTPException,Query
from sqlalchemy.orm import Session

from crud import item as item item_crud
from database import get_db
from xxx import ItemCreate,ItemUpdate,ItemOut

router = APIRouter(prefix='/items',tags=['物品管理'])
DbSession = Annotated[Session,Depends(get_db)]

@router.post('/',response_model=ItemOut,summary='新增物品')
def create_items(item:ItemCreate,db:DbSession):
  return item_crud.create_item(db,item)

@router.get('/',response_model=list[ItemOut],summary='物品列表(分页)')
def list_item(
  db:Dbsession
  skip: int = Query(0,ge=0,title='跳过条数')
  limit: int = Query(10,ge=1,le=100,title='返回条数条数')
)
  return item_crud.get_items(db,skip=skip,limit=limit)

@router.get('/{item_id}'，response_model=ItemOut,summary='查询单个物品')
def read_item(item_id:int,db:Dbsession):
  db_item=item_crud.get_item(db,item_id)
  if not db_item:
    raise HTTPException(status_code=404,detail='Item项不存在')
  return db_item

#更新
@router.put('/{item_id}',response_model=ItemOut,summary='更新物品')
def update_item(item_id:int,item:ItemUpdate,db:Dbsession):
  db_item=item-crud,update_item(db,item_id,item)
  if not db_item:
    raise HTTPException(status_code=404,detail='Item项不存在')
  return db_item

@router.delete('/{item_id}',summary='删除物品')
def remove_item(item_id:int,db:DbSession):
  success =Item_crud.delete_item(db,itemf_db)
  if not success:
    raise HTTPException(status_code=404,detail='Item项不存在')
  return {'message':'删除成功'，'item_id':item_id}
```
# 注册路由启动
```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from database import engine,Base
from xxx import ItemModel
from routers.item import router as item_router

#这段代码是FastAPI的应用生命周期管理，用来在服务启动时做初始化，在服务关闭时做清理
@asynccontextmanager
async def lifespan(app: FastAPI):
  # 应用启动时自动建表
  Base.metadata.create_all(bind=engine)
  yield
  engine.dispose() # 释放数据库连接池

app = FastAPI(title='FastAPI + SQAlchemy示例',lifespan=lifespan)

#注册路由：
app.include_router(item_router)

@app.get('/')
async def root():
  return {'message':'你好，FastAPI+SQLalchemy'}
```
