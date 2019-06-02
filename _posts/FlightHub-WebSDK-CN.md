---
title: FlightHub-WebSDK（中文）
date: 2019-05-20 16:00:16
tags: 文档
keywords:
description:
---
<script type="text/javascript" src="/js/src/bai.js"></script>

> 可通过 FlightHub WebSDK 对系统进行扩展和重构，搭建私有平台，从中挖掘商业价值。
[大疆司空官网](https://www.dji.com/cn/flighthub)
[大疆司空安装包、Pilot PE APP、使用说明书、安装辅助软件](https://www.dji.com/cn/flighthub/info#downloads)

大疆司空（FlightHub） WebSDK REST api 介绍
粗略翻译，仅供参考。

## Overview（概览）
### 关于 REST API
大疆司空 WebSDK REST API v3 官方文档提供了无人机、团队、驾驶员用户、飞行统计和直播流。

### 版本信息
版本 : 3.0.0

### URI 方案
Host : Server_Address
BasePath : /fhsdk/v3
Schemes : HTTP, HTTPS

### 功能分类

* User : 用户管理
* Team : 团队管理
* Drone : 无人机管理
* Flight Records : 飞行记录
* Flight Statistics : 飞行统计
* Live Streaming : 直播流
* Real-time Report: 实时报告

#### Consumes （content-type）

* application/json

### 如何使用

有两种用户类型的REST API：
1.**Application Server（应用服务器）** Application Server直接通过超级管理员权限控制FlightHub，Application Server必须带有两个Key ：**AppId** 和 **SignKey**.
2.**End User(终端用户)** 意思就是移动端（如:DJI Pliot）直接控制无人机并连接FlightHub平台，**End User(终端用户)** 必须使用账户(**account**)和密码(**password**)并且需要带着**Common AppId**(flighthub-pilot默认)权限和每个会话(session)中动态生成的**SignKey**

## Security（安全认证）
### 概览
Http Request请求的Headers中必须带有 ：**FH-AppId, FH-ReqId, FH-Ts, FH-Sign**
终端用户的Header中必须带有 **FH-Token**

#### 怎样认证请求（sign a request）
##### 一、获取签名秘钥

获取签名秘钥的伪代码如下：
```
SignKey1 = HMAC_SHA1(FH-Ts, SignKey)
MasterSignKey = HMAC_SHA1(FH-AppId, SignKey1)
```
HMAC_SHA1 表示一个以二进制格式返回输出的 HMAC-SHA1函数。
这个函数的第一个参数是数据信息，第二个参数是Key。
若生成SignKey1，应该使用**SignKey** 作为 hash key, FH-Ts 作为数据信息。
若生成签名秘钥，应该使用SignKey1作为 hash key,FH-AppId 作为数据。

##### 二、计算签名

```
Sign = std_base64(HMAC_SHA1(sign_data, MasterSignKey))
```

要签名的数据（**sign_data**）是整个请求主体的内容。在GET请求中sign_data是空字符串。
使用主签名密钥作为key来计算sign_data的签名。

最后的签名字符串是base64编码的（带填充的标准编码）字符串签名。

##### 三、HTTP request请求中添加签名
可以将签名字符串放入以下两个方法中的其中一个：

* HTTP header中命名 **FH-Sign**
* 查询字符串命名 **FH-Sign**

#### 代码简述

```
fhTs  := 1543478800840       //  自Unix时代以来的时间（毫秒）
appID := "flighthub-admin"
reqID := "reqXXXX"           //  用于跟踪的每个请求的唯一ID。

req.Header.Set("FH-AppId", appID)
req.Header.Set("FH-Ts",fhTs)
req.Header.Set("FH-ReqId", reqID)

key1 := HmacSha1([]byte(fhTs), []byte(key)) // hmacsha1结果是二进制格式，不是十六进制或base64编码的。
masterKey := HmacSha1([]byte(appID), key1)

var buf []byte
if req.Body != nil {
        buf, _ = ioutil.ReadAll(req.Body)
}
rc := ioutil.NopCloser(bytes.NewBuffer(buf))
req.Body = rc

req.Header.Add("FH-Sign", Sha1HashBase64(buf, masterKey)) //  放入fh符号头时使用base64编码哈希
```

### FH-AppId

App ID

类型 : apiKey
名称 : FH-AppId
位置 : HEADER

### FH-Token

Token ID

类型 : apiKey
名称 : FH-Token
位置 : HEADER

### FH-ReqId

用于跟踪的每个请求的唯一ID。

类型 : apiKey
名称 : FH-ReqId
位置 : HEADER

### FH-Ts

自Unix时代以来的时间（毫秒）

类型 : apiKey
名称 : FH-Ts
位置 : HEADER

### FH-Sign

签名

类型 : apiKey
名称 : FH-Sign
位置 : HEADER

## Resources ( 资源 )
### 身份验证
用户身份验证
#### 用户登录
```
POST /auth/login
```
**Parameters（参数）**

| **类型** |**名称**  |**描述**  | **方案**  |
| --- | --- | --- | --- |
| **FormData** |**account**（必须）  |用户账户  | string |
| **FormData** |**password**（必须）  |用户密码  | string |

**Responses（响应）**

| **HTTP编码** |**描述**  | **方案** |
| --- | --- | --- |
| **200** |成功响应  |  [Response 200](#Response200)  |
| **400** |            | 无 |
| **500** |            | 无 |

**<a name="Response200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#data_login) |

**<a name="data_login"></a>data**

| **名称** |**描述**  |**方案**  |
| --- | --- | --- |
| **account** |登录账户  |string  |
| **user_id** |用户id  |integer  |
| **nick_name** |用户的nick名称  | string |
| **token** |在会话中使用  | string |
| **signkey** | 用于请求中 |string  |
| **validity** | token过期时间，以Unix秒为单位  | integer |


**Consumes**

* application/x-www-form-urlencoded
* application/json

**Produces**

* application/json

#### 用户退出
```
POST /auth/logout
```
**Responses**

| Http编码 | 方案 |
| --- | --- |
|200  | 无 |
|400  | 无 |
|500  | 无 |


**Consumes**

* application/json

**Produces**

* application/json


#### 更新Token，扩展认证到期时间

```
POST /auth/updatetoken
```

**Responses**

| **HTTP 编码** | **描述**  | **方案** |
| --- | --- | --- |
| **200** |  成功响应  | [Response 200](#updatetoken_resp200)  |
| **400** |             | 无 |
| **500** |             | 无 |

**<a name="updatetoken_resp200"><a/>Response 200**

| **名称** |**描述**  |**方案**  |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  |string (string)  |
|**data**  |  | [data](#updatetoken_data)  |

**<a name="updatetoken_data"></a>data**

| **名称** |**描述**  |**方案**  |
| --- | --- | --- |
| **token** |会话中用的令牌  |string  |
| **signkey** |用于请求中的key  |string  |
| **validity** | 令牌过期时间，以Unix秒为单位  |integer  |


**Produces**

* application/json

### 用户
用户管理
#### 创建用户
```
POST /users
```
**Parameters**

| **类型**  | **名称** | **描述**  | **方案** |
| --- | --- | --- | --- |
| **Body**  | **user** | 用于创建用户  | [UserCreateModel](#UserCreateModel)  |

**Responses**

| **HTTP** **编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 成功响应  |  [Response 200](#users_resp200)   |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="users_resp200"></a>Response 200**

| **名称** | **描述**  | **方案**  |
| --- | --- | --- |
| **code**  |  | integer (int64) |
| **message** |**如：**""  | string (string) |
|  **data** |  |  [data](#users_data) |

**<a name="users_data"></a>data**

| 名称 |描述  |方案  |
| --- | --- | --- |
| **id** | 用户ID **如：**`"USERID_1"` |string  |

#### 获取所有用户
```
GET /users
```

**Parameters**

| **类型** |**名称**  |**方案**  |
| --- | --- | --- |
|**Query**  |**teamId**（非必须）  | integer |

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 成功响应  | [Response 200](#users_get_resp200)  |
| **400** |              | 无 |
| **500** |              | 无 |

**<a name="users_get_resp200"></a>Response 200**

| **名称** | **描述**  | **方案**  |
| ---  |  --- |     --- |
| **code**  |  | integer (int64) |
| **message** |**如：**""  | string (string) |
|  **data** |    | [data](#getusersdata)  |

**<a name="getusersdata"></a>data**

| **名称**  |**方案**  |
| --- | --- |
| **users**  | < [UserGetModel](#UserGetModel) > array |

#### 通过用户ID获取用户信息
```
GET /users/{id}
```
**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  |
| --- | --- | --- | --- |
| **Path**  | **id**（必须）  |用户ID,使用`self`更新当前会话用户 | string |

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 成功响应  | [Response 200](#get_userid_resq200)  |
| **400** |  | 无 |
| **500** |  | 无 |

**<a href="get_userid_resq200"></a>Response 200**

| **名称** | **描述**  | **方案**  |
| --- | --- | --- |
| **code**  |  | integer (int64) |
| **message** |**如：**""  | string (string) |
|  **data** |  | [UserGetModel](#UserGetModel) |

#### 修改用户信息
```
PUT /users/{id}
```
**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  |
| --- | --- | --- | --- |
| **Path**  | **id**（必须）  |用户ID,使用`self`更新当前会话用户 | string |
| **Body**  | **user**（非必须）  |用于更新的用户 | [UserUpdateModel](#UserUpdateModel) |

**Responses**

| **HTTP编码** |  **方案**  |
| --- | --- |
| **200** |   无 |
| **400** |   无 |
| **500** |  无 |

#### 删除用户

```
DELETE /users/{id}
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  |
| --- | --- | --- | --- |
| **Path**  | **id**（必选）  |用户ID,使用`self`更新当前会话用户 | string |

**Responses**

| **HTTP编码** |  **方案**  |
| --- | --- |
| **200** | 无 |
| **400** | 无 |
| **500** | 无 |

### 团队
团队管理

#### 创建团队
```
POST /teams
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  |
| --- | --- | --- | --- |
| **Body**  | **team**（可选）  |用于创建团队 | [TeamCreateModel](#TeamCreateModel) |

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 成功响应  |  [Response 200](#postTeamsResp200)  |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="postTeamsResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#postTeamsData) |

**<a name="postTeamsData"></a>data**

| **名称**  |**描述**|**方案**  |
| --- |--- | --- |
| **id**  | 团队ID **如：**`32` | integer |


#### 获取所有团队

```
GET /teams
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  |
| --- | --- | --- | --- |
| **Query**  | **showAncestorTeams**（非必须）  | 显示祖先团队的层次结构 | boolean |
| **Query**  | **showSubTeams**（非必须）  | 显示子团队层次结构 | boolean |

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 成功响应  | [Response 200](#getTeamResp200)  |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="getTeamResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#data_teams_get) |

**<a name="data_teams_get"></a>data**

| **名称**  |**方案**  |
| --- | --- |
| **teams**（必须）  | < [UserGetModel](#UserGetModel) > array |

#### 通过团队ID获取团队

```
GET /teams/{id}
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  |
| --- | --- | --- | --- |
| **Path**  | **id**（必须）  |团队ID | string |

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 成功响应  |  [Response 200](#getTeamIdResp200)  |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="getTeamIdResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [TeamGetModel](#TeamGetModel) |

#### 更新团队信息

```
PUT /teams/{id}
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  |
| --- | --- | --- | --- |
| **Path**  | **id**（必须）  |团队ID | string |
| **Body**  | **team**（非必须）  |要创建的团队 | [TeamUpdateModel](#TeamUpdateModel) |

**Responses**

| **HTTP编码** | **方案**  |
| ---      |  --- |
| **200** | 无  |
| **400** |   无 |
| **500** |   无 |

#### 删除团队

```
DELETE /teams/{id}
```

**Parameters**

| **类型** | **名称**  | **描述** | **方案** |
| --- | --- | --- | --- |
| **Path**  | **id**（必须） |团队ID | string |

**Responses**

| **HTTP编码** | **方案**  |
| --- |--- |
| **200** | 无  |
| **400** |   无 |
| **500** |   无 |

#### 添加团队成员

```
POST /teams/{id}/members
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  |
| --- | --- | --- | --- |
| **Path**  | **id**（必须）  |团队ID | string |
| **Body**  | **member**（非必须）  |参数 | [TeamMemberCreateModel](#TeamMemberCreateModel) |

**Responses**

| **HTTP编码** | **方案**  |
| --- |--- |
| **200** | 无  |
| **400** |   无 |
| **500** |   无 |

#### 获取团队成员

```
GET /teams/{id}/members
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  |
| --- | --- | --- | --- |
| **Path**  | **id**（必须） | 团队ID | string |

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 成功响应  | [Response 200](#getTIMResp200)  |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="getTIMResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#data_members)  |

**<a name="data_members"></a>data**

| **名称** | **方案**  |
| --- | --- | --- |
| **members**（必须） | < [TeamMemberGetModel](#TeamMemberGetModel) > array |

#### 修改团队成员属性

```
PUT /teams/{id}/members/{userId}
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  |
| --- | --- | --- | --- |
| **Path**  | **id**（必须）  |团队ID | integer |
| **Path**  | **userId**（必须）  |用户ID | integer |
| **Path**  | **member**（非必须）  |参数 | [TeamMemberUpdateModel](#TeamMemberUpdateModel) |

**Responses**

| **HTTP编码** | **方案**  |
| --- | --- |
| **200** |  无 |
| **400** |  无 |
| **500** |  无 |

#### 删除团队成员

```
DELETE /teams/{id}/members/{userId}
```

**Parameters**

| **类型** | **名称**  | **描述**  | **方案**  |
| --- | --- | --- | --- |
| **Path**  | **id**（必须）  |团队ID | integer |
| **Path**  | **userId**（必须）  |用户ID | integer |

**Responses**

| **HTTP编码** | **方案**  |
| --- | --- |
| **200** | 无  |
| **400** |  无 |
| **500** |  无 |

### 无人机
无人机管理
#### 通过团队ID获取无人机

```
GET /drones
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  |
| --- | --- | --- | --- |
| **Query**  | **teamId**（非必须）  |团队ID | integer |

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 成功响应  | [Response 200](#droneResp200)  |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="droneResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#data_get_drones) |

**<a name="data_get_drones"></a>data**

| **名称** | **方案**  |
| --- | --- | 
| **drones**（必须） | < [DroneGetModel](#DroneGetModel) > array |

#### 修改无人机信息
```
PUT /drones/{sn}
```
**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  |
| --- | --- | --- | --- |
| **Path**  | **sn**（必选）  |无人机序列号 | string |
| **Body**  | **drone**（非必须）  |要创建的无人机 | [DroneEditModel](#DroneEditModel) |

**Responses**

| **HTTP编码** |  **方案**  |
| --- | --- |
| **200** |  无  |
| **400** |  无 |
| **500** |  无 |


#### 删除无人机信息
```
DELETE /drones/{sn}
```

**Parameters**

| **类型** | **名称**  | **描述**  | **方案**  |
| --- | --- | --- | --- |
| **Path**  | **sn**（必选）| 无人机序列号 | string |

**Responses**

| **HTTP编码** |  **方案**  |
| --- | --- |
| **200** |  无  |
| **400** |  无 |
| **500** |  无 |


#### 无人机绑定团队

```
POST /teams/{id}/drones
```

**Parameters**

| **类型** | **名称**  | **描述**  |**方案**  |
| --- | --- | --- | --- |
| **Query**  | **teamId**（非必须）  |       | integer |
| **Body**  | **member**（非必须）  |参数 | [DroneBindPara](#DroneBindPara) |

**Responses**

| **HTTP编码** |  **方案**  |
| --- | --- |
| **200** |  无  |
| **400** |  无 |
| **500** |  无 |

#### 无人机从团队解除绑定

```
DELETE /teams/{id}/drones/{sn}
```

**Parameters**

| **类型** | **名称**  | **描述**  | **方案**  |
| ---  | --- | --- | --- |
| **Path**  | **sn**（必须）  | 无人机序列号 | string |
| **Query**  | **teamId**（非必须）  | | integer |

**Responses**

| **HTTP编码** |  **方案**  |
| --- | --- |
| **200** |  无  |
| **400** |  无 |
| **500** |  无 |

### 飞行记录
#### 通过团队ID,用户ID,无人机序列号 获取飞行记录

```
GET /flightrecords
```

**Parameters**

| **类型** | **名称**  | **描述**  | **方案**  | **默认** |
| --- | --- | --- | --- | --- |
| **Query**  | **endTs**（非必须）  |以Unix毫秒为单位的结束时间。默认为当前时间 | integer | |
| **Query**  | **limit**（非必须）  | 返回数目 | integer | 20|
| **Query**  | **offset**（非必须）  | 偏移量 | integer | 0 |
| **Query**  | **sn**（非必须）  |无人机筛选条件 | string | |
| **Query**  | **startTs**（非必须）  |以Unix毫秒为单位的开始时间。默认提前1天 | integer | |
| **Query**  | **teamId**（非必须）  |团队筛选条件 | integer | |
| **Query**  | **userId**（非必须）  |用户筛选条件 | integer | |

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 飞行记录列表  | [Response 200](#getflightrecordsResp200)  |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="getflightrecordsResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#data_flightrecords) |

**<a name="getflightrecordsResp200"></a>data**

| **名称** | **方案**  |
| --- | --- |
| **flightRecords**（非必须） | < [FlightGetModel](#FlightGetModel) > array |
| **drones**（非必须） | [PaginationModel](#PaginationModel) |

#### 按记录ID获取详细记录

```
GET /flightrecords/{id}
```

**Parameters**

| **类型** | **名称** | **描述**  |**方案**  |
| --- | --- | --- | --- |
| **Path**  | **id**（必须） | 飞行记录ID| string |

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 响应成功  | [Response 200](#getflightrecordsidResp200)  |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="getflightrecordsidResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
| **code**  |  | integer (int64) |
| **message**  |**如：**""  | string (string) |
| **data**  |  | [data](#data_flightrecords_id) |

**<a name="data_flightrecords_id"></a>data**

| **名称** | **方案**  |
| --- | --- |
| **summary**（非必须） | [FlightSummaryModel](#FlightSummaryModel) |
| **recordPoints**（非必须） |< [FightRecordPointModel](#FightRecordPointModel) > array  |

#### 删除飞行记录

```
DELETE /flightrecords/{id}
```

**Parameters**

| **类型** | **名称** | **描述**  |**方案**  |
| --- | --- | --- | --- |
| **Path**  | **id**（必须）  | 飞行记录号 | string |


**Responses**

| **HTTP编码** |  **方案**  |
| --- | --- |
| **200** |  无  |
| **400** |  无 |
| **500** |  无 |

###  飞行统计

#### 获取飞行统计，按照团队ID和飞行ID分组

```
GET /statistics/flightrecords
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  | 
| --- | --- | --- | --- | --- |
| **Query**  | **droneSn**（可选）  | 无人机筛选条件 | string | 
| **Query**  | **endTs**（可选）  |以Unix毫秒为单位的结束时间。默认为当前时间 | integer | 
| **Query**  | **startTs**（可选）  |以Unix毫秒为单位的开始时间。默认提前1天 | integer | 
| **Query**  | **teamId**（可选）  |团队筛选条件 | integer | 
| **Query**  | **userId**（可选）  |用户筛选条件 | integer | 

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 响应成功  | [Response 200](#sfrResp200)  |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="sfrResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#data_statistics_flightrecords) |

**<a name="data_statistics_flightrecords"></a>data**

| **名称** | **方案**  |
| --- | --- |
| **statistics**（必须） | [FightStatisticsModel](#FightStatisticsModel) |

### 直播流

#### 获取直播流状态
```
GET /drones/{sn}/stream
```

**Parameters**

| **类型** | **名称**  | **描述**  | **方案**  | 
| --- | --- | --- | --- |
| **Path**  | **sn**（必选）  | 无人机序列号 | string | 

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 响应成功  | [Response 200](#gdssRes200)  |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="gdssRes200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#data_drones_stream) |

**<a name="data_drones_stream"></a>data**

| **名称** | **描述** | **方案**  |
| --- | --- | --- |
| **status**（必须） | 直播流状态 | boolean |

#### 获取播放直播流的地址

```
GET /drones/{sn}/stream/play
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  | 
| --- | --- | --- | --- | --- |
| **Path**  | **sn**（必选）  | 无人机序列号 | string | 

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 响应成功  | [Response 200](#getdsspResp200) |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="getdsspResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#data_stream_play) |

**<a name="data_stream_play"></a>data**

| **名称** | **方案**  |
| --- | --- | --- |
| **temp**（必须） |[rtmp](#dronesrtmp) |

**<a name="dronesrtmp"></a>rtmp**

| **名称** | **描述**  |**方案**  |
| --- | --- | --- |
| **playUrl**（非必须） |播放流URL **如：**`"rtmp://10.61.91.204/fhstream/SIM-0005?type=play&wowzatokenhash=e60c14af7c9b2a8c56f7734823d43f5acb1830c3&wowzatokenendtime=1536071501"`  | string |

#### 获取用于发布实时流的地址

```
GET /drones/{sn}/stream/publish
```

**Parameters**

| **类型** | **名称**  | **描述**  |**方案**  | 
| --- | --- | --- | --- | 
| **Path**  | **sn**（必选）  | 无人机序列号 | string | 

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 响应成功  | [Response 200](#dsspublishResp200)   |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="dsspublishResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#data_stream_publish) |

**<a name="data_stream_publish"></a>data**

| **名称** | **方案**  |
| --- | --- | --- |
| **temp**（必须） |[rtmp](#dssprtmp) |

**<a name="dssprtmp"></a>rtmp**

| **名称** | **描述**  | **方案**  |
| --- | --- | --- |
| **playUrl**（非必须） |播放流URL **如：**`"rtmp://10.61.91.204/fh/SIM-0005?type=publish&wowzatokenhash=2d5fc4145667401263afca90800b5428b986d626&wowzatokenendtime=1536071478" ` | string |

#### 获取所有带有直播流的无人机

```
GET /streams
```

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 响应成功  | [Response 200](#streamResp200)  |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="streamResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#data_streams) |

**<a name="data_streams"></a>data**

| **名称** | **方案**  |
| --- | --- | --- |
| **streams**（必须） |< [DroneGetModel](#DroneGetModel) > array |

### 实时报告
#### 获取在线无人机
```
GET /online/drones
```

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 响应成功  |  [Response 200](#odResp200)   |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="odResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
| **code**  |  | integer (int64) |
| **message**  | **如：**""  | string (string) |
| **data**  |  | [data](#data_online_drones) |

**<a name="data_online_drones"></a>data**

| **名称** | **方案**  |
| --- | --- |
| **drones**（必须） |< [DroneOnlineModel](#DroneOnlineModel) > array |

#### 飞行事件报告，飞行起飞后定期报告

```
POST /online/flightevents
```

**Parameters**

| **类型** | **名称** | **描述**  | **方案**  | 
| --- | --- | --- | --- | 
| **Body**  | **body**（必须）  | 在线状态 | [body](#ofbody) | 

**<a name="ofbody"></a>body**

| 类型 |名称  |
| --- | --- | 
| **recordList**  | < [FlyReport](#FlyReport) > array | 

**Responses**

| **HTTP编码** |  **方案**  |
| --- | --- |
| **200** |  无  |
| **400** |  无 |
| **500** |  无 |

**Produces**

* application/json

### 存储
对象存储
#### 上传对象
```
POST /objects
```

**Description**

请求的Content-type 取决于上传的内容,并且将保存成对象的元数据

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  | 
| --- | --- | --- | --- | --- |
| **Query**  | **category**（非必须）  | “支持的类别：`flightpic`、`flightvideo`、`missionpic`、`flightrecord`” | string | 
| **Query**  | **contentType**（非必须）  |MIME 内容类型的对象 | string | 
| **Query**  | **createTime**（非必须）  |创建时间，以Unix毫秒为单位 | integer | 
| **Query**  | **filename**（非必须）  | | string | 
| **Query**  | **recordId**（非必须）  |与对象关联的记录ID | string | 
| **Query**  | **teamId**（非必须）  | | integer | 
| **Query**  | **body**（非必须）  |二进制对象 | string | 

**Responses**

| **HTTP** **编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 响应成功  | [Response 200](#objectsResp200)   |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="objectsResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |  | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#data_online_drones) |

**<a name="data_online_drones"></a>data**

| **名称** | **描述**| **方案**  |
| --- | --- | --- |
| **path**（必须） |下载路径 如：`"/objects/missionPic/31243-2314-2341-4234"`|string|

**Consumes**

* application/binary

#### 获取录像,图片的文件列表
```
GET /objects
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  | **默认** |
| --- | --- | --- | --- | --- |
| **Query**  | **account**（非必须）  |  | string |  | 
| **Query**  | **category**（必须）  | 文件的类别：`flightpic`、`flightvideo`、`flightMedia` | **string** |  | 
| **Query**  | **endTs**（非必须）  |以Unix毫秒为单位的结束时间。默认为当前时间 | integer |  | 
| **Query**  | **limit**（非必须）  |返回条数 | integer |20  | 
| **Query**  | **offset**（非必须）  | 偏移量| string | 0  | 
| **Query**  | **sortDirection**（非必须）  | 排序方向、升序或降序 | string |  | 
| **Query**  | **sortField**（非必须）  |排序字段 | string |  | 
| **Query**  | **startTs**（非必须）  |以Unix毫秒为单位的开始时间。默认提前1天 | integer |  | 
| **Query**  | **teamId**（必须）  |二进制对象 | integer |  | 

**Responses**

| **HTTP编码** | **描述** | **方案**  |
| --- | --- | --- |
| **200** | 文件列表  | [Response 200](#postoResp200)  |
| **400** |  | 无 |
| **500** |  | 无 |

**<a name="postoResp200"></a>Response 200**

| **名称** |**描述**  | **方案** |
| --- | --- | --- |
|**code**  |     | integer (int64) |
|**message**  |**如：**""  | string (string) |
|**data**  |  | [data](#pdata_objects) |

**<a name="pdata_objects"></a>data**

| **名称** | **方案**| 
| --- | --- |
| **files**（非必须） | < [FileListModel](#FileListModel) > array|
| **pagination**（非必须） |[PaginationModel](#PaginationModel)|

#### 下载对象
```
GET /objects/{category}/{id}
```

**Description**

只在上传的时候指定 `Content-Type`

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  | 
| --- | --- | --- | --- | 
| **Path**  | **category**（必选）  | | string | 
| **Path**  | **id**（必选）  | 对象ID | string | 

#### 删除对象

```
DELETE /objects/{category}/{id}
```

**Description**

只在上传的时候指定 `Content-Type`

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  | 
| --- | --- | --- | --- | 
| **Path**  | **category**（必选）  | | string | 
| **Path**  | **id**（必选）  | 对象ID | string | 

**Responses**

| **HTTP编码** |  **方案**  |
| --- | --- |
| **200** |  无  |
| **400** |  无 |
| **500** |  无 |

#### 修改元数据对象
```
PUT /objects/{category}/{id}/metadata
```

**Parameters**

| **类型** |**名称**  |**描述**  |**方案**  | 
| --- | --- | --- | --- | 
| **Path**  | **category**（必选）  | | string | 
| **Path**  | **id**（必选）  | 对象ID | string | 
| **Body**  | **body**（非必选）  | 对象ID | [body](#ocimbody) | 

**<a name="ocimbody"></a>body**

| **名称** |  **描述**  | **方案**  |
| --- | --- |--- |
| **filename**（必须）| 对象的文件名，**如：**`shenzhen_001.jpg` | string |

**Responses**

| **HTTP编码** |  **方案**  |
| --- | --- |
| **200** |  无  |
| **400** |  无 |
| **500** |  无 |

### 远程控制
远程控制功能将命令从服务器发送到客户端，应仅与PILOT PE一起使用。

#### 开始直播流
```
PUT /remote/{sn}/livestreaming/start
```

**Parameters**

| **类型** | **名称** | **描述** | **方案** |
| --- | --- | --- |--- |
| **Path** |  **sn**（必须）  | 无人机序列号 | string  |

**Consumes**

* multipart/mixed

## Definitions（定义）

### <a name="DroneBindPara"></a>DroneBindPara

| **名称** | **描述** | **方案** |
| --- | --- |--- |
| **droneType**（必须） | 无人机类型，**如：**`M200`  | string |
| **sn**（必须） | 无人机序列号 , **如：** `DRONE_SN_1`  | string |


### <a name="DroneEditModel"></a>DroneEditModel

| **名称** | **描述** | **方案** |
| --- | --- |--- |
| **name**（必须） | 无人机名称 ，**如：**`Drone No.1`  | string |
| **teamId**（必须） | 无人机所属团队 **如：**1|  integer |


### <a name="DroneGetModel"></a>DroneGetModel

| **名称** | **描述** | **方案** |
| --- | --- |--- |
| **sn**（必须） | 无人机序列号 , **如：** `DRONE_SN_1`  | string |
| **name**（必须） | 无人机名称 ，**如：**`Drone No.1`  | string |
| **droneType**（必须） | 无人机类型，**如：**`M200`  | string |
| **teamId**（必须） | 无人机所属团队 **如：**1  |  integer |


### <a name="DroneOnlineModel"></a>DroneOnlineModel

多态性：组成

| **名称** | **描述** | **方案** |
| --- | --- |--- |
| **sn**（必须） | 无人机序列号 , **如：** `DRONE_SN_1`  | string |
| **name**（必须） | 无人机名称 ，**如：**`Drone No.1`  | string |
| **droneType**（必须） | 无人机类型，**如：**`M200`  | string |
| **teamId**（必须） | 无人机所属团队**如：**1  |  integer |
| **date**（必须） | 报告时间，以Unix毫秒为单位 |  integer |
| **lat**（必须） |   |  number (float) |
| **lng**（必须） |   |  number (float) |
| **altitude**（必须） |  |  number (float) |
| **speed**（必须） |   |  number (float) |
| **yaw**（必须） |   |  number (float) |
| **batteryLevel**（必须） |   |  integer |
| **homeLat**（非必须） |   |  number (float) |
| **homeLng**（非必须） |   |  number (float) |

### <a name="FightRecordPointModel"></a>FightRecordPointModel

| **名称** | **描述** | **方案** |
| --- | --- |--- |
| **date**（必须） | 报告时间，以Unix毫秒为单位 |  integer |
| **lat**（必须） |   |  number (float) |
| **lng**（必须） |   |  number (float) |
| **altitude**（必须） |  |  number (float) |
| **speed**（必须） |   |  number (float) |
| **yaw**（必须） |   |  number (float) |
| **batteryLevel**（必须） |   |  integer |

### <a name="FightStatisticsModel"></a>FightStatisticsModel

| **名称** | **方案** |
| --- | --- |
| **totalDuration**（必须） |  number (float) |
| **avgDuration**（必须） |  number (float) |
| **durationDistribution**（非必须） | [durationDistribution](#durationDistribution) |

**<a name="durationDistribution"></a>durationDistribution**

| **名称** | **方案** |
| --- | --- |
| **0-5**（非必须） | integer |
| **5-10**（非必须） |  integer |
| **10-15**（非必须） | integer |
| **20+** （非必须） |integer |

### <a name="FileListModel"></a>FileListModel

| **名称** | **描述** | **方案** |
| --- | --- |--- |
| **name**（非必须） | 文件名称 |  string |
| **path**（非必须） | 文件路径 **如：**`/objects/flightPic/31243-2314-2341-4234` |  string |
| **path**（非必须） | 文件路径 **如：**`/objects/flightPic/31243-2314-2341-4234/metadata` |  string |
| **metadataPath**（非必须） | 操作文件的路径，如更改文件名  |  string |
| **category**（非必须） | 对象类型**如：**`flightPic`|  string |
| **size**（非必须） | 文件大小|  string |
| **date**（非必须） | 文件修改时间，单位为Unix毫秒, **如：**1536064281000 |  integer |
| **thumbnail**（非必须） |base64编码的文件缩略图 |  integer |
| **contentType**（非必须） | 无人机所属团队 |  string |

### <a name="FlightGetModel"></a>FlightGetModel

多态性：组成

| **名称** | **描述** | **方案** |
| --- | --- |--- |
| **id**（必须） | 飞行记录ID |  string |
| **md5**（非必须） | 飞行结果的MD5签名 |  string |
| **fileName**（非必须） | 飞行文件名 |  string |
| **fileSize**（非必须） | 文件大小（字节）  | integer |
| **isFavourite**（非必须） | 最喜欢标识  |  integer |
| **path**（非必须） | 文件下载路径  |  string |
| **account**（非必须） | 用户账户  |  string |
| **teamId**（非必须） | 团队ID  |  integer |
| **sn**（非必须） | 无人机序列号 |  string |
| **droneType**（非必须） | 无人机型号 |  string |
| **duration**（非必须） | 单位：秒 |  integer |
| **distance**（非必须） | 飞行距离，单位： |  number (float) |
| **maxHeight**（非必须） | 最大飞行高度，单位： |  number (float) |
| **takeoffLongitude**（非必须） | 起飞经度，**如：**`102.20215` |  number (float) |
| **takeoffLatitude**（非必须） | 起飞纬度，**如：**`29.980776` |  number (float) |

### <a name="FlightSummaryModel"></a>FlightSummaryModel

| **名称** | **描述** | **方案** |
| --- | --- |--- |
| **account**（非必须） | 用户账户  |  string |
| **teamId**（非必须） | 团队ID  |  integer |
| **sn**（非必须） | 无人机序列号 |  string |
| **droneType**（非必须） | 无人机型号 |  string |
| **duration**（非必须） | 单位：秒 |  integer |
| **distance**（非必须） | 飞行距离，单位： |  number (float) |
| **maxHeight**（非必须） | 最大飞行高度|  number (float) |
| **takeoffLongitude**（非必须） | 起飞经度，**如：**`102.20215` |  number (float) |
| **takeoffLatitude**（非必须） | 起飞纬度，**如：**`29.980776` |  number (float) |


### <a name="FlyReport"></a>FlyReport

| **名称** | **描述** | **方案** |
| --- | --- |--- |
| **date**（必须） | 报告时间，以Unix毫秒为单位  | integer |
| **droneId**（非必须） | 无人机序列号  |  string |
| **latitude**（必须） |  |  number (float) |
| **longitude**（必须） |  |  number (float) |
| **altitude**（非必须） | 实际上，这是和`HOME`相比的飞行高度 |  number (float) |
| **yaw**（必须） |  |  number (float) |
| **speed**（必须） |  |  number (float) |
| **speedH**（非必须） | 水平速度 |  number (float) |
| **speedV**（非必须） | 垂直速度|  number (float) |
| **flightMode**（非必须） | 飞行模式，显示在Pilot APP的topbar|  integer |
| **waypointMode**（非必须） | 航点模式，0-暂停 1-执行（默认）|  integer |
| **isAllowLive**（非必须） |  | boolean |
| **batteryLevel**（非必须） |  | integer |
| **rcSignal**（非必须） |  | integer |
| **homeLatitude**（非必须） |  | number (float) |
| **homeLongitude**（非必须） |  | number (float) |
| **gimbal**（非必须） |  | < [FlyReportGimbalModel](#FlyReportGimbalModel)> array |
| **camera**（非必须） |  | < [FlyReportCameraModel](#FlyReportCameraModel)> array |
| **accessoryModel**（非必须） |  | string |
| **recordId**（必须） |  | string |
| **taskId**（非必须） |  | string |
| **missionId**（非必须） |  | string |

### <a name="FlyReportAccessoryModel"></a>FlyReportAccessoryModel

| **名称**  | **方案** |
| ---  |--- |
| **model**（非必须） | string |

### <a name="FlyReportCameraModel"></a>FlyReportCameraModel

| **名称**  | **方案** |
| ---  |--- |
| **id**（非必须） | integer |
| **status**（非必须） | integer |
| **model**（非必须） | string |

### <a name="FlyReportGimbalModel"></a>FlyReportGimbalModel

| **名称**  | **方案** |
| ---  |--- |
| **id**（非必须） | integer |
| **yaw**（非必须） | number (float) |
| **pitch**（非必须） | number (float) |
| **roll**（非必须） | number (float) |

### <a name="InputErrorResponse"></a>InputErrorResponse

客户端的错误信息
多态性：组成

| **名称**  | **描述** |**方案** |
| ---  |--- |--- |
| **code**（必须） |  |integer (int64) |
| **message**（必须） | **如：**"" | string (string)|
| **data**（非必须） |  | string |

### <a name="MissionBriefModel"></a>MissionBriefModel

| **名称**  | **描述** |**方案** |
| ---  |--- |--- |
| **name**（必须） |  任务名称，**如：**`mission1` | string |
| **teamId**（非必须） | 任务所属团队，**如：**1 | integer |
| **type**（非必须） | 任务类型，0: waypoints（航点） 1: survey（测量），**如：**1 | integer |
| **version**（非必须） | version，**如：**1  | integer |
| **coverPic**（非必须） | 绘图路径  | string |
| **latitude**（必须） | 纬度，**如: **`29.980776` | number (float) |
| **longitude**（必须） | 纬度，**如:** `102.20215` | number (float) |
| **locationDesc**（非必须） | 任务地点，**如: **`Nanshan, Shenzhen, China` | string |
| **distance**（必须） |任务距离（单位：m），**如: **`328.1` |  number (float)  |
| **duration**（必须） |预期任务持续时间（单位：秒），**如:** `512` | integer |


### <a name="MissionCreateModel"></a>MissionCreateModel

多态性：组成

| **名称**  | **描述** |**方案** |
| ---  |--- |--- |
| **name**（必须） |  任务名称，**如：**`mission1` | string |
| **teamId**（非必须） | 任务所属团队，**如：**1 | integer |
| **type**（非必须） | 任务类型，0: waypoints（航点） 1: survey（测量），**如：**1 | integer |
| **version**（非必须） | version，**如：**1  | integer |
| **coverPic**（非必须） | 绘图路径  | string |
| **latitude**（必须） | 纬度，**如: **`29.980776` | number (float) |
| **longitude**（必须） | 纬度，**如:** `102.20215` | number (float) |
| **locationDesc**（非必须） | 任务地点，**如: **`Nanshan, Shenzhen, China` | string |
| **distance**（必须） |任务距离（单位：m），**如: **`328.1` |  number (float)  |
| **duration**（必须） |预期任务持续时间（单位：秒），**如:** `512` | integer |
| **flyParam**（必须） |`flyparam`数据结构取决于任务`类型`，具体在link上指定：[飞行任务数据结构]**如：**`{"gotoFirstPointMode" : 0,"actionOnFinish" : 1,"actionOnRcLost" : 0,"gimbalPatchMode" : 0,"flightPathMode" : 0,"autoFlightSpeed" : 12.1,"maxFlightSpeed" : 10,"headingMode" : 0,"interestPointLat" : 29.980911,"interestPointLng" : 102.202581,"repeatTimes" : 1,"waypoints" : [ {"latitude" : 29.98091092019941,"longitude" : 102.2025812475324,"altitude" : 345.69214,"heading" : 20,"cornerRadius" : 1.2,"actionTimeLimit" : 20,"cameraAction" : 1,"cameraActionParam" : 1000,"hasAction" : 1,"hasSpeed" : 1,"speed" : 13.4,"gimbalPitch" : -10,"turnMode" : 1,"actions" : [ {"type" : 1,"param" : 20} ]} ]}`|object |





### <a name="MissionGetBriefModel"></a>MissionGetBriefModel

多态性：组成

| **名称**  | **描述** | **方案** |
| ---  |--- | --- |
| **id**（必须） | **如：**`"mission-id-1"`  | string |
| **createTime**（必须） | **如：**`"1.531813501928E12" ` | number |
| **createBy**（非必须） | **如：**`"zach"`  | string |
| **updateTime**（必须） | **如：**`1.531813501928E12`  | number |
| **updateBy**（非必须） | **如：**`"dylan"` | string |
| **name**（必须） | 任务名称，**如：**`"mission1"` | string |
| **teamId**（非必须） | 任务所属团队，**如：**1 | integer |
| **type**（非必须） | 任务类型，0: waypoints（航点） 1: survey（测量）  | integer |
| **version**（非必须） | version，**如：**1  | integer |
| **coverPic**（非必须） | 绘图路径  | string |
| **latitude**（必须） | 纬度，**如: **`29.980776` | number (float) |
| **longitude**（必须） | 纬度，**如:** `102.20215` | number (float) |
| **locationDesc**（非必须） | 任务地点，**如:** `Nanshan, Shenzhen, China` | string |
| **distance**（必须） |任务距离（单位：m），**如:** `328.1` |  number (float)  |
| **duration**（必须） |预期任务持续时间（单位：秒），**如:** `512` | integer |

### <a name="MissionGetDetailModel"></a>MissionGetDetailModel

多态性：组成

| **名称**  | **描述** |**方案** |
| ---  |--- | --- |
| **id**（必须） | **如：**`"mission-id-1"`  | string |
| **createTime**（必须） | **如：**`"1.531813501928E12" ` | number |
| **createBy**（非必须） | **如：**`"zach"`  | string |
| **updateTime**（必须） | **如：**`1.531813501928E12`  | number |
| **updateBy**（非必须） | **如：**`"dylan"` | string |
| **name**（必须） | 任务名称，**如：**`"mission1"` | string |
| **teamId**（非必须） | 任务所属团队，**如：**1 | integer |
| **type**（非必须） | 任务类型，0: waypoints（航点） 1: survey（测量）  | integer |
| **version**（非必须） | version，**如：**1  | integer |
| **coverPic**（非必须） | 绘图路径  | string |
| **latitude**（必须） | 纬度，**如:** `29.980776` | number (float) |
| **longitude**（必须） | 纬度，**如:** `102.20215` | number (float) |
| **locationDesc**（非必须） | 任务地点，**如:** `Nanshan, Shenzhen, China` | string |
| **distance**（必须） |任务距离（单位：m），**如:** `328.1` |  number (float)  |
| **duration**（必须） |预期任务持续时间（单位：秒），**如:** `512` | integer |
| **flyParam**（必须） |`flyparam`数据结构取决于任务`类型`，具体在link上指定：[飞行任务数据结构]**如：**`{"gotoFirstPointMode" : 0,"actionOnFinish" : 1,"actionOnRcLost" : 0,"gimbalPatchMode" : 0,"flightPathMode" : 0,"autoFlightSpeed" : 12.1,"maxFlightSpeed" : 10,"headingMode" : 0,"interestPointLat" : 29.980911,"interestPointLng" : 102.202581,"repeatTimes" : 1,"waypoints" : [ {"latitude" : 29.98091092019941,"longitude" : 102.2025812475324,"altitude" : 345.69214,"heading" : 20,"cornerRadius" : 1.2,"actionTimeLimit" : 20,"cameraAction" : 1,"cameraActionParam" : 1000,"hasAction" : 1,"hasSpeed" : 1,"speed" : 13.4,"gimbalPitch" : -10,"turnMode" : 1,"actions" : [ {"type" : 1,"param" : 20} ]} ]}`| object |


### <a name="MissionServerGenModel"></a>MissionServerGenModel

| **名称**  | **描述** |**方案** |
| ---  |--- | --- |
| **id**（必须） |**如：**`"mission-id-1"`  | string |
| **createTime**（必须） | **如：**`"1.531813501928E12" ` | number |
| **createBy**（非必须） | **如：**`"zach"`  | string |
| **updateTime**（必须） | **如：**`1.531813501928E12`  | number |
| **updateBy**（非必须） | **如：**`"dylan"` | string |


### <a name="OnlineReport"></a>OnlineReport

| **名称**  | **描述** |**方案** |
| ---  |--- | --- |
| **date**（非必须） | 报告时间，以Unix毫秒为单位 | integer |
| **sn**（必须） | 无人机序列号 | string |
| **latitude**（必须） | | number (float) |
| **longitude**（必须） | | number (float) |
| **height**（必须） |  | number (float) |


### <a name="PaginationModel"></a>PaginationModel

| **名称**  | **描述** |**方案** |
| ---  |--- | --- |
| **pages**（必须） | **如：**`10` | integer |
| **total**（必须） | **如：**`191` | integer |

### <a name="ServerInternalErrorResponse"></a>ServerInternalErrorResponse

服务端的错误信息
多态性：组成

| **名称**  | **描述** |**方案** |
| ---  |--- |--- |
| **code**（必须） |  |integer (int64) |
| **message**（必须） | **如：**"" | string (string)|
| **data**（非必须） |  | string |


### <a name="StandardResponse"></a>StandardResponse

| **名称**  | **描述** |**方案** |
| ---  |--- |--- |
| **code**（必须） |  |integer (int64) |
| **message**（必须） | **如：**"" | string (string)|

### <a name="SuccessResponse"></a>SuccessResponse

成功响应
多态性：组成

| **名称**  | **描述** |**方案** |
| ---  |--- |--- |
| **code**（必须） |  |integer (int64) |
| **message**（必须） | **如：**"" | string (string)|

### <a name="TeamCreateModel"></a>TeamCreateModel

| **名称**  | **描述** |**方案** |
| ---  |--- |--- |
| **name**（必须） | **如: **`"Beijing Team"` | string |
| **parentId**（必须） | 父团队ID。创建顶级团队时设置为0。**如：**0 | integer |

### <a name="TeamGetModel"></a>TeamGetModel

| 名称  | 描述 |方案 |
| ---  |--- |--- |
| **id**（必须） | **如：**2  |integer  |
| **name**（必须） | **如：**`"Beijing Team"` | string |
| **parentId**（非必须） | 父团队ID，**如：**1 | integer |
| **role**（必须） | 用户在团队中的角色，**如：**1 | integer |
| **subTeams**（非必须） | 子团队，使用与TeamGetModel相同的结构，**如：**`[ {"id" : 3,"name" : "ShenZhen","role" : 2,"parentId" : 2,"subTeams" : [ {"id" : 4,"name" : "NanShan","role" : 2,"parentId" : 3} ]} ]`| < object > array |


### <a name="TeamMate"></a>TeamMate

| **名称**  | **描述** | **方案** |
| ---  |--- |--- |
| **name**（非必须） | 用户账户  |string  |
| **level**（非必须） | 团队等级，**如：**2 | integer |
| **realName**（非必须） | 用户在团队中的名字 | string |


### <a name="TeamMemberCreateModel"></a>TeamMemberCreateModel

| **名称**  | **描述** | **方案** |
| ---  |--- |--- |
| **userId**（必须） | **如：**`1`  | integer  |
| **role**（必须） | 用户在团队中的角色，**如：**2 | integer |

### <a name="TeamMemberGetModel"></a>TeamMemberGetModel

| **名称**  | **描述** | **方案** |
| ---  |--- |--- |
| **userId**（必须） | **如：**`1`  | integer  |
| **name**（必须） | 用户名称 | string  |
| **account**（必须） | 用户账户，**如：**`"USER_1"`  | string  |
| **role**（必须） | 用户在团队中的角色，**如：**`2` | integer |


### <a name="TeamMemberUpdateModel"></a>TeamMemberUpdateModel

| **名称**  | **描述** | **方案** |
| ---  |--- |--- |
| **role**（必须） | 用户在团队中的角色，**如：**`2` | integer |

### <a name="TeamUpdateModel"></a>TeamUpdateModel

| **名称**  | **描述** | **方案** |
| ---  |--- |--- |
| **name**（必须） | **如：**`"Beijing Team"` | string |


### <a name="UserCreateModel"></a>UserCreateModel

| **名称**  | **描述** | **方案** |
| ---  |--- |--- |
| **account**（必须） | **如：**`"p1@dji.com"` | string |
| **password**（必须） | **如：**`"XXXXXXXXX"` | string |
| **name**（必须） | **如：**`"Pilot Bob"` | string |

### <a name="UserGetModel"></a>UserGetModel

| **名称**  | **描述** | **方案** |
| ---  |--- |--- |
| **userId**（必须） | **如：**`1` | integer |
| **account**（必须） | **如：**`"p1@dji.com"` | string |
| **name**（必须） | **如：**`"Pilot Bob"` | string |

### <a name="UserUpdateModel"></a>UserUpdateModel

| **名称**  | **描述** | **方案** |
| ---  |--- |--- |
| **password**（必须） | **如：**`"XXXXXXXXX"` | string |
| **name**（必须） | **如：**`"Pilot Steve"` | string |

