# Serverless Ecommerce Platform

[中文版](./README.cn.md)

![License](https://img.shields.io/badge/License-MIT-green)
![Huawei Cloud](https://img.shields.io/badge/Huawei%20Cloud-Ready-blue)

## Overview

This project is a complete serverless e-commerce platform solution built on Huawei Cloud. The platform adopts a microservices architecture design, deploying each business module as serverless functions, providing unified RESTful API services through APIG. The entire platform features high availability, high elasticity, and low spend, automatically scaling based on business traffic.

### Core Functional Modules

This e-commerce platform includes complete e-commerce business functions, covering the entire process from user registration to order completion:

1. **User Authentication Module** - Provides complete user lifecycle management functions, including user registration, login/logout, password recovery, and token refresh. The authentication module adopts a self-built JWT solution, with user data stored in RDS MySQL, supporting optional Multi-Factor Authentication (MFA). The system also provides user permission management, supporting administrator and regular user roles.

2. **Product Management Module** - Merchants can manage product information through the backend, including product name, price, inventory, description, images, and other attributes. Product management supports category management, tag management, custom product attributes, and other features. The system provides value-added services such as product search, product recommendations, and product reviews to help improve user shopping experience. Product data is stored in RDS MySQL, supporting flexible queries and filtering.

3. **Order Processing Module** - Responsible for handling core processes such as order placement, order payment, and order fulfillment. The order module works with the inventory module to automatically deduct inventory when orders are placed and automatically roll back inventory when orders are cancelled. Order status includes multiple states: pending payment, paid, shipped, completed, cancelled, etc., supporting order status change callback notifications.

4. **Payment Module** - Integrates Huawei Cloud Payment Gateway, supporting multiple payment methods (including WeChat Pay, Alipay, UnionPay, etc.). The payment module provides interfaces for payment creation, payment callback, payment query, and refunds. Payment security is guaranteed through KMS key management services, with all transaction sensitive information encrypted.

5. **Inventory Management Module** - Real-time management of product inventory quantities, supporting inventory warnings, low inventory alerts, inventory locking, and other functions. The inventory module works closely with the order module to ensure no overselling or underselling. The system supports multi-warehouse inventory management, intelligently allocating shipping warehouses based on user addresses.

6. **Delivery Service Module** - Integrates multiple express logistics services, providing waybill generation, logistics tracking, delivery confirmation, and other functions. The delivery module supports multiple delivery methods (standard express, expedited delivery, same-city delivery, etc.), automatically matching the optimal delivery solution based on user selection and product attributes.

### Technical Architecture Features

- **Microservices Architecture**: Each business module is independently deployed with independent scaling and fault isolation
- **Serverless Computing**: Uses FunctionGraph, pay-per-request, no resource pre-provisioning required
- **Event-Driven**: Achieves event communication between modules through EventGrid, improving system response speed
- **Data Consistency**: Uses distributed transactions to ensure cross-module data consistency
- **Security Compliance**: Achieves sensitive data encryption through KMS, complying with e-commerce security standards

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Huawei Cloud Ecommerce Platform                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐        │
│  │   Users   │    │ Products │    │  Orders  │    │ Payment  │        │
│  │ Function │    │ Function │    │ Function │    │ Function │        │
│  │  Graph  │    │  Graph   │    │  Graph   │    │  Graph   │        │
│  └────┬─────┘    └────┬─────┘    └────┬─────┘    └────┬─────┘        │
│       │                │                │                │               │
│       └────────────────┼────────────────┼────────────────┘               │
│                        │                │                               │
│                        ▼                ▼                               │
│               ┌─────────────────┐  ┌─────────────────┐                 │
│               │  APIG          │  │  EventGrid     │                 │
│               │  (API Gateway) │  │  (Event Bus)   │                 │
│               └─────────────────┘  └─────────────────┘                 │
│                        │                │                               │
│                        ▼                ▼                               │
│               ┌─────────────────────────────────────┐                 │
│               │              RDS MySQL                │                 │
│               └─────────────────────────────────────┘                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Cloud Services

| Service | Purpose |
|---------|---------|
| FunctionGraph | Serverless functions |
| APIG | API Gateway |
| RDS MySQL | Relational Database |
| OBS | Object Storage |
| EventGrid | Event-driven |
| VPC | Virtual Private Cloud |
| KMS | Key Management |

## Prerequisites

- Huawei Cloud account
- AK/SK credentials
- cn-north-4 region

## Quick Start

### 1. Configure credentials

```bash
export HW_ACCESS_KEY="your-access-key"
export HW_SECRET_KEY="your-secret-key"
```

### 2. Initialize Terraform

```bash
cd infra
terraform init
```

### 3. Deploy

```bash
terraform apply
```

## Usage

### Call APIs

```bash
# User registration
curl -X POST https://{apig-url}/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username": "user1", "password": "pass123"}'

# User login
curl -X POST https://{apig-url}/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "user1", "password": "pass123"}'

# Get product list
curl https://{apig-url}/products

# Create order
curl -X POST https://{apig-url}/orders \
  -H "Content-Type: application/json" \
  -d '{"user_id": "user123", "items": [{"product_id": "1", "quantity": 2}]}'
```

## Cleanup

```bash
terraform destroy
```

## Detailed Description

### Microservices Architecture

The platform includes the following microservices:

1. **Users Service** - User authentication and management
2. **Products Service** - Product queries
3. **Orders Service** - Order creation and processing
4. **Warehouse Service** - Inventory management
5. **Delivery Service** - Delivery processing
6. **Delivery Settlement Service** - Shipping fee calculation
7. **Payment Service** - Payment processing

### Event-Driven

After order creation, EventGrid automatically triggers subsequent services:
- Order created → Warehouse → Delivery → Payment

## License

MIT License
