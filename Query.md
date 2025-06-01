## So sánh `query_wasm_smart` và `query` trong CosmWasm

### 1. `query_wasm_smart`

Đây là **hàm tiện ích (helper)** có sẵn trong CosmWasm, dùng riêng để query smart contract.

```rust
fn query_wasm_smart<T: DeserializeOwned>(
    &self,
    contract_addr: impl Into<String>,
    msg: &impl Serialize,
) -> StdResult<T>;
```

#### Tham số:

| Tham số        | Kiểu                    | Giải thích |
|----------------|--------------------------|------------|
| `contract_addr` | `impl Into<String>`     | Địa chỉ của smart contract cần query |
| `msg`           | `&impl Serialize`       | Message gửi vào, sẽ được tự động `to_binary` |
| **trả về**      | `StdResult<T>`          | Kiểu trả về mong muốn, cần implement `DeserializeOwned` |

#### Ưu điểm:
- Dễ dùng, gọn.
- Tự động mã hóa `msg` sang `Binary`.
- Tự động giải mã kết quả về kiểu cụ thể `T`.

#### Ví dụ:

```rust
let msgs: Vec<CosmosMsg> = deps.querier.query_wasm_smart(
    "contract_b_addr",
    &QueryMsg::ResolveOperation { ... },
)?;
```

---

### 2. `query`

Là hàm **tổng quát hơn**, hỗ trợ mọi loại query (`Wasm`, `Bank`, `Staking`, `Custom`,...).

```rust
fn query<T: DeserializeOwned>(&self, request: &QueryRequest<Empty>) -> StdResult<T>;
```

#### Tham số:

| Tham số   | Kiểu                         | Giải thích |
|-----------|------------------------------|------------|
| `request` | `&QueryRequest<Empty>`       | Toàn bộ cấu trúc query cụ thể bạn muốn thực hiện |
| **trả về**| `StdResult<T>`               | Kết quả trả về mong muốn, cần implement `DeserializeOwned` |

#### Ví dụ dùng `WasmQuery::Smart`:

```rust
let msgs: Vec<CosmosMsg> = deps.querier.query(
    &QueryRequest::Wasm(WasmQuery::Smart {
        contract_addr: "contract_b_addr".to_string(),
        msg: to_binary(&QueryMsg::ResolveOperation { ... })?,
    })
)?;
```

---

### So sánh nhanh

|                     | `query_wasm_smart`                                      | `query` with `WasmQuery::Smart`                          |
|---------------------|---------------------------------------------------------|-----------------------------------------------------------|
| Dễ viết             | ✅ Gọn hơn                                              | ❌ Phải tự wrap `QueryRequest::Wasm(...)`                 |
| Tự động to_binary   | ✅ Có                                                   | ❌ Bạn phải tự `to_binary(...)`                           |
| Linh hoạt mở rộng   | ❌ Chỉ dùng cho smart contract                          | ✅ Có thể dùng cho Bank, Staking, Custom query            |
| Giao diện           | Cao cấp, dành riêng cho Wasm                           | Thấp hơn, tổng quát hơn                                  |

---

### Khi nào nên dùng?

| Mục đích                           | Gợi ý dùng |
|-----------------------------------|------------|
| Gọi query đơn giản đến contract   | `query_wasm_smart` |
| Tự build query nâng cao, đa loại  | `query`            |
| Viết framework đa năng (như Burve)| `query`            |
| Thử nghiệm hoặc đọc code dễ hơn   | `query_wasm_smart` |
