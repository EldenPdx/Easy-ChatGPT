### RustChatGPT项目接口设计说明书

编制单位：东莞理工学院RustChatGPT小组
日期：2024年06月

------

## 修订历史记录

| 版本 | 日期     | AMD  | 修订者 | 说明               |
| ---- | -------- | ---- | ------ | ------------------ |
| V1.0 | 20240625 | A    | 彭铭琨 | 新增接口设计说明书 |

------

## 1 引言

### 1.1 背景

接口设计是指系统内部，系统和操作系统间、多个系统间以及系统和人之间如何通信。本设计文档用于描述用户登录、注册、提示词交互、查询用户历史对话记录以及删除指定对话记录的接口。

### 1.2 概述

本设计说明书定义了以下接口：

- 用户登录接口
- 用户注册接口
- 提示词接口
- 查询用户历史对话记录接口
- 查询用户对应对话的历史记录接口
- 删除指定id的历史对话记录接口

## 2 适用范围

本设计说明书适用于与用户认证、与ChatGPT交互、以及管理用户与ChatGPT交互历史记录的接口。

## 3 接口列表

### 3.1 用户登录接口

**接口信息**

| 接口连接 | 域名/api/user/login    |
| -------- | ---------------------- |
| 接口说明 | 用于用户登录系统的接口 |
| 请求方式 | POST                   |

**接口参数**

| 参数     | 定义                 | 是否必须 | 类型   | 备注 |
| -------- | -------------------- | -------- | ------ | ---- |
| username | 用户用于登录的用户名 | 是       | string | /    |
| password | 用户用于登录的密码   | 是       | string | /    |

**示例**

```
{
  "username": "test",
  "password": "test"
}
```

**接口返回结果**

| 参数  | 定义     | 类型   | 备注 |
| ----- | -------- | ------ | ---- |
| msg   | 结果标识 | string | /    |
| code  | 状态码   | int    | /    |
| token | 登录标识 | string | /    |

**示例**

```
{
  "code": 0,
  "msg": "login success",
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIxNTU0NjU1OTEwNDYxMTgxOTUzIiwiaWF0IjoxNjYxOTM0MTE2LCJleHAiOjUzOTE2NjU0MTE2fQ.M-ydCLLWZM9nNZGSDEss34vHftWB8PamC1kcJ0BBWjeDDuMuMTDGoYCHqjdNOcRJklup70AatrKSylWlCUv1Q"
}
```

### 3.2 用户注册接口

**接口信息**

| 接口连接 | 域名/api/user/register |
| -------- | ---------------------- |
| 接口说明 | 用于用户注册的接口     |
| 请求方式 | POST                   |

**接口参数**

| 参数     | 定义                 | 是否必须 | 类型   | 备注 |
| -------- | -------------------- | -------- | ------ | ---- |
| username | 用户用于登录的用户名 | 是       | string | /    |
| password | 用户用于登录的密码   | 是       | string | /    |

**示例**

```
{
  "username": "test",
  "password": "test"
}
```

**接口返回结果**

| 参数 | 定义     | 类型   | 备注 |
| ---- | -------- | ------ | ---- |
| msg  | 结果标识 | string | /    |
| code | 状态码   | int    | /    |

**示例**

```
{
  "code": 0,
  "msg": "register success"
}
```

### 3.3 提示词接口

**接口信息**

| 接口连接 | 域名/api/user/prompt                        |
| -------- | ------------------------------------------- |
| 接口说明 | 用于接收提示词参数并与ChatGPT进行交互的接口 |
| 请求方式 | POST                                        |

**接口参数**

| 参数   | 定义                          | 是否必须 | 类型   | 备注 |
| ------ | ----------------------------- | -------- | ------ | ---- |
| prompt | 用户用于与ChatGPT交互的提示词 | 是       | string | /    |
| token  | 用户登录标识                  | 是       | string | /    |

**示例**

```
{
  "prompt": "今天天气怎么样？",
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIxNTU0NjU1OTEwNDYxMTgxOTUzIiwiaWF0IjoxNjYxOTM0MTE2LCJleHAiOjUzOTE2NjU0MTE2fQ.M-ydCLLWZM9nNZGSDEss34vHftWB8PamC1kcJ0BBWjeDDuMuMTDGoYCHqjdNOcRJklup70AatrKSylWlCUv1Q"
}
```

**接口返回结果**

| 参数 | 定义          | 类型   | 备注 |
| ---- | ------------- | ------ | ---- |
| msg  | ChatGPT的回复 | string | /    |
| code | 状态码        | int    | /    |

**示例**

```
{
  "code": 0,
  "msg": "今天天气很好，是晴天。"
}
```

### 3.4 查询用户所有的历史对话记录接口

**接口信息**

| 接口连接 | 域名/api/user/all/histories    |
| -------- | ------------------------------ |
| 接口说明 | 用于查询用户所有的历史对话记录 |
| 请求方式 | POST                           |

**接口参数**

| 参数  | 定义         | 是否必须 | 类型   | 备注 |
| ----- | ------------ | -------- | ------ | ---- |
| token | 用户登录标识 | 是       | string | /    |

**示例**

```
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIxNTU0NjU1OTEwNDYxMTgxOTUzIiwiaWF0IjoxNjYxOTM0MTE2LCJleHAiOjUzOTE2NjU0MTE2fQ.M-ydCLLWZM9nNZGSDEss34vHftWB8PamC1kcJ0BBWjeDDuMuMTDGoYCHqjdNOcRJklup70AatrKSylWlCUv1Q"
}
```

**接口返回结果**

| 参数      | 定义                       | 类型  | 备注 |
| --------- | -------------------------- | ----- | ---- |
| code      | 状态码                     | int   | /    |
| histories | 查询得到的所有历史对话记录 | array | /    |

**示例**

```
{
  "code": 0,
  "histories": [
    {
      "id": 1,
      "title": "今天天气"
    },
    {
      "id": 2,
      "title": "美食菜谱"
    }
  ]
}
```

### 3.5 查询用户对应对话的历史记录接口

**接口信息**

| 接口连接 | 域名/api/user/history          |
| -------- | ------------------------------ |
| 接口说明 | 用于查询用户对应对话的历史记录 |
| 请求方式 | POST                           |

**接口参数**

| 参数  | 定义               | 是否必须 | 类型   | 备注 |
| ----- | ------------------ | -------- | ------ | ---- |
| id    | 对话记录的唯一标识 | 是       | long   | /    |
| token | 用户登录标识       | 是       | string | /    |

**示例**

```
{
  "id": 1,
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIxNTU0NjU1OTEwNDYxMTgxOTUzIiwiaWF0IjoxNjYxOTM0MTE2LCJleHAiOjUzOTE2NjU0MTE2fQ.M-ydCLLWZM9nNZGSDEss34vHftWB8PamC1kcJ0BBWjeDDuMuMTDGoYCHqjdNOcRJklup70AatrKSylWlCUv1Q"
}
```

**接口返回结果**

| 参数    | 定义          | 类型  | 备注 |
| ------- | ------------- | ----- | ---- |
| history | ChatGPT的回复 | array | /    |
| code    | 状态码        | int   | /    |

**示例**

```
{
  "code": 0,
  "history": [
    {
      "role": "user",
      "content": "今天天气怎么样？"
    },
    {
      "role": "assistant",
      "content": "今天天气很好，是晴天。"
    }
  ]
}
```

### 3.6 删除指定id的历史对话记录接口

**接口信息**

| 接口连接 | 域名/api/user/delete/history     |
| -------- | -------------------------------- |
| 接口说明 | 用于删除用户指定id的历史对话记录 |
| 请求方式 | POST                             |

**接口参数**

| 参数  | 定义               | 是否必须 | 类型   | 备注 |
| ----- | ------------------ | -------- | ------ | ---- |
| id    | 对话记录的唯一标识 | 是       | long   | /    |
| token | 用户登录标识       | 是       | string | /    |

**示例**

```
{
  "id": 1,
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIxNTU0NjU1OTEwNDYxMTgxOTUzIiwiaWF0IjoxNjYxOTM0MTE2LCJleHAiOjUzOTE2NjU0MTE2fQ.M-ydCLLWZM9nNZGSDEss34vHftWB8PamC1kcJ0BBWjeDDuMuMTDGoYCHqjdNOcRJklup70AatrKSylWlCUv1Q"
}
```

**接口返回结果**

| 参数 | 定义     | 类型   | 备注 |
| ---- | -------- | ------ | ---- |
| msg  | 结果标识 | string | /    |
| code | 状态码   | int    | /    |

**示例**

```
{
  "code": 0,
  "msg": "delete success"
}
```

------

## 4 安全性设计

接口采用Token认证方式，每个用户在登录成功后会得到一个Token。该Token需要在每次请求时通过请求头部传递，服务器会对Token进行校验，确保用户身份的合法性。

## 5 接口调用顺序

用户在调用接口时应按以下顺序进行：

1. 用户注册接口
2. 用户登录接口
3. 提示词接口
4. 查询用户所有的历史对话记录接口
5. 查询用户对应对话的历史记录接口
6. 删除指定id的历史对话记录接口

------

## 6 异常处理

对于接口请求失败的情况，系统会返回相应的错误码和错误信息，开发者需要根据错误信息进行相应处理。

------

## 7 术语表

| 术语  | 定义                           |
| ----- | ------------------------------ |
| Token | 用户登录标识，用于验证用户身份 |
| API   | 应用程序编程接口               |

------

### 附录

**接口返回状态码说明**

| 状态码 | 说明           |
| ------ | -------------- |
| 0      | 请求成功       |
| 1      | 请求失败       |
| 401    | 用户未认证     |
| 403    | 用户无权限     |
| 404    | 请求资源未找到 |