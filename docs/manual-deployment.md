# 手动创建资源详细指南

本文档提供在华为云控制台手动创建电商平台所需资源的详细步骤。

---

## 当前已通过 Terraform 部署的资源

| 资源 | 名称 | ID |
|------|------|-----|
| VPC | ecommerce-platform-vpc | 27e51a2a-c9bb-4d95-aef8-170846445198 |
| 子网 | ecommerce-platform-subnet | 5f6569a4-d6a8-44c4-8081-6c82d2ae018d |
| 安全组 | ecommerce-platform-sg | b6dda708-13ef-4598-949d-bdb4b056dcbd |
| OBS Bucket | ecommerce-platform-assets | - |
| OBS Bucket | ecommerce-platform-function-code | - |

---

## 需要手动创建的资源

以下资源需要登录华为云控制台手动创建。

---

## 步骤 1：创建 RDS MySQL

### 1.1 登录控制台

1. 打开浏览器访问：https://console.huaweicloud.com
2. 使用账号登录

### 1.2 进入 RDS 创建页面

1. 在控制台顶部搜索框输入 **"云数据库 RDS"**，点击搜索结果
2. 进入 **云数据库 RDS** 页面
3. 点击 **实例管理**
4. 点击 **创建实例** 按钮

### 1.3 配置实例参数

按照以下表格配置：

| 配置项 | 推荐值 | 说明 |
|--------|--------|------|
| 结算模式 | 按量付费 | 按小时结算，随时可释放 |
| 实例类型 | 单机 | 降低规格，减少资源消耗 |
| 地域 | cn-north-4 | 必须与已创建的 VPC 同一区域 |
| 数据库版本 | MySQL 8.0 | 兼容主流应用 |
| 规格 | 选择可用的最低规格 | 如 rds.mysql.s2.large 或 rds.mysql.s1.large |
| 存储类型 | LOCALSSD 或 CLOUDSSD | 根据可用性选择 |
| 存储空间 | 20GB | 起步容量，可后续扩容 |
| VPC | ecommerce-platform-vpc | 选择已创建的 VPC |
| 子网 | ecommerce-platform-subnet | 选择已创建的子网 |
| 安全组 | ecommerce-platform-sg | 选择已创建的安全组 |
| 端口 | 3306 | MySQL 默认端口 |

### 1.4 设置密码

1. 设置数据库 root 用户密码
2. 密码需包含：大小写字母 + 数字，长度 8-32 位
3. 点击 **立即购买**

### 1.5 等待创建完成

1. 实例创建需要 5-10 分钟
2. 状态变为 **运行中** 后，记下以下信息：
   - 内网地址（用于应用连接）

---

## 步骤 2：创建 RDS MySQL MongoDB

### 2.1 进入 GaussDB 创建页面

1. 在控制台顶部搜索框输入 **"GaussDB"**，点击搜索结果
2. 进入 **GaussDB(for Mongo)** 页面
3. 点击 **实例管理**
4. 点击 **创建实例** 按钮

### 2.2 配置实例参数

| 配置项 | 推荐值 | 说明 |
|--------|--------|------|
| 结算模式 | 按量付费 | |
| 实例类型 | 单机 | 降低规格 |
| 地域 | cn-north-4 | 与 VPC 同一区域 |
| 数据库版本 | Mongo 3.4 | |
| 规格 | 选择可用的最低规格 | 如 gaussdb.mongodb.xlarge.x2 |
| 存储空间 | 20GB | |
| VPC | ecommerce-platform-vpc | 选择已创建的 VPC |
| 子网 | ecommerce-platform-subnet | 选择已创建的子网 |
| 安全组 | ecommerce-platform-sg | 选择已创建的安全组 |

### 2.3 设置密码

1. 设置数据库密码
2. 点击 **立即购买**

### 2.4 等待创建完成

1. 实例创建需要 10-15 分钟
2. 状态变为 **运行中** 后，记下连接地址和端口（默认 8635）

---

## 步骤 3：创建 APIG

### 3.1 进入 APIG 创建页面

1. 在控制台顶部搜索框输入 **"API网关"**，点击搜索结果
2. 进入 **API网关 APIG** 页面
3. 点击 **实例管理**
4. 点击 **创建实例** 按钮

### 3.2 配置实例参数

| 配置项 | 推荐值 | 说明 |
|--------|--------|------|
| 结算模式 | 按量付费 | |
| 实例版本 | BASIC | 基础版 |
| 地域 | cn-north-4 | 与 VPC 同一区域 |
| 实例名称 | ecommerce-platform-apig | |
| VPC | ecommerce-platform-vpc | 选择已创建的 VPC |
| 子网 | ecommerce-platform-subnet | 选择已创建的子网 |
| 安全组 | ecommerce-platform-sg | 选择已创建的安全组 |

### 3.3 等待创建完成

1. 实例创建需要 3-5 分钟
2. 状态变为 **运行中** 后，记下：
   - 访问地址（ingress_address）

### 3.4 创建 API 分组

1. 在 APIG 页面左侧菜单点击 **API管理** -> **API分组**
2. 点击 **创建分组**
3. 配置：
   - 分组名称：`ecommerce-platform-api-group`
   - 所属实例：选择刚创建的 APIG 实例
4. 点击 **确定**

---

## 步骤 4：更新 Terraform 配置

手动创建完成后，可以更新 Terraform 配置引用这些资源：

```hcl
# 在 database.tf 中添加：

data "huaweicloud_rds_instance" "mysql" {
  name = "ecommerce-platform-mysql"
}

output "mysql_endpoint" {
  value = data.huaweicloud_rds_instance.mysql.private_ips[0]
}
```

---

## 资源清理

如需释放资源，在各服务控制台：

1. **RDS**：实例管理 -> 选择实例 -> 更多 -> 删除
2. **GaussDB**：实例管理 -> 选择实例 -> 更多 -> 删除
3. **APIG**：实例管理 -> 选择实例 -> 更多 -> 删除

---

## 费用估算

| 资源 | 规格 | 参考价格/小时 |
|------|------|--------------|
| RDS MySQL | 单机 2核4G | 按小时 |
| RDS MySQL | 单机 2核4G | 包月 |
| APIG BASIC | 默认 | 按小时 |
| 合计 | - | 以控制台实付为准 |

---

## 常见问题

### Q: 创建时提示"规格不可用"怎么办？
A: 选择更低的规格，或更换可用区

### Q: 账号没有创建 RDS/GaussDB 权限怎么办？
A: 联系华为云客户经理申请开通权限

### Q: VPC/子网/安全组找不到怎么办？
A: 确认区域选择为 cn-north-4
