---
title: ginTodoList——基于gin和gorm的待办小清单
date: 2023-05-04 20:21:53
tags:
- Golang
- Project
- MySQL
- Gin
- Web
categories:
- [Golang]
- [后端开发]
---

# 前言
[ginTodoList]()是一款基于Vue.js、Golang、gin、gorm的的待办小清单，项目可通过前后端分离的方式进行部署，并按照web开发的规范对项目的功能进行了划分。项目的[demo](http://120.24.232.79:8080/#/)演示效果如下：

最近在整理该[项目](https://github.com/charfole/ginTodoList)，也一并将其同步到博客当中以作记录。项目学习自[该课程](https://www.bilibili.com/video/BV1gJ411p7xC/?vd_source=1f061ced4e9d8953f513321c4a44f897)，并在原基础上增加了记录待办清单的创建日期功能。下面是项目的详细文档。

![image-20230114192014784](https://charfole-blog.oss-cn-shenzhen.aliyuncs.com/image/image-20230114192014784.png)

<!--more-->

## 1. 项目部署

- 下载源代码并切换目录

  ```shell
  git clone
  cd ./ginTodoList
  ```

- 前端部署（请确保本地的8080和9090端口能使用）

  ```shell
  # 进入目录
  cd codeOfFrontend
  # 依赖安装
  npm install
  # 前端项目启动
  npm run serve
  ```

- 数据库配置

  需要环境中有MySQL数据库，同时数据库的用户名、密码、端口号和数据库名称需要与下面的配置相对应。下面提供一种快速使用[docker](https://www.runoob.com/docker/ubuntu-docker-install.html)安装MySQL8.0.19的方法：

  ```shell
  # 在本地的33306端口运行一个名为mysql8019，root为用户名，密码为mysql1234的MySQL容器环境
  docker run --name mysql8019 -p 33306:3306 -e MYSQL_ROOT_PASSWORD=mysql1234 -d mysql:8.0.19
  
  # 启动一个MySQL客户端，连接上面的MySQL容器，密码为上一步指定的密码mysql1234
  docker run -it --network host --rm mysql mysql -h127.0.0.1 -P33306 --default-character-set=utf8mb4 -uroot -p
  
  # 创建项目用到的ginTodoList数据库，也可以是其他数据库，但需要修改配置文件
  CREATE DATABASE ginTodoList DEFAULT CHARSET=utf8mb4;
  ```

- 后端服务配置

  ```shell
  # 打开conf文件夹下的config.ini文件
  # 下面的信息和
  [mysql]
  ; 你的数据库用户名
  user = root
  ; 你的数据库密码
  password = mysql1234
  ; 你的IP（一般输入127.0.0.1即可）
  host = 127.0.0.1
  ; 你的数据库端口号
  port = 33306
  ; 你的数据库名称
  db = ginTodoList
  ```

- 后端部署

  ```shell
  # 返回项目根目录
  cd ..
  # 项目启动
  ./ginTodoList conf/config.ini
  # 如改变了代码则需要重新编译
  go build
  ```

## 2. 项目架构

**ginTodoList**

**├── codeOfFrontend** // 前端部分代码

**├── conf** // 配置文件夹

│  └── config.ini // 配置文件

**├── controller** // 控制器：负责处理请求与调用逻辑

│  └── controller.go

**├── dao** // dao (Data Access Object) 层：负责控制数据库

│  └── mysql.go

**├── models** // 模型层：负责与数据库进行交互，实现ORM中的逻辑

│  └── todo.go

├── **routers** // 路由层：负责注册路由并调用控制器的函数

│  └── routers.go

└── **setting** // 配置器：根据配置文件对应用中的服务进行配置

  └── setting.go

**├── main.go** //应用主入口

├── ginTodoList // 应用的可执行文件

├── go.mod

├── go.sum

## 3. 项目各模块解析

项目主要为前后端分离架构，前端由Vue.js和Element UI实现，后端主要通过gin实现业务逻辑、gorm实现数据库操作。后端部分可以分为配置文件（conf）、控制器（controller）、数据持久层（dao）、模型层（models）、路由层（routers）、设置文件（setting）。

1. 配置文件（conf）：配置文件夹，存储MySQL配置文件。

2. 设置文件（setting）：定义结构体，读入配置文件中的参数，从而对应用中的各项服务进行配置，当前应用仅包含MySQL服务。

   ```go
   // 定义结构体，读入配置文件中的参数，从而对应用中的各项服务进行配置
   // 当前应用仅包含MySQL服务
   package setting
   
   import (
   	"gopkg.in/ini.v1"
   )
   
   var Conf = new(AppConfig)
   
   // AppConfig 应用程序配置
   type AppConfig struct {
   	*MySQLConfig `ini:"mysql"`
   }
   
   // MySQLConfig 数据库配置
   // 设置MySQL的用户名、密码，数据库名称、IP地址、端口号
   type MySQLConfig struct {
   	User     string `ini:"user"`
   	Password string `ini:"password"`
   	DB       string `ini:"db"`
   	Host     string `ini:"host"`
   	Port     int    `ini:"port"`
   }
   
   // 将conf文件夹下的配置文件config.ini映射到Conf中
   func Init(file string) error {
   	return ini.MapTo(Conf, file)
   }
   ```

3. 数据持久层（dao）：负责初始化数据库连接与关闭数据库连接。

   ```go
   // dao(Data Access Object)：负责初始化数据库连接与关闭数据库连接
   package dao
   
   import (
   	"fmt"
   	"ginTodoList/setting"
   
   	"github.com/jinzhu/gorm"
   	_ "github.com/jinzhu/gorm/dialects/mysql"
   )
   
   // 定义数据库对象
   var (
   	DB *gorm.DB
   )
   
   // 初始化MySQL数据库，指定数据库用户、密码、IP、端口号和数据库名称信息
   func InitMySQL(cfg *setting.MySQLConfig) (err error) {
   	dsn := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=utf8mb4&parseTime=True&loc=Local",
   		cfg.User, cfg.Password, cfg.Host, cfg.Port, cfg.DB)
   	DB, err = gorm.Open("mysql", dsn)
   	if err != nil {
   		return
   	}
   	return DB.DB().Ping()
   }
   
   // 关闭数据库
   func Close() {
   	DB.Close()
   }
   ```

4. 模型层（models）：与dao层中的数据库对象进行交互，负责实现数据库中某张表的增删改查等基本操作。通过gorm，将type与MySQL的某张表进行映射，从而通过操作type来操作数据表。

   ```go
   /*
   与dao层中的数据库进行交互，负责数据库中某张表的增删改查等基本操作
   通过gorm，将type与MySQL的某张表进行映射，通过操作type来操作数据表
   */
   package models
   
   import (
   	"ginTodoList/dao"
   )
   
   // Todo Model 默认映射到当前数据库中的todos表
   type Todo struct {
   	ID     int    `json:"id"`
   	Title  string `json:"title"`
   	Date   string `json:"date"`
   	Status bool   `json:"status"`
   }
   
   // 创建一个todo：直接传入一个结构体进行创建，有错误则返回
   func CreateATodo(todo *Todo) (err error) {
   	err = dao.DB.Create(&todo).Error
   	return
   }
   
   // 获取所有todo：传入todo结构体指针数组todoList，获取当前todos表中的所有数据
   // 同时更新todoList，有错误则返回
   func GetAllTodo() (todoList []*Todo, err error) {
   	if err = dao.DB.Find(&todoList).Error; err != nil {
   		return nil, err
   	}
   	return
   }
   
   // 获取某个todo：通过id获取某个todo事项，有错误则返回
   func GetATodo(id string) (todo *Todo, err error) {
   	todo = new(Todo)
   	if err = dao.DB.Debug().Where("id=?", id).First(todo).Error; err != nil {
   		return nil, err
   	}
   	return
   }
   
   // 更新某个todo：传入一个todo结构体指针，更新todos表中对应的todo项
   func UpdateATodo(todo *Todo) (err error) {
   	err = dao.DB.Save(todo).Error
   	return
   }
   
   // 删除某个todo：传入一个id，删除对应id的todo项
   func DeleteATodo(id string) (err error) {
   	err = dao.DB.Where("id=?", id).Delete(&Todo{}).Error
   	return
   }
   ```

5. 路由层（routers）：负责注册路由，引导请求到对应的控制函数中。

   ```go
   // 负责注册应用中的路由，并引导请求到对应的控制函数中
   package routers
   
   import (
   	"ginTodoList/controller"
   
   	"github.com/gin-gonic/gin"
   )
   
   func SetupRouter() *gin.Engine {
   	// 初始化路由引擎
   	r := gin.Default()
   	// 初始化v1路由组，用于处理对应的请求
   	v1Group := r.Group("v1")
   	{
   		// 操作待办事项
   		// 添加某一个待办事项
   		v1Group.POST("/todo", controller.CreateTodo)
   
   		// 查看所有的待办事项
   		v1Group.GET("/todo", controller.GetTodoList)
   
   		// 修改某一个待办事项
   		v1Group.PUT("/todo/:id", controller.UpdateATodo)
   
   		// 删除某一个待办事项
   		v1Group.DELETE("/todo/:id", controller.DeleteATodo)
   	}
   	// 注册好路由信息后，返回路由引擎
   	return r
   }
   ```

6. 控制器（controller）：负责实现控制路由中对应的各个函数，其中各个函数只调用具体的逻辑，而不进行实现。

   ```go
   //  控制器：负责实现控制路由中对应的各个函数，其中各个函数只调用具体的逻辑，而不进行实现
   /*
   	url      -->  controller  -->       models
   	请求来了  -->    控制器    -->   模型层的增删改查
   */
   package controller
   
   import (
   	"ginTodoList/models"
   	"net/http"
   
   	"github.com/gin-gonic/gin"
   )
   
   // 下面的操作主要逻辑为：接收并处理前端请求，调用models层中的增删改查基础功能进行处理
   // 创建一个待办事项
   func CreateTodo(c *gin.Context) {
   	// 定义一个结构体用于接收前端传来的信息
   	var todo models.Todo
   	// 将JSON信息绑定到结构体中
   	c.BindJSON(&todo)
   	// 调用CreateATodo()函数，添加一个todo事项到todos表
   	err := models.CreateATodo(&todo)
   	// 如有错误则返回错误信息
   	if err != nil {
   		c.JSON(http.StatusOK, gin.H{
   			"error": err.Error(),
   		})
   	} else {
   		// 无错误则以JSON形式返回todo信息给前端
   		c.JSON(http.StatusOK, todo)
   	}
   }
   
   // 获取所有待办事项
   func GetTodoList(c *gin.Context) {
   	// 调用模型层GetAllTodo()函数获取所有待办事项
   	todoList, err := models.GetAllTodo()
   	// 如有错误则返回错误信息
   	if err != nil {
   		c.JSON(http.StatusOK, gin.H{
   			"error": err.Error(),
   		})
   	} else {
   		// 无错误则以JSON形式返回todo列表给前端
   		c.JSON(http.StatusOK, todoList)
   	}
   }
   
   // 更新一个待办事项
   func UpdateATodo(c *gin.Context) {
   	// 获取当前id对应的待办事项
   	id, ok := c.Params.Get("id")
   	// 如果没有传过来id，则返回错误
   	if !ok {
   		c.JSON(http.StatusOK, gin.H{
   			"error": "无效的id",
   		})
   		return
   	}
   	// 调用GetATodo()函数，首先查询出该id对应的待办事项
   	todo, err := models.GetATodo(id)
   	// 有错误则返回
   	if err != nil {
   		c.JSON(http.StatusOK, gin.H{
   			"error": err.Error(),
   		})
   		return
   	}
   	// 将前端传来的json绑定到当前todo中
   	c.BindJSON(&todo)
   	// 调用UpdateATodo()函数更新todo事项
   	if err = models.UpdateATodo(todo); err != nil {
   		c.JSON(http.StatusOK, gin.H{
   			"error": err.Error(),
   		})
   	} else {
   		// 返回更新后的todo给前端（这里更新的主要是status字段，已完成或未完成）
   		c.JSON(http.StatusOK, todo)
   	}
   }
   
   // 删除一个待办事项
   func DeleteATodo(c *gin.Context) {
   	// 获取当前id对应的待办事项
   	id, ok := c.Params.Get("id")
   	// 如果没有传过来id，则返回错误
   	if !ok {
   		c.JSON(http.StatusOK, gin.H{"error": "无效的id"})
   		return
   	}
   	// 调用DeleteATodo()函数删除todo，删除成功则返回deleted，否则返回错误
   	if err := models.DeleteATodo(id); err != nil {
   		c.JSON(http.StatusOK, gin.H{"error": err.Error()})
   	} else {
   		c.JSON(http.StatusOK, gin.H{id: "deleted"})
   	}
   }
   ```