# Rust-Overview

**Personal opinion**: viết contract cosmwasm cũng na ná viết updgradable contract theo Proxy pattern trong solidity, sẽ có mọt file chính quản lý khởi tạo, gọi hàm, storage,... rồi gọi logic của các logic contract.

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
- This will generate new JSON files, for our config, poll and ballot `cargo schema`
- Still 0 tests but should pass `cargo test`
- Will generate the wasm under target `cargo wasm`

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

**Note**: Khi compile nó sẽ compile lib.rs rồi từ đó compile khác mod được import bên trong
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
- các `String` truyền vào khi khởi tạo các biến trạng thái (`Item`, `Map`,...) là các key định danh trong `blockchain storage` để truy xuất dữ liệu

| **Cấu trúc** | **Chuỗi truyền vào `::new()` là gì?** | **Ý nghĩa** |
|-------------|---------------------------------|----------|
| `Item<T>::new("key")` | `"key"`  | Key duy nhất trong storage. |
| `Map<K, V>::new("prefix")` | `"prefix"` | Tiền tố để tạo key động. |
| `Bucket<K, V>::new("namespace")` | `"namespace"` | Giúp phân vùng dữ liệu. |
| `SnapshotMap<K, V>::new("key", "checkpoints", "changelog")` | `"key"`, `"checkpoints"`, `"changelog"` | Lưu dữ liệu có thể truy vấn theo block height. |

📌 Các String truyền vào ::new() trong cw-storage-plus là gì?

Khi bạn gọi ::new("some_string") trong Item<T>, Map<K, V>, Bucket<K, V>, v.v., chuỗi "some_string" chính là key định danh trong blockchain storage.

1️⃣ **Item<T>::new("key")**

🔹 Ví dụ:

```rust
pub const CONFIG: Item<Config> = Item::new("config");
```

`"config"` là tên key trong storage.

Mọi dữ liệu của `CONFIG` sẽ được lưu vào key `"config"`.

Khi truy xuất dữ liệu, contract sẽ tìm trong storage với key "config".

📌 **Tóm lại**:

Mỗi Item<T> có một key duy nhất, giúp truy xuất dễ dàng.

2️⃣ **Map<K, V>::new("prefix")**

🔹 Ví dụ:
```rust
pub const POLLS: Map<String, Poll> = Map::new("polls");
```

`"polls"` là prefix (tiền tố) trong storage.

Mọi dữ liệu lưu trong POLLS sẽ có key dạng:

`"polls" + poll_id`

Nếu `poll_id = "123"`, thì key trong storage sẽ là `"polls123"`.

📌 **Tóm lại:**

`Map<K, V>` không có một key cố định, mà nó là một prefix để tạo ra nhiều key khác nhau.

3️⃣ **Bucket<K, V>::new("namespace")**

🔹 Ví dụ:
```rust
pub const USER_DATA: Bucket<u128> = Bucket::new("user_data");
```

`"user_data"` là namespace.

Tương tự `Map`, nhưng dữ liệu sẽ được lưu với namespace `"user_data"`.

📌 **Tóm lại:**

`Bucket<K, V>` giúp tránh trùng key khi có nhiều nhóm dữ liệu khác nhau.

4️⃣ **SnapshotMap<K, V>::new("key", "checkpoints", "changelog")**

🔹 Ví dụ:
```rust
pub const STAKE_BALANCES: SnapshotMap<&Addr, u128> = SnapshotMap::new(
    "stake_balances",
    "stake_balances__checkpoints",
    "stake_balances__changelog",
);
```

`"stake_balances"`: Prefix chính của dữ liệu staking.

`"stake_balances__checkpoints"`: Lưu checkpoint (dữ liệu theo block height).

`"stake_balances__changelog"`: Lưu lịch sử thay đổi.

📌 **Tóm lại:**

SnapshotMap<K, V> giúp lưu dữ liệu theo thời gian để có thể truy xuất lịch sử.


## Module viết test 
```rust
mod tests {
    use cosmwasm_std::attr;
    //module attr, helper mod
    //tạo và sử dụng các thuộc tính(attributes)
    // e.g. : ("action", "instantiate")

    use cosmwasm_std::testing::{mock_dependencies, mock_env, mock_info};
    //các hàm giả lập (mock function to mock an envirionment, message info, dependencies) 
    //mock_dependencies tạo ra một đối tượng giả lập cho các Deps trong môi trường CosmWasm. Nó bao gồm các phần như bộ lưu trữ (storage), API, và querier mà hợp đồng sẽ sử dụng. Điều này giúp bạn kiểm tra các hành động như ghi dữ liệu vào bộ lưu trữ mà không cần một blockchain thật.
    //mock_env tạo ra một đối tượng giả lập cho môi trường (Env) mà hợp đồng thông minh chạy trong đó. Nó bao gồm các thông tin như thời gian, địa chỉ của người gọi, và các yếu tố khác liên quan đến môi trường thực thi.
    //mock_info giúp tạo ra thông tin giả lập cho MessageInfo, bao gồm địa chỉ người gọi và các tiền tệ gửi kèm (nếu có). Đây là đối tượng chứa các thông tin về người gửi giao dịch

    use crate::contract::instantiate; // hàm init của contract
    use crate::msg::InstantiateMsg; // 


    //các account giả lập
    pub const ADDR1: &str = "addr1";
    pub const ADDR2: &str = "addr2";





}

```

cau truc chuan
cargo.toml o root phai khai bao workspace
```bash
cw-starter/
├── Cargo.lock         # File lock cho dependencies (tự động tạo)
├── Cargo.toml         # Định nghĩa workspace và dependencies chung
├── contracts/         # Chứa các smart contracts
│   ├── my_contract/   # Một smart contract cụ thể
│   │   ├── Cargo.toml  # Định nghĩa dependencies cho contract
│   │   ├── src/       # Chứa code của contract
│   │   │   ├── contract.rs  # Xử lý logic chính
│   │   │   ├── lib.rs       # Điểm vào chính của contract
│   │   │   ├── msg.rs       # Định nghĩa message (instantiate, execute, query)
│   │   │   ├── state.rs     # Lưu trữ dữ liệu contract
│   │   │   ├── error.rs     # Định nghĩa lỗi (tùy chọn)
│   │   ├── schema/    # (Tùy chọn) Chứa schema JSON cho contract
│   │   ├── tests/     # (Tùy chọn) Test contract bằng Rust
├── scripts/           # Chứa script deploy, interact với contract
├── tests/             # (Tùy chọn) Test tổng hợp cho tất cả contracts
├── examples/          # (Tùy chọn) Chứa mẫu sử dụng contract
└── artifacts/         # Chứa file .wasm đã build và tối ưu

```


Lenh toi uu wasm
//Optimize nhieu contract
```

docker run --rm -v "${PWD}:/code" `
  --mount type=volume,source="$(Get-Item -Path ${PWD} | Select-Object -ExpandProperty Name)_cache",target=/code/target `
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry `
  cosmwasm/workspace-optimizer:0.14.0

```
//Optimize 1 contracts
```

docker run --rm -v "D:/desktop/Security_Research/Cosmos/Rust Learning/rustlings/cw-starter:/code" `
  --mount type=volume,source="cw-starter_cache",target=/code/target `
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry `
  cosmwasm/rust-optimizer:0.16.0

```
cac function and object
```
Hàm	Chức năng
Secp256k1HdWallet.fromMnemonic(mnemonic, options?)	Tạo ví từ mnemonic
setupWebKeplr()	Kết nối với Keplr Wallet (nếu có)
coin(amount, denom)	Tạo một đối tượng token (Coin)
toBinary(obj)	Chuyển JSON thành nhị phân
```

```
Đối tượng	Chức năng
SigningCosmWasmClient	Client để gửi giao dịch có chữ ký
CosmWasmClient	Client để chỉ đọc dữ liệu từ blockchain
UploadResult	Kết quả upload contract
InstantiateResult	Kết quả khởi tạo contract
```



✅ dyn dùng để tạo trait object cho phép dynamic dispatch.
✅ dyn Iterator giúp Rust xử lý iterator có kích thước không cố định tại compile time.
✅ Dùng Box<dyn Iterator> để tối ưu bộ nhớ và tránh copy dữ liệu lớn trên stack.



Boxes don’t have performance overhead, other than storing their data on the heap instead of on the stack. But they don’t have many extra capabilities either. You’ll use them most often in these situations:

When you have a type whose size can’t be known at compile time and you want to use a value of that type in a context that requires an exact size
When you have a large amount of data and you want to transfer ownership but ensure the data won’t be copied when you do so
When you want to own a value and you care only that it’s a type that implements a particular trait rather than being of a specific type


## **Note**
- Khi lay reference thi luon co the dereference
example

```rust
let mut x = 5;
let y = &mut x;
println!("{}", *y);

struct P{
value: u64
};

let mut x = P {value: 5};
let y = &mut x;
(*y).value = 8;
....
```

**=> lay reference chinh la lay con tro**
```rust
trait Speak {
    fn speak(&self);
}

struct Dog;
struct Cat;

impl Speak for Dog {
    fn speak(&self) {
        println!("Woof!");
    }
}

impl Speak for Cat {
    fn speak(&self) {
        println!("Meow!");
    }
}

fn say_hello(animal: &dyn Speak) {
    animal.speak();
}

fn main() {
    let dog = Dog;
    let cat = Cat;

    say_hello(&dog); // Woof!
    say_hello(&cat); // Meow!
}
```
Ở đây, say_hello nhận vào tham chiếu đến dyn Speak, nên nó có thể chấp nhận bất kỳ kiểu nào 

triển khai Speak — dù là Dog, Cat, hay các kiểu khác.
