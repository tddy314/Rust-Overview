# Rust-Overview

**`NOTE`**: `cargo` là trình quản lý gói, thư viện (có thể em tương đương như `npm` hợp `npx`)

## **Basic command**

- `cargo new project_name` : tạo project mới
  - Cấu trúc project:
    - `src`: source code `rs`
    - `Cargo.toml`: Đây là file cấu hình chính, chứa thông tin về dự án như tên, phiên bản, tác giả, và danh sách thư viện phụ thuộc.
    - `Cargo.lock`: File này được Cargo tự động tạo để "khóa" các phiên bản chính xác của dependencies, đảm bảo dự án hoạt động đồng nhất trên mọi máy.
    - `Cargo.target`: Chứa các file thực thi
    - ..... Một số file khác nữa
- `cargo build`: compile và tạo ra file thực thi nhưng không run
- `cargo check`: chỉ compile
- `cargo run`: compile + tạo file thực thi + run
