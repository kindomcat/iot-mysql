# 1 业务流程

## 1.1 名词定义

+ 类编码: IoT平台为电气设备和传感器提供唯一的标识符
+ sn: 物联⽹关序列号
+ productKey: IoT平台分配的企业授权码
+ gatewayId: 物联网关的IoT平台编码
+ gatewaySecret: 物联网关的IoT平台密码
+ 管理员: 特指IoT平台运营方的超级管理员
+ 用户: 指的企业用户，包含企业管理员等
+ deviceCatId: 电气设备类编码
+ sensorCatId: 传感器类编码
+ 母板：为同一使用场景的物联网关设定统一的配置模板，资产项的电气设备和传感器不匹配物联网关，使用类编码 S_XXX_XXX、D_XXX_XXX_XXX
+ 生产模板：物联网关实际生产环境使用的配置模板，资产项的电气设备和传感器匹配物联网关，使用类编码 S_XXX_XXX_id、D_XXX_XXX_XXX_id
+ 子项：同一物联网关的配置模板使用多个相同的电气设备，基础属性编辑一次或多次。

## 1.3 类编码设定

IoT平台为电气设备和传感器提供唯一的标识符，用下划线组合在一起。

分类 | 定义
:- | :-
传感器 | S_XXX_XXX
电气设备 | D_XXX_XXX_XXX

+ 第二项 cid: 在配置文件中配置的类型编码，参考下面列表格式。
+ 第三项 sid: 根据厂商和型号自动分配唯一标识，从1开始累积。
+ 第四项 version: 适用于同种电气设备中不同业务场景，由管理员输入，IoT保证唯一，从1开始累积。

**传感器类型**

编码 | 名称 | 类型
:--|:--|:--
001 | 电能表 | ammeter
002 | 温度传感器 | temperature
003 | 湿度传感器 | humidity
004 | 北斗GPS | gps
005 | 综保 | insurance
006 | 变压器温显 | transformer
007 | 有载调压控制器 | loaded
008 | 数显表 | amc
009 | 无功补偿控制器 | recoup

**电气设备类型**

编码 | 名称 | 类型
:--|:--|:--
001 | 电表 | ammeter
002 | 高压柜 | highVoltage
003 | 低压柜 | lowVoltage
004 | 干式变压器 | transformer
005 | 无功补偿装置 | svg

## 1.3 设备库

### 1.3.1 传感器物模型

```sequence
title: 传感器物模型流程
participant 管理员
participant IoT平台

管理员->IoT平台: 创建传感器: 录入基础属性（类型、名称、型号、厂商、描述）
IoT平台->IoT平台: 生成设备项: 类编码sensorCatId、基础属性
IoT平台-->>管理员: 成功
管理员->IoT平台: 配置属性（协议、停止位、校验位等）
IoT平台->IoT平台: 更新设备项: 配置属性
IoT平台-->>管理员: 成功
管理员->IoT平台: 配置命令
IoT平台->IoT平台: 更新设备项: 命令
IoT平台-->>管理员: 成功
管理员->IoT平台: 配置点位，关联命令
IoT平台->IoT平台: 更新设备项: 点位及关联命令
IoT平台-->>管理员: 成功
管理员-->>管理员: 单个传感器物模型配置完成
管理员->管理员: 使用上述流程创建多个传感器物模型
```

### 1.3.1 电气设备物模型

```sequence
title: 电气设备物模型流程
participant 管理员
participant IoT平台

管理员->IoT平台: 创建电气设备: 录入基础属性（类型、名称、型号、厂商、描述、版本号）
IoT平台->IoT平台: 生成设备项: 类编码deviceCatId、基础属性
IoT平台-->>管理员: 成功，返回所有编辑好的传感器列表
管理员->IoT平台: 选择需要的传感器
IoT平台->IoT平台: 更新设备项: 关联传感器类编码绑定
IoT平台-->>管理员: 成功
管理员->IoT平台: 配置点位（筛选传感器点位）
IoT平台->IoT平台: 更新设备项: 关联点位
IoT平台-->>管理员: 成功
管理员->IoT平台: 配置事件/规则
IoT平台->IoT平台: 更新设备项: 事件/规则
IoT平台-->>管理员: 成功
管理员->IoT平台: 配置服务
IoT平台->IoT平台: 更新设备项: 服务
IoT平台-->>管理员: 成功
管理员-->>管理员: 单个电气设备物模型配置完成
管理员->管理员: 使用上述流程创建多个电气设备物模型
```

## 1.4 配置母板

```sequence
title: 配置母板流程
participant 用户
participant IoT平台

用户->IoT平台: 创建母板: 名称、描述
IoT平台->IoT平台: 生成母板
IoT平台-->>用户: 成功，返回所有编辑好的电气设备列表
用户->IoT平台: 选择设备库中电气设备（相同配置 子项 x N）
IoT平台->IoT平台: 临时资产库自动添加[电气设备和传感器]，不匹配物联网关
IoT平台-->>用户: 返回电气设备和传感器
用户->IoT平台: 编辑基础属性（相同配置一次，不同则配置多次）
IoT平台->IoT平台: 更新基础属性
IoT平台-->>用户: 成功
用户-->>用户: 单个母板配置完成
用户->用户: 使用上述流程创建多个母板
```

## 1.5 物联网关接入

基于物联网关录入用户名和密码，IoT平台对接入物联网关进行身份认证，通过后，接入MQTT broker进行业务通信。

1. 首次登录
    + 首次登录或者登录失败后，物联网关重新使用 `⽤户名: test.sn.timer 密码: md5(mxny.timer)` 进行重新授权操作。
    + 通过 `Topic: /test/sn/# ` 请求 gatewayId 和 gatewaySecret
2. 身份认证
    + 初始化认证，物联网关使用 `⽤户名: test.sn.timer 密码: md5(mxny.timer)` 进行认证。
    + 分配gatewayId后，物联网关使用 `⽤户名: test.gatewayId.timer 密码: md5(gatewaySecret.timer)` 进行认证。

```sequence
title: 物联网关接入流程
participant 物联网关
participant IoT平台

物联网关->IoT平台: 首次登录
Note over 物联网关: ⽤户名: test.sn.timer\n密码: md5(mxny.timer)
IoT平台->IoT平台: 生成临时资产项: sn、gatewayId、gatewaySecret
物联网关->IoT平台: Topic: /test/sn/# 请求gatewayId
IoT平台-->>物联网关: 返回gatewayId、gatewaySecret

物联网关->物联网关: 保存gatewayId、gatewaySecret
Note over 物联网关: ⽤户名: test.gatewayId.timer\n密码: md5(gatewaySecret.timer)
物联网关->IoT平台: 身份认证
IoT平台->IoT平台: 认证通过
IoT平台-->>物联网关: 成功，准许绑定企业授权码/配置模板
物联网关->IoT平台: 上报数据
IoT平台-->>IoT平台: 不存储

物联网关->物联网关: 输入企业授权码productKey
Note over 物联网关: ⽤户名: productKey.gatewayId.timer\n密码: md5(gatewaySecret.timer)
物联网关->IoT平台: 激活认证
IoT平台->IoT平台: 认证通过，移到资产库
IoT平台-->>物联网关: 成功，准许正式上报数据
物联网关->IoT平台: 上报数据
IoT平台->IoT平台: 存储
```

## 1.6 物联网关配置模板

### 1.6.1 使用母板

```sequence
title: 使用母板流程
participant 用户
participant IoT平台
participant 物联网关

物联网关->IoT平台: 接入
用户->IoT平台: 点击使用母板
IoT平台-->>用户: 返回所有母板
用户->IoT平台: 选定母板
IoT平台->IoT平台: 资产库自动添加，匹配物联网关，\n生成唯一标识: 类编码_id
IoT平台-->>用户: 返回电气设备和传感器
用户->IoT平台: 提交编辑基础属性（如有需要）
IoT平台->IoT平台: 更新基础属性，生成配置模板
IoT平台-->>用户: 完成
用户-->>用户: 模板配置完成
物联网关->IoT平台: 请求下载模板
IoT平台->物联网关: 自动计算rate，组装模板（物模型、组织结构、基础属性）
```

### 1.6.2 配置新模板

```sequence
title: 配置新模板流程
participant 用户
participant IoT平台
participant 物联网关

物联网关->IoT平台: 接入
用户->IoT平台: 点击使用母板
IoT平台-->>用户: 返回所有设备库中电气设备
用户->IoT平台: 选定电气设备（相同配置 子项 x N）
IoT平台->IoT平台: 资产库自动添加，匹配物联网关，\n生成唯一标识: 类编码_id\n生成配置模板
IoT平台-->>用户: 返回电气设备和传感器
用户->IoT平台: 提交编辑基础属性（相同配置一次，不同则配置多次）
IoT平台->IoT平台: 更新基础属性
IoT平台-->>用户: 完成
用户-->>用户: 模板配置完成
物联网关->IoT平台: 请求下载模板
IoT平台->物联网关: 自动计算rate，组装 生产模板（物模型、组织结构、基础属性）
```

## 1.6 资产库
