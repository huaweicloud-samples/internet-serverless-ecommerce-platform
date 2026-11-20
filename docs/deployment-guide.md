# 部署指南：无服务器电商平台

## 前置条件

1. 华为云账号
2. AK/SK 凭证
3. Terraform >= 1.5
4. cn-north-4 区域

## 步骤 1：配置凭证

```bash
export HW_ACCESS_KEY="your-access-key"
export HW_SECRET_KEY="your-secret-key"
```

## 步骤 2：初始化

```bash
cd infra
terraform init
```

## 步骤 3：部署

```bash
terraform apply
```

## 步骤 4：验证

部署完成后，查看输出：

```bash
terraform output
```

获取 APIG URL：

```bash
terraform output apig_url
```

### API 测试

```bash
# 用户注册
curl -X POST https://{apig-url}/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username": "testuser", "password": "testpass"}'

# 获取商品列表
curl https://{apig-url}/products
```

## 步骤 5：清理

```bash
terraform destroy
```

## 手动创建资源

以下资源需要手动创建（Terraform Provider 限制）：

1. **Apollo Server (GraphQL)** - 部署在 CCE 上
2. **CI/CD 流水线** - 使用 DevStar

## 故障排除

### FunctionGraph 调用失败

检查：
- 函数超时配置
- 内存是否足够
- VPC 网络配置

### RDS MySQL 连接失败

检查：
- 安全组是否允许访问
- 端点地址是否正确

### APIG 404

检查：
- API 路径是否正确
- 函数是否已部署
