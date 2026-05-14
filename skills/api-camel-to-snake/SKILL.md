---
name: "api-camel-to-snake"
description: "接口字段统一驼峰转下划线；当后端仅返回下划线字段时取消驼峰兼容。在编写/修改接口入参与字段映射时调用。"
---

# 接口字段驼峰转下划线（中文版）

## 使用规则

1. 入参统一使用下划线命名（snake_case）。
2. 若文档/示例为驼峰（camelCase），在实际请求中转换为下划线。
3. 默认不做驼峰兼容；当后端明确“仅返回下划线字段”时，去掉对 camelCase 的兼容处理。
4. 常用字段映射：
   - tableName → table_name
   - tableNameCh → table_name_ch
   - appType → app_type
   - businessType → business_type
   - orderType → order_type

## 适用场景

- 用户提供接口返回结构或请求示例，需要确定字段命名风格
- 需要编写接口参数、前端回填或字段映射逻辑

## 使用指引

1. 编写请求参数时，统一采用下划线风格。
2. 如果后端示例为驼峰，按上述映射转换为下划线再下发。
3. 当后端仅返回下划线字段：移除所有驼峰字段的读取与兼容逻辑。

## 示例

- 示例入参（文档驼峰 → 实际转换后下划线）：
  - 文档：`{ tableName: 'order', businessType: 'SALE' }`
  - 实际：`{ table_name: 'order', business_type: 'SALE' }`

- 响应读取：后端保证下划线时，仅读取 `table_name` 等下划线字段，不再读取 `tableName`。
