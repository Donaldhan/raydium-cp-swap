当然可以，以下是你提供内容的标准 Markdown 格式版本，可以直接复制进 README 或文档：

# 📄 Token Utils 模块分析

本模块封装了基于 `spl_token_2022` 的常见 SPL Token 操作，包括转账、铸造、销毁、费用计算和账户创建等。

---

## 📌 常量

```rust
const MINT_WHITELIST: [&'static str; 4] = [ ... ];

用途：

定义允许使用的 SPL Token Mint 地址白名单（包含 SPL2022 token 时的额外校验辅助）。

⸻

🔁 Token 转账函数

transfer_from_user_to_pool_vault

pub fn transfer_from_user_to_pool_vault(...)

	•	用户 → 池子 Vault 的转账
	•	使用 token_2022::transfer_checked
	•	支持 Token2022 的扩展（如转账费用）

⸻

transfer_from_pool_vault_to_user

pub fn transfer_from_pool_vault_to_user(...)

	•	池子 Vault → 用户 的转账
	•	同样支持签名者种子 signer_seeds 用于 PDA 授权

⸻

🪙 Token 操作函数

token_mint_to

pub fn token_mint_to(...)

	•	Token 铸造，指定目标账户和铸造数量
	•	支持 PDA 授权

⸻

token_burn

pub fn token_burn(...)

	•	Token 销毁，从账户中扣除指定数量
	•	使用 spl_token_2022::burn

⸻

💸 手续费相关函数

get_transfer_inverse_fee

pub fn get_transfer_inverse_fee(mint_info: &AccountInfo, post_fee_amount: u64) -> Result<u64>

	•	给定 “手续费后” 数额，反推“原始”应支付金额
	•	用于用户需要精确收到某个数额的场景

⸻

get_transfer_fee

pub fn get_transfer_fee(mint_info: &AccountInfo, pre_fee_amount: u64) -> Result<u64>

	•	给定“原始转账数额”，计算手续费
	•	内部根据 TransferFeeConfig 自动判断

⸻

✅ Mint 支持性校验

is_supported_mint

pub fn is_supported_mint(mint_account: &InterfaceAccount<Mint>) -> Result<bool>

判断一个 Mint 是否受支持：
	•	是原生 Token (Token::id())
	•	或在白名单中
	•	或仅包含特定扩展：
	•	TransferFeeConfig
	•	MetadataPointer
	•	TokenMetadata

⸻

🧱 Token 账户创建

create_token_account

pub fn create_token_account(...)

支持创建 SPL Token 或 SPL Token 2022 类型账户：
	•	根据 Mint 类型判断所需空间
	•	调用 create_or_allocate_account 实现：
	•	lamports 充值
	•	分配空间
	•	分配 owner
	•	然后使用 initialize_account3 初始化账户

⸻

🏗️ create_or_allocate_account

pub fn create_or_allocate_account(...)

核心账户分配逻辑：
	1.	若账户 lamports 为 0，则为新账户 → system_program::create_account
	2.	若账户已存在但空间不足或 rent 不足：
	•	先 transfer lamports
	•	再 allocate 空间
	•	最后 assign 给目标 program

⸻

🧠 总结

此模块聚焦于构建通用 Token 操作封装，适配 SPL Token v1 和 v2（Token2022），特别支持包含 TransferFee 扩展的代币，适用于：
	•	流动性池管理
	•	Token 跨账户操作
	•	铸造销毁逻辑抽象
	•	Token 扩展兼容性校验

⸻



如果你还需要将其转换为 HTML、PDF 或集成到文档系统（如 GitBook、Docusaurus、VuePress）里，我也可以帮你搞定～你想导出哪种？