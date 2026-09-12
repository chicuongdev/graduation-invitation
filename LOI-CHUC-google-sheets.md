# Nối lời chúc về Google Sheets

Mặc định lời chúc chỉ nằm trong trình duyệt của từng người — bạn không nhận được, và
khách cũng không thấy lời chúc của nhau. Làm 4 bước dưới đây là xong.

Sau khi xong: lời chúc của mọi người chảy về một Google Sheet của bạn, **và sổ lưu bút
trên trang sẽ hiển thị lời chúc của tất cả mọi người** (ai vào cũng đọc được).

---

## Bước 1 — Tạo Google Sheet

Tạo một Google Sheet mới. Hàng đầu tiên đặt tiêu đề cột (để dễ đọc):

| A | B | C | D | E | F | G |
| --- | --- | --- | --- | --- | --- | --- |
| Thời gian | Loại | Họ tên | SĐT | Lời chúc | IP | Trình duyệt |

Lấy `SHEET_ID` trong URL:
`https://docs.google.com/spreadsheets/d/1k4SbwafyOogYP5JrIYymmKjJtAKv2-ZJjs8d1RU8fsw/edit`

## Bước 2 — Dán Apps Script

Trong Sheet: **Extensions → Apps Script**, xoá hết code mẫu, dán đoạn này:

```js
var SHEET_ID   = '1k4SbwafyOogYP5JrIYymmKjJtAKv2-ZJjs8d1RU8fsw';
var SHEET_NAME = 'loichuc';   // đổi nếu tab của bạn tên khác

function sheet_() {
  return SpreadsheetApp.openById(SHEET_ID).getSheetByName(SHEET_NAME);
}

// Nhận lời chúc / xác nhận tham gia gửi lên
function doPost(e) {
  sheet_().appendRow([
    new Date(),
    e.parameter.type,      // "loi-chuc" hoặc "xac-nhan-tham-gia"
    e.parameter.name,
    e.parameter.phone,
    e.parameter.message,
    e.parameter.ip,        // IP public của khách (chống spam)
    e.parameter.ua         // Trình duyệt / thiết bị
  ]);
  return ContentService.createTextOutput('ok');
}

// Trả lời chúc về cho trang web hiển thị
function doGet(e) {
  var rows = sheet_().getDataRange().getValues();
  var out = [];
  for (var i = 1; i < rows.length; i++) {
    if (rows[i][1] !== 'loi-chuc') continue;   // bỏ qua dòng chỉ xác nhận tham gia
    if (!rows[i][4]) continue;                 // bỏ qua dòng không có lời chúc
    out.push({
      name: rows[i][2],
      msg:  rows[i][4],
      time: Utilities.formatDate(new Date(rows[i][0]), 'Asia/Ho_Chi_Minh', 'HH:mm · d/M')
    });
  }
  out.reverse();   // mới nhất lên đầu

  var json = JSON.stringify(out);
  if (e && e.parameter && e.parameter.callback) {
    return ContentService
      .createTextOutput(e.parameter.callback + '(' + json + ')')
      .setMimeType(ContentService.MimeType.JAVASCRIPT);
  }
  return ContentService.createTextOutput(json).setMimeType(ContentService.MimeType.JSON);
}
```

## Bước 3 — Deploy

**Deploy → New deployment → chọn type "Web app"**

- Description: gì cũng được
- Execute as: **Me**
- Who has access: **Anyone** ← bắt buộc, nếu để "Anyone with Google account" thì khách sẽ lỗi

Bấm Deploy, cho phép quyền truy cập, rồi copy **Web app URL** — dạng:
`https://script.google.com/macros/s/AKfy..../exec`

## Bước 4 — Dán URL vào trang

Mở `index.html`, tìm dòng:

```js
var SHEET_URL = "";
```

Sửa thành:

```js
var SHEET_URL = "https://script.google.com/macros/s/AKfy..../exec";
```

Push lại lên GitHub → Vercel tự deploy. Xong.

---

## Kiểm tra

1. Mở trang, điền tên + lời chúc, bấm **Gửi lời chúc**
2. Mở Google Sheet — phải thấy một dòng mới
3. Mở lại trang bằng trình duyệt ẩn danh — phải thấy lời chúc vừa gửi trong sổ lưu bút

Nếu sổ lưu bút trống: mở Console (F12) xem lỗi. Hay gặp nhất là quên đặt
**Who has access: Anyone** ở bước 3.

## Lưu ý

- **Mỗi lần sửa Apps Script phải Deploy lại** (`Deploy → Manage deployments → ✏️ → Version: New version`),
  nếu không URL cũ vẫn chạy code cũ.
- Lời chúc hiện công khai cho mọi người xem. Nếu muốn duyệt trước, thêm cột F "Duyệt",
  và trong `doGet` thêm điều kiện `if (rows[i][5] !== 'x') continue;` — chỉ dòng bạn đánh dấu `x` mới hiện.
- Số điện thoại **không** được trả về trang web, chỉ nằm trong Sheet của bạn.
- IP và User-Agent cũng chỉ nằm trong Sheet, không trả về trang. IP lấy qua
  `api.ipify.org` — nếu service này bị chặn thì cột IP để trống, không ảnh hưởng
  đến việc ghi lời chúc.
