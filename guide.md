# Game Cloud 使用指南
## SDK简介

Game Cloud 帮助开发人员快速构建多人联网实时互动的游戏产品。
Game Cloud 由SDK和一组应用服务组成。 开发人员可以利用这个SDK
开发可以访问应用服务的产品来快速构建具有联网功能的游戏产品。

SDK又分为户端的SDK(简记为"GP_Client")和服务端SDK ("GP_Server")。
本文通过示例，介绍SDK的具体用法。

## 获取 SDK

### GP_Client/SDK的组成
~~~~~~
clientsdk/docs/guide  // it's me

clientsdk/clientsdk/GP_Client  // include files
	
clientsdk/clientsdk/Demo  // demo project with cocos2d-x 3.6
clientsdk/clientsdk/ccb   // ui for Demo
clientsdk/clientsdk/android/lib // libs for android
clientsdk/clientsdk/ios/lib  // libs for ios

~~~~~~

## 使用步骤

### iOS 接入

#### 导入库

在XCode中，选PROJECT的TARGETS选中"Demo-mobile"，然后在Build Settings中，找到
"Search Paths",在Header Search中，加入 include的路径。

同样，在"Library Search Paths"中，加入 lib的路径。

### android 接入
#### 用cocos 或ant 打包
* 修改 AndroidManifest.xml增加权限

```xml
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

* 修改Application.mk

增加

``` makefile
## step 1
APP_CPPFLAGS += -DACE_HAS_CUSTOM_EXPORT_MACROS=0 -D__ACE_INLINE__ -DACE_AS_STATIC_LIBS  -DTAO_AS_STATIC_LIBS

```

* 修改Android.mk

``` makefile
### step 2
LOCAL_SRC_FILES := hellocpp/main.cpp \
##                   ../../Classes/AppDelegate.cpp \
##                   ../../Classes/HelloWorldScene.cpp
FILE_LIST := $(wildcard $(LOCAL_PATH)/../../Classes/*.cpp)
LOCAL_SRC_FILES += $(FILE_LIST:$(LOCAL_PATH)/%=%)

LOCAL_C_INCLUDES := $(LOCAL_PATH)/../../Classes
LOCAL_C_INCLUDES += $(LOCAL_PATH)/../../../
PRJ_ROOT := $(LOCAL_PATH)/../../..
### step 3
## {{ lib
##### step 3.1

LOCAL_LDLIBS := -L$(PRJ_ROOT)/android/lib

#### step 3.2
LOCAL_LDLIBS +=-lGP_Client
LOCAL_LDLIBS +=-lGP_Application
LOCAL_LDLIBS +=-lGP_World_Factory
LOCAL_LDLIBS +=-lGP_Common
LOCAL_LDLIBS +=-lTAO_CosNaming
LOCAL_LDLIBS +=-lTAO_Utils
LOCAL_LDLIBS +=-lTAO_Svc_Utils
LOCAL_LDLIBS +=-lTAO_Messaging
LOCAL_LDLIBS +=-lTAO_CodecFactory
LOCAL_LDLIBS +=-lTAO_PI
LOCAL_LDLIBS +=-lTAO_BiDirGIOP
LOCAL_LDLIBS +=-lTAO_PortableServer
LOCAL_LDLIBS +=-lTAO_AnyTypeCode
LOCAL_LDLIBS +=-lTAO_Valuetype
LOCAL_LDLIBS +=-lTAO
LOCAL_LDLIBS +=-lACE
## lib}}

```

   * 用cocos 编译打包

~~~~~
cd  ./clientsdk/Demo/
ls -al
drwxr-xr-x  14 android  staff   476 May 26 09:50 .
drwxr-xr-x  11 android  staff   374 May 26 16:19 ..
-rw-r--r--   1 android  staff    76 May 26 09:26 .cocos-project.json
-rw-r--r--   1 android  staff  5026 May 26 09:26 CMakeLists.txt
drwxr-xr-x  38 android  staff  1292 May 26 16:19 Classes
drwxr-xr-x  13 android  staff   442 May 26 09:26 Resources
drwxr-xr-x   3 android  staff   102 May 26 09:50 bin
drwxr-xr-x  17 android  staff   578 May 26 09:26 cocos2d
drwxr-xr-x  23 android  staff   782 May 26 16:40 proj.android
drwxr-xr-x  10 android  staff   340 May 26 09:26 proj.android-studio
drwxr-xr-x   5 android  staff   170 May 26 09:26 proj.ios_mac
drwxr-xr-x   3 android  staff   102 May 26 09:26 proj.linux
drwxr-xr-x  12 android  staff   408 May 26 09:26 proj.win32
drwxr-xr-x   6 android  staff   204 May 26 09:26 proj.win8.1-universaldrwxr-xr-x  14 android  staff   476 May 26 09:50 .
drwxr-xr-x  11 android  staff   374 May 26 16:19 ..
~~~~~

~~~~
$ cocos run -p android
~~~~

   * 用ant打包

  进入 Demo/proj.android目录

~~~~
cd ./proj.android

./build_native.py

./ant debug install
~~~~


#### 用eclipse

#### 用android studio


## 代码集成概述

### 基本步骤


1. 设置本地存储的可写路径

2. 初始化SDK

3. 实现User类 （My_User.{h,cpp})，最好实现为单体
（可以不重载任何函数）


4. 实现Controller类 (My_Controller.{h,cpp}) 最好实现为单体
   
5. 定义交互的数据类型

6. 实现数据的显示

5. 实现数据的发布和订阅


###  代码示例
1. 包含SDK头文件

SDK内部实现做到了跨平台的工作，为不同平台的提供了一致的文件，这减少开发人员对多同平支持的工作量。

//todo

2. 设置本地存储的可写路径


3. 创建用户和控制器

4. 初始化控制器


### 代码集成详解


*  初始化Game_Cloud

``` cpp
  World_Factory* wf = World_Factory::instance ();
  int retval = wf->init_game_cloud(
                                   HOST_NAME,
                                   2809,
                                   "World_Factory",
                                   "001");
```
	World_Factory作为一个工厂类，除了创建用户和控制器之外，它还提供了一个初始化的功能。

``` cpp
class World_Factory
{
	// ...
	
	int init_game_cloud (
	const char* address,
	unsigned short port,
	const char* world_id,
	const char* world_loc);

// ...
};

```

*  订阅应用的状态

	控制器初始化后，SDK内部会向服务器发起会话请求，如果成功，应用端会与服务器建立连接，之后，服务器与应用双方都能够进行通讯，定点与传统的客户端发起请求，服务端作一个简单应答不同。

	示例中的

	Welcome_Layer

	从 "GP_Client::App_Listener"
	派生，同时重载
``` cpp
void on_status (int status);
```

	根据参数status，即可获知当前的连接状态，示例中，通过 另一个函数
``` cpp
void update_ui_status (int status)
```
	控制界面UI元素在不同状态下的显示外观。


* 初始化用户或初始化游客

	头文件

```cpp
class World_Factory
{
//...
	int create_user (const char* account_name,
									 const char* account_type,
									 const char* passwd);
									 
	int init_user (const char* user_id,
								 const char* passwd,
								 const char* nickname);

    int init_guest (const char* user_id);
//...
};
```
	使用示例：

```cpp
 World_Factory* wf = World_Factory::instance ();
  
  std::string uid = Local_Storage::instance ()->get_string_value ("user_id");
  int retval = wf->init_guest (uid.c_str());
 
```
* 获取房间列表(app_list)

	异步调用：查找游戏房间

```cpp
  Data_Manager* dm = Game_Controller::instance ()->data_manager();
  Game_Data* data = dm->find_data_by_id(kApp_List);
  this->subject(data);
  
  // get room list
  try
  {
    User* user = World_Factory::instance ()->get_user ();
    user->get_app_list ("a","key","demo_service");
  }
  catch (const std::exception& ex)
  {
    CCLOG ("Lobby_Layer::onEnter() exception: %s",ex.what());
  }
 ```
 异步调用的返回：

``` cpp
void My_User::on_get_app_list_complete (
	const char* app_id,
	const String_List& app_list)
{
  Data_Manager* dm = Game_Controller::instance ()->data_manager();
  String_List_Data* data = (String_List_Data*) dm->find_data_by_id(kApp_List);
  if (data)
  {
    data->value (app_list);
  }
 
}

```

反回的结果会更新代号为kApp_List的游戏数据。

使 Lobby_Layer作为观察者订阅kApp_List数据。

``` cpp
class Lobby_Layer
// ...
:public GP_Client::Observer
{
public:
virtual void upadte (const GP_Client::Subject* subject);
};
```

用观察者订阅主题：


``` cpp
void Lobby_Layer::onEnter ()
{
//...
 Data_Manager* dm = Game_Controller::instance ()->data_manager();
  Game_Data* data = dm->find_data_by_id(kApp_List);
  this->subject(data);
//...
}
```



* 进入游戏房间

``` cpp
 User* user = World_Factory::instance ()->get_user ();

 user->enter_into_app("appid", "appkey","demo_service",name.c_str());

```

* 安全退出

   使用示例
```cpp
     World_Factory* wf = World_Factory::instance ();
	 wf->shutdown ();

 ```

* 发送消息到服务端

在游戏主场景中，玩家触摸了屏幕，人物则前往屏幕所在的点，然后把这个点以及玩家的状态发送到服务端。

```cpp
void TerrainWalkThru::onTouchesEnd(
  const std::vector<cocos2d::Touch*>& touches, cocos2d::Event* event)
{
//...
   My_User::instance ()->move_to( collisionPoint.x,
       collisionPoint.y,
       collisionPoint.z,
       angle,
       _player->_headingAxis.y);
//...

}
```

  对消息编码，为简明，本例用的json

``` cpp
void My_User::move_to(float x, float y, float z,float a,float b)
{
  CCLOG ("My_User::move_to (%f,%f,%f)",x,y,z);
  Document d;
  d.SetObject();
  rapidjson::Value move;
  move.SetObject();
  move.AddMember("x",x,d.GetAllocator());
  move.AddMember("y",y,d.GetAllocator());
  move.AddMember("z",z,d.GetAllocator());
  move.AddMember("a",a,d.GetAllocator());
  move.AddMember("b",b,d.GetAllocator());

  d.AddMember("move",move,d.GetAllocator());
  StringBuffer buffer;
  Writer<StringBuffer> writer(buffer);
  d.Accept(writer);
  
  unsigned int cmd_id = CMD_GAME;
  const char* msg = buffer.GetString();
  try
  { 
     this->send_data(cmd_id,msg, strlen(msg)+1);
  }
  catch (const std::exception& ex)
  {
     // something wrong
  }
  CCLOG ("msg==> [%s]",msg);
}	
```

* 处理服务端发送的消息
  * 定义主题

``` cpp
class Move_Msg
{
public:
  Move_Msg (float x = 0,
            float y = 0,
            float z = 0,
            float a = 0,
            float b = 0,
            const char* uid = "")
  : x_ (x)
  , y_ (y)
  , z_ (z)
  , a_ (a)
  , b_ (b)
  , uid_ (uid)
  {
    
  }
  float x_;
  float y_;
  float z_;
  float a_;
  float b_;
  std::string uid_;
};

typedef Game_Data_T<Move_Msg> Move_Msg_Data;

```

  * 定义网络消息编号

``` cpp
enum
{
	kData_Base = 0x00001000,
	kChat_Data,
    kApp_List,
    kMove_Msg, // <---- this is my cmd_id
};
```
  * 观察者订阅放题并处理消息

``` cpp
class TerrainWalkThru;

class Move_Msg_Observer
: public Observer
{
  typedef Observer Base;
  typedef TerrainWalkThru UI;
public:
  Move_Msg_Observer (Subject* s, UI* ui);
  virtual void update (const Subject* s);
private:
  UI* ui_;
};

```

* 解析消息格式

``` cpp
int Game_Controller::parse_data(const char* uid,
                                const rapidjson::Document &d)
{
  if (!d.IsObject())
  {
    return -1;
  }
  
  if (d.HasMember("move") && d.IsObject())
  {
    float x = (float) d["move"]["x"].GetDouble();
    float y = (float) d["move"]["y"].GetDouble();
    float z = (float) d["move"]["z"].GetDouble();
    float a = (float) d["move"]["a"].GetDouble();
    float b = (float) d["move"]["b"].GetDouble();
    CCLOG (" Game_Controller::parse_data :%s  ===> (%f,%f,%f)",uid,x,y,z);
    Move_Msg v (x,y,z,a,b,uid);
  
    
    Data_Manager* dm = this->data_manager();
    Move_Msg_Data* data = (Move_Msg_Data*) dm->find_data_by_id(kMove_Msg);
    data->value (v);
    
  }
  return 0;
}

```

* 处理显示

``` cpp
void TerrainWalkThru::on_move(const char *uid,
                              float x, float y, float z,
                              float a,float b)
{
  //get user by id
  Player* p = getChildByName<Player*> (uid);
  if (p)
  {
    Vec3 collisionPoint (x,y,z);
  
    p->_headingAngle = a;
    p->_headingAxis.y = b;
    p->_targetPos = collisionPoint;
    p->forward();
  }
  else
  {
    // create other user
    create_other(uid,x,y,z);
  }
}

```

## 服务端 SDK (SDK GP_Server) 的使用
### 服务端SDK简介

GP_Server为开发者提供了一个简洁又功能强大的框架。

简洁：用户只需要重写几个函数，实现具体业务逻辑即可。

强大：
* 自注册、自注销功能
* 热交换功能： 不停进程完成系统升级替换的功能
* 弹性伸缩功能： 用户可以灵活开启多个服务实现同一功能的负载均衡，也可以用多个服务实现一个“大”服务的逻辑部分
其它系统级的功能
* 负载平衡
* 出错检测和出错恢复
* 服务质量配置

GP_Server 将被编译成动态库，然后由一个同一的加载器加载


### 支持的平台

* Linux
Ubuntu 12.04, 14.04 

* Windows



