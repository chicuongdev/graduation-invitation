# Thiệp mời lễ tốt nghiệp — Nguyễn Chí Cường

Trang web tĩnh (HTML/CSS/JS thuần), **không cần build**, deploy thẳng lên Vercel.

## Cấu trúc thư mục

```
.
├── index.html          ← toàn bộ trang (HTML + CSS + JS trong 1 file)
├── image/              ← ảnh của bạn
│   ├── avatar.jpg
│   ├── ae01.jpg
│   ├── check-in-viettel.jpg
│   ├── lover.jpg
│   ├── lover01.jpg
│   ├── lover02.jpg
│   ├── lover04.jpg
│   ├── my-company.jpg
│   └── thanksparty.jpg
└── music/
    └── tinh-ve.mp3     ← file nhạc nền
```

Copy thư mục `image/` và `music/` của bạn vào cạnh `index.html` là xong.

## Chạy thử ở máy

Mở trực tiếp `index.html` bằng trình duyệt là chạy được. Nếu muốn giống môi trường thật:

```bash
npx serve .
```

## Đưa lên GitHub

```bash
cd <thư-mục-này>
git init
git add .
git commit -m "Thiệp mời lễ tốt nghiệp 27.09.2026"
git branch -M main
git remote add origin https://github.com/<tên-github>/<tên-repo>.git
git push -u origin main
```

## Deploy lên Vercel

1. Vào https://vercel.com → **Add New… → Project**
2. Chọn repo vừa push → **Import**
3. Framework Preset: **Other** · Build Command: để trống · Output Directory: để trống · Root Directory: `./`
4. Bấm **Deploy**

Xong. Mỗi lần `git push` là Vercel tự deploy lại.

## 3 chỗ cần sửa trong `index.html`

| Cần sửa | Tìm | Ghi chú |
| --- | --- | --- |
| Số điện thoại | `0900000000` và `0900 000 000` | có 2 chỗ, trong card "Liên hệ" |
| Nhận lời chúc thật | `var SHEET_URL = "";` | dán URL Apps Script (xem dưới) |
| Tên file nhạc | `var MUSIC_SRC = "music/tinh-ve.mp3";` | đổi nếu file bạn tên khác |

## Nhận lời chúc về Google Sheets

1. Tạo một Google Sheet mới, lấy `SHEET_ID` từ URL.
2. **Extensions → Apps Script**, dán:

```js
function doPost(e) {
  SpreadsheetApp.openById('SHEET_ID').getSheetByName('Sheet1').appendRow([
    new Date(),
    e.parameter.type,      // "loi-chuc" | "xac-nhan-tham-gia"
    e.parameter.name,
    e.parameter.phone,
    e.parameter.message
  ]);
  return ContentService.createTextOutput('ok');
}
```

3. **Deploy → New deployment → Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
4. Copy URL (dạng `https://script.google.com/macros/s/.../exec`) dán vào `SHEET_URL`.

Khi `SHEET_URL` để trống, lời chúc vẫn hiện trong sổ lưu bút nhưng chỉ lưu ở trình duyệt người xem —
bạn sẽ không nhận được.

## Lưu ý về nhạc nền

Trình duyệt chặn tự phát nhạc. Trang xử lý bằng cách: hiện thông báo khi mở trang, và bật nhạc
ngay khi người xem chạm/click lần đầu vào bất kỳ đâu. Bấm đĩa DVD để bật/tắt thủ công.
