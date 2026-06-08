# 订货会 MVP 后台接入说明

这个页面目前可以本机演示：客户选款、填写意见、提交后会进入页面里的“统计后台”。

正式发给客户使用时，需要把提交数据接到 Google 表格。接好后，客户从外部链接提交的数据会进入你的表格后台。

## 你需要准备

1. 一个 Google 表格，表名建议叫：`品牌订货会MVP_客户选款统计后台`
2. 表格第一行放下面这些表头：

```text
提交时间,订单号,客户码,客户姓名,店铺/公司,联系方式,城市,款号,款名,风格,衣长,颜色,尺码,数量,单价,金额,款式意见,整体意见
```

## Apps Script 代码

在 Google 表格里打开：

`扩展程序 -> Apps Script`

把下面代码粘进去，然后部署为 Web App。

```javascript
const SHEET_NAME = "Sheet1";

function doPost(e) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
  const data = JSON.parse(e.postData.contents);
  const rows = data.items.map(item => [
    data.createdAt,
    data.orderId,
    data.visitorCode || data.client.accessCode || "",
    data.client.name,
    data.client.shop,
    data.client.contact,
    data.client.city,
    item.productId,
    item.name,
    item.style,
    item.length,
    item.color,
    item.size,
    item.qty,
    item.price,
    item.qty * item.price,
    item.feedback || "",
    data.client.remark || ""
  ]);
  sheet.getRange(sheet.getLastRow() + 1, 1, rows.length, rows[0].length).setValues(rows);
  return ContentService.createTextOutput(JSON.stringify({ ok: true }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

## 部署设置

部署时建议选择：

```text
执行身份：我
谁可以访问：任何人
```

部署完成后，Google 会给你一个 Web App URL。

把这个 URL 填到 `品牌订货会MVP.html` 里的这一行：

```javascript
const BACKEND_ENDPOINT = "";
```

改成：

```javascript
const BACKEND_ENDPOINT = "你的 Google Web App URL";
```

## 后台统计方式

最简单的统计可以直接在 Google 表格里做数据透视表：

1. 按 `款号` 汇总 `数量`，看热款排行
2. 按 `颜色` 汇总 `数量`，看热门颜色
3. 按 `客户姓名/店铺` 查看每个客户选款
4. 筛选 `款式意见`，集中看客户修改建议

后面也可以继续做一个专门的网页后台，直接读取 Google 表格数据并生成图表。
