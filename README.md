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

  ## **Smart Contract on Cosmos**
  - Sử dụng template `cosmwasm`
  - Cấu trúc thư mục dự án `cosmwasm`
```bash
cw-starter/ # Root
├── .cargo/
│   └── config # Configuration for cargo commands such as cargo wasm, cargo schema, etc.
├── .circleci/
│   └── config.yml # Circle CI workflows and integration
├── .github/workflows/
│   └── Basic.yml # GitHub actions integration
├── examples/
│   └── schema.rs # Rust file to generate JSON schema via cargo schema. Outputs to schema/
├── schema/ # Output folder for JSON schema
├── src/ # Where our smart contract rust files are located
│   ├── contract.rs # Main contract logic, instantiate, execute, query
│   ├── error.rs # Where we define our contract errors
│   ├── lib.rs # Where we define our modules
│   ├── msg.rs # Where we define our message types
│   └── state.rs # Where we define any state variables
└── target/ # Where unoptimised WASM files are outputted
```
- **Command for generating CW-template**
  - Lastest Ver : `cargo generate --git https://github.com/CosmWasm/cw-template.git --name <PROJECT_NAME>`
  - Older Ver   : `cargo generate --git https://github.com/CosmWasm/cw-template.git --name cw-starter -d minimal=true`
  Example with lastVer: `cargo generate --git https://github.com/CosmWasm/cw-template.git --name cw-starter -d minimal=true`
- Giải thích một số thư mục:
  - `examples`: dùng để tạo ra các file json (kiểu như abi) -> là các "bản hướng dẫn" để tương tác với contract.
  - `schema`: chứa các file json (abi)
  - `target`: chứa các file build artifact để thao tác trên blockchain
## **Giải thích code trong src**
- state.rs:
```rust
use schemars::JsonSchema; //là một derive macro
                          // Giúp tự động tạo JSON schema từ các struct và enum 
                          // Làm việc với crate serde
use serde::{Deserialize, Serialize}; 
// Serialize : Chuyển đổi struct/enum -> JSON
// Deserialize: chuyển đổi JSON -> struct/enum

use cosmwasm_std::Addr; // làm việc với Cosmos address
use cw_storage_plus::{Item, Map}; // Lưu trữ giá trị trên chain

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)] // marco để impl trait cho struct
pub struct Config {
    // cấu trúc định nghĩa các biến state 
    pub admin: Addr, // Admin address
}

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct Poll {//Lưu thông tin Poll
    pub creator: Addr,
    pub question: String,
    pub options: Vec<(String, u64)>,
}

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct Ballot {// Lưu thông tin vote của user
    pub option: String,
}

pub const CONFIG: Item<Config> = Item::new("config"); // lưu trữ state
// định danh truy cập vào storage

pub const POLLS: Map<String, Poll> = Map::new("polls");
```
 - `use schemars::JsonSchema`:
   
   🔹 `JsonSchema` là một derive `macro` giúp tạo `JSON Schema` tự động từ các `struct` và `enum` trong `Rust`.
   
   🔹 Nó được sử dụng phổ biến trong `CosmWasm` để tạo `JSON Schema` cho messages và state.
   
   🔹 `JsonSchema` làm việc với `serde`, giúp đảm bảo rằng dữ liệu có thể được `serialize` và `deserialize` thành JSON.
- `use cosmwasm_std::Addr`:
  - Làm việc với Address trên mạng
- `cw_storage_plus`: các kiểu dữ liệu để lưu trữ state

