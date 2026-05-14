---
name: "xlb-edit-modal"
description: "实现可编辑的 XLB 弹窗列表（新增/删除/行内编辑/校验）。在新建或改造可编辑弹窗时调用。"
---

# XLB 可编辑弹窗实现

该技能总结了使用 XLB 组件实现可编辑弹窗页面的模式，来源于“汇总维护/查询维护”等实现实践。

## 何时调用

- 新建包含“表格行内编辑”的弹窗页面
- 改造弹窗行为：新增/删除、字段来源、校验逻辑
- 接入后端接口（查询/更新），并统一 snake_case 入参
- 保持 UI/交互一致性：可搜索下拉、输入方式、条件样式

## 能力范围

- 字段来源 API + options 映射，并用 table_name.column_name 作为唯一键
- 行内编辑（XlbSelect/XlbInput），默认值与受控状态
- 新增/删除行（uid 作为主键，新增为空）
- 提交前必填校验并提示具体行号
- 表格条件样式（有数据才显示下边框）
- 弹窗宽度按动作类型统一管理

## 设计规范

1. 数据加载
   - 字段列表接口用于下拉选项
   - 详情查询接口用于初始行
   - options 形如 `{ value: "{table_name}.{column_name}", label }`，并保留 `fieldMap`

2. 状态结构
   - 行数据包含 `uid`、基础字段，以及嵌套配置对象（`summary_data` / `query_data`）
   - 请求/响应/本地状态统一使用 snake_case

3. 编辑控件
   - 字段名称：可搜索下拉，清空时重置联动字段
   - 输入方式/汇总方式：下拉可清空
   - 顺序（num）：输入框转 number 或 undefined

4. 新增/删除
   - 新增：push 一行空数据并生成 `uid: uId()`
   - 删除：按勾选 uid 过滤并清空选中态

5. 校验
   - 提交前校验必填项，提示格式：`第{index}行{字段}不能为空`

6. 提交结构
   - 外层 `{ table_name, details: [...] }`
   - details 每一项与行数据一致，并包含嵌套配置对象

7. UI 细节
   - 表格无数据时不显示下边框
   - 弹窗宽度：FIELD_MAINTENANCE/QUERY_MAINTENANCE 为 1100；SUMMARY_MAINTENANCE 为 720

## 示例组件

- 汇总维护：字段名称 + 汇总方式；`columnSummaryFind` 查询、`columnSummaryUpdate` 更新
- 查询维护：顺序/字段/类型/必填/默认值/输入方式/限制；`columnQueryFind` 查询、`columnQueryUpdate` 更新

## 备注

- 后端保证下划线字段时，移除驼峰兼容逻辑
- 选项唯一键需包含表名与字段名，避免主/子表重名冲突
