---
name: xlbfetch-api
description: 实现和检查基于 XlbFetch 的 FSS service 层接口。在新增或修改 `server.ts`、`service.ts` 接口封装，判断非 0 响应是否还需要手动抛错，或避免重复错误提示时调用，因为 XlbFetch 已经处理了常见请求异常和业务报错。
---

# XlbFetch API

Use this skill when writing or refactoring interface wrappers that call `XlbFetch` in FSS projects.

If the task also changes request/response field naming or backend field mapping, use `$api-camel-to-snake` together with this skill.

## Default Rule

Treat `XlbFetch` as the first layer of error handling.

For ordinary business interfaces:

```ts
import { XlbFetch } from '@xlb/utils'

export const fetchSomething = async (data: any) => {
  return await XlbFetch.post('/fss/xxx.xxx.xxx', data)
}
```

At the call site, branch on `res.code` instead of throwing the same error again:

```ts
const res = await fetchSomething(params)
if (res.code === 0) {
  // success flow
  return
}

// Usually stop here. XlbFetch has already shown the common error prompt.
```

## What XlbFetch Already Handles

Read `references/xlbfetch-behavior.md` if you need the source evidence.

Assume these cases are already handled before business code sees the response:

- Business failure with `code !== 0`: `XlbFetch` shows `XlbTipsModal({ tips: data.msg })` and still returns the response object.
- `404`: shows `message.error("接口未找到")` and returns `{ code: 404, msg }`.
- `401`: shows `message.error("该用户无权限")` and returns `{ code: 401, msg }`.
- `500`: shows `message.error("服务器错误")` and returns `{ code: 500, msg }`.
- `502`: shows `message.warning("服务器正在重启，请稍后")` and returns `{ code: 502, msg }`.
- Other thrown `Error`: shows `XlbTipsModal(error.message)` and returns `{ code: -1, msg }`.

Because of that, do not add the same `throw new Error(res.msg)`, `Promise.reject(res)`, or duplicate `message.error(res.msg)` in normal business flows.

## FSS Project Add-ons

In this repo, `XlbFetch` has extra global behavior:

- `401` and `1005` are registered globally for reload/token refresh handling.
- `2005` is registered globally to show `message.error(data.msg)`.
- Non-zero responses are also collected by the project response interceptor for failure tracking.
- Request interceptor auto-adds `company_id` for non-GET requests.
- Query-style interfaces containing `find` / `page` / `detail` convert empty string values to `undefined`.
- Request timeout is increased to `70000`.

So when adding a normal `XlbFetch` interface in this repo, do not manually re-add `company_id`, do not manually strip empty query strings again, and do not manually duplicate the standard error prompt.

## When Manual Handling Is Still Needed

Manual handling is still reasonable in these cases:

- You need to branch on a specific business code, such as `2005`, and continue a special UI flow.
- You need success-only feedback such as `"保存成功"` or refresh actions after `res.code === 0`.
- The backend returns structured error details like `res.data.error_messages`, and the page must render them in a custom modal/list.
- The request does not go through `XlbFetch`, such as third-party upload, native `fetch`, SSE, or other custom clients.
- The product explicitly requires a custom interaction instead of the default prompt.

## Review Checklist

- Service wrappers stay thin and mainly return the `XlbFetch` call result.
- Callers judge `res.code === 0` for success and avoid duplicate error throwing.
- Special code handling is added only when the page really has a custom branch.
- If field naming is being adapted for the backend, also apply `$api-camel-to-snake`.
