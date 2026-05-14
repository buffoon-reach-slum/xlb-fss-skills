---
name: "xlbpagecontainer-list-page"
description: "生成符合 FSS 代码风格的 XlbPageContainer 列表页（查询/表格/导出/枚举下拉）。当你要新建或改造列表页为“门店费用式通用导出+查询表格”写法时调用。"
---

# XlbPageContainer 列表页（FSS 风格）

## 适用场景

- 需要用 `XlbPageContainer` 快速搭一个“查询区 + 工具栏 + 表格”的标准列表页
- 查询条件含枚举下拉：通过枚举接口拉取 `value/label`，用于筛选与表格显示
- 导出走“通用导出（受理）”模式：点击导出调用导出接口，提示“导出受理成功，请前往下载中心查看”

## 参考范式

以 `src/pages/fidManagement/` 的写法为基准：

- `index.tsx`：页面容器 + 查询 + 导出按钮 + 枚举加载
- `data.tsx`：`SearchFormList` 与 `tableColumn` 的配置集中管理
- `server.ts`：接口封装（枚举 / 查询 / 导出）

## 目录与文件拆分

建议按以下结构落地（与现有模块一致）：

- `index.tsx`：页面逻辑与组合
- `data.tsx`：表单/表格配置（尽量纯配置，少逻辑）
- `server.ts`：请求封装（全部走 `XlbFetch.post`）

## 标准实现步骤

### 1) server.ts：封装枚举/查询/导出

- 枚举接口返回统一 `code/msg/data`，定义 `ApiResponse<T>` 与 `EnumItem`
- 使用 `process.env.BASE_URL + '/xxx'` 的拼接方式需与项目同类模块保持一致
- 导出接口采用“受理导出”返回：`code === 0` 时 `data` 常为提示文案

### 2) data.tsx：配置 SearchFormList 与 tableColumn

- `SearchFormList(options)`：把枚举 options 透传进来，做 `select` 下拉
- `tableColumn(maps)`：通过 `map[value] -> label` 渲染列文本，避免页面里写渲染逻辑

### 3) index.tsx：页面组合（XlbPageContainer 标准写法）

固定骨架（保持与业务结账/门店费用一致）：

- `const { Table, ToolBtn, SearchForm } = XlbPageContainer`
- `const [form] = Form.useForm()`
- `prevPost()`：从表单取值并做入参归一（数组/空字符串兜底）
- `useEffect()`：`Promise.all` 拉枚举，`isMounted` 防卸载后 setState
- `ToolBtn`：render props 拿 `fetchData/loading/setLoading` 统一控制 loading
- `exportItem(setLoading)`：按门店费用方式 `try/finally`，成功后 `message.success(res?.data || '导出受理成功，请前往下载中心查看')`

## 关键细节（不要偏离）

- 导出不要本地拼 CSV/Excel；优先走后端“通用导出受理”
- 导出按钮：`disabled={loading}`，并用 `setLoading(true/false)` 包裹请求
- 查询入参：确保 `appTypes/businessTypes/states` 始终为数组，`name` 为空串兜底
- 表格 `primaryKey` 选择稳定字段（如 `code`）

## 示例（精简骨架）

```tsx
const { Table, ToolBtn, SearchForm } = XlbPageContainer
const [form] = Form.useForm()

const prevPost = () => {
  const { appTypes, businessTypes, states, name } = form.getFieldsValue(true)
  return {
    appTypes: Array.isArray(appTypes) ? appTypes : appTypes ? [appTypes] : [],
    businessTypes: Array.isArray(businessTypes) ? businessTypes : businessTypes ? [businessTypes] : [],
    states: Array.isArray(states) ? states : states ? [states] : [],
    name: name || '',
  }
}

return (
  <XlbPageContainer url="/xxx.find" tableColumn={tableColumn(maps)} prevPost={prevPost} immediatePost={false}>
    <SearchForm>
      <XlbForm isHideDate form={form} formList={SearchFormList(options)} />
    </SearchForm>
    <ToolBtn showColumnsSetting={false}>
      {({ fetchData, loading, setLoading }: ContextState<any>) => (
        <XlbButton.Group>
          <XlbButton label="查询" loading={loading} type="primary" onClick={() => fetchData()} icon={<XlbIcon name="sousuo" />} />
          <XlbButton
            label="导出"
            loading={loading}
            disabled={loading}
            type="primary"
            onClick={() => exportItem(setLoading)}
            icon={<XlbIcon name="daochu" />}
          />
        </XlbButton.Group>
      )}
    </ToolBtn>
    <Table primaryKey="code" selectMode="single" />
  </XlbPageContainer>
)
```
