# XlbFetch Behavior Reference

Use this file when you need the concrete project evidence behind the skill rules.

## Component Library Baseline

Source: `D:\code\xlb-components\src\utils\xlbFetch.tsx`

- Lines 125-137: when `data.code !== 0`, `XlbTipsModal({ tips: data.msg })` is shown and the response is still returned with `Promise.resolve(data)`.
- Lines 143-166: HTTP `404`, `401`, `500`, `502` each show a default message and return a normalized `{ code, msg }` object instead of throwing to business code.
- Lines 171-176: generic `Error` also shows a modal and returns `{ code: -1, msg: error.message }`.

Practical conclusion:

- Business pages normally receive a resolved response object even for common failures.
- Re-throwing or re-prompting the same error in business code usually creates duplicate feedback.

## FSS Project Global Extensions

Source: `D:\code\xlb_fss_test0417\xlb_fss_web\src\app.tsx`

- Lines 41-48: `XlbFetch.register([401, 1005], ...)` adds global reload/token recovery behavior.
- Lines 51-53: `XlbFetch.register([2005], ...)` adds a global `message.error(data.msg)`.
- Lines 100-127: project-level interceptors record non-zero responses for failure tracking.

Practical conclusion:

- `2005` is not a plain silent return in this project; it already has global prompt behavior.
- Non-zero responses may already participate in tracking, so business code should stay light unless there is a clear page-level need.

## FSS Project Request-Side Defaults

Source: `D:\code\xlb_fss_test0417\xlb_fss_web\src\provider.tsx`

- Lines 26-44: request interceptor sets `timeout = 70000`.
- Line 31: non-GET requests auto-append `company_id`.
- Lines 33-40: object payloads filter out `undefined`, and query-style interfaces with `find` / `page` / `detail` convert empty string values to `undefined`.
- Line 33: `import` and `upload` requests are excluded from this object cleanup rule.

Practical conclusion:

- Standard CRUD/query interfaces usually do not need to hand-write `company_id`.
- Query interfaces should not add another layer of empty-string cleanup unless they bypass this interceptor.

## Recommended Service Pattern

```ts
import { XlbFetch } from '@xlb/utils'

export const pageApi = async (data: any) => {
  return await XlbFetch.post('/fss/hxl.xxx.page', data)
}
```

```ts
const res = await pageApi(params)
if (res.code === 0) {
  setData(res.data)
}
```

Only add extra handling when the page truly needs custom success flow or custom non-zero-code branching.
