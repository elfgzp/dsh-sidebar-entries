# Fork 修复说明

## 上游

- 包名：`dsh-sidebar-entries`
- 版本：**0.7.46**（npm 上的最新版，缺陷依然存在）
- 上游仓库地址：`https://github.com/FragrantLotus/dsh-sidebar-entries` → **返回 404，无法 fork**

## 缺陷

侧边栏面板列表顶部的折叠箭头（"one arrow to fold them all"）点下去只翻转箭头方向，
**面板图标一个都收不起来**。

### 根因

`lib/client.js` 的 `collapsibleRows()` 只在 `nav` 的**直接子元素**里找面板行：

```js
Array.prototype.slice.call(nav.children).filter((el) =>
  el.tagName === 'BUTTON' && String(el.className).indexOf('panelRow') >= 0)
```

新版 DSH 把每个面板行包进了 `div.panelEntry`（宿主用它画 hover 高亮药丸）：

```html
<nav class="…panelList">
  <div class="…panelEntry"><button class="…panelRow" aria-label="插件">…</button></div>
  <div class="…panelEntry"><button class="…panelRow" aria-label="自动化任务">…</button></div>
  …
</nav>
```

于是 `nav.children` 里全是 `DIV`，过滤器匹配不到任何行，`applyCollapsed()`
拿到的列表几乎是空的（只剩插件自己注入的 toggle 按钮，而它被显式跳过），
所以**一个都收不起来**。

### 修复

两处改动，都在 `lib/client.js`：

1. `collapsibleRows()`：改用 `nav.querySelectorAll('button')` 做**后代**匹配，
   不再只看直接子元素。
2. 新增 `collapsibleTarget()`：隐藏时优先隐藏面板行的**包装层** `div.panelEntry`
   （有包装层时），没有包装层则退回按钮本身。只隐藏按钮会在导航栏里留下包装层的
   填充和 36px 占位，列表看上去仍然没收起。

改动对没有 `div.panelEntry` 包装的旧版宿主保持向后兼容。
