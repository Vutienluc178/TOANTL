# Math Tools Hub — Ultra+ (Convenience)
Tính năng tiện lợi:
- **PWA**: Cài như app, chạy offline.
- **Teacher Mode** (PIN mặc định: 2703) để ẩn/hiện mục GVCN.
- **Multi-Tag** (chế độ 'any' / 'all'), **Share link**, **Random**, **In danh sách**.
- **Export/Import** ⭐ Yêu thích & Gần đây (JSON).

Triển khai:
1) Push lên nhánh `main` (Settings → Pages: Source = GitHub Actions).
2) Workflow `.github/workflows/pages.yml` (đã kèm) sẽ build `manifest.json` + deploy.
Cách nhanh nhất (không sửa code)

Tạo một file .html “shortcut” trong thư mục tools/ (vẫn xuất hiện như công cụ), và chỉ việc gắn meta chỉ đến trang ngoài.

Ví dụ: tools/khac-geogebra.html
<!DOCTYPE html><html lang="vi"><head>
<meta charset="utf-8"/>
<title>Khác — GeoGebra</title>
<meta name="viewport" content="width=device-width, initial-scale=1"/>
<meta name="tool-category" content="khac">
<meta name="tool-tags" content="geogebra, hinh-hoc, khac">
<meta name="tool-external" content="https://www.geogebra.org/">
</head><body>
<h1>GeoGebra</h1>
<p>Shortcut mở GeoGebra (ngoài trang). Thẻ này chỉ để đưa vào danh sách và gắn tag.</p>
</body></html>
MỌI TRANG ĐỀU CÓ
<meta name="tool-category" content="gvcn|lop|khac">
<meta name="tool-grade"    content="10|11|12">        <!-- khi category=lop -->
<meta name="tool-track"    content="dai-so|hinh-hoc">  <!-- khi category=lop -->
<meta name="tool-tags"     content="...">
Tạo file: .github/workflows/pages.yml với nội dung này (bản deploy ở ROOT):
name: Pages

on:
  push:
    branches: ["main"]     # đổi nếu nhánh mặc định của repo là 'master'
  workflow_dispatch: {}

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      # 1) Build manifest.json từ thư mục tools/
      - run: node scripts/build-manifest.mjs
      # 2) Đóng gói artifact để Pages deploy
      - uses: actions/upload-pages-artifact@v3
        with:
          path: .                 # deploy toàn bộ root (có manifest.json)
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
