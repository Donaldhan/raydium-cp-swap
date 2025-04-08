当然可以，以下是你提供内容的 纯 Markdown 格式版本，适合复制到文档或 README 中使用：

# 📥 Deposit 指令说明（流动性添加）

此指令允许用户向池子中存入两种 Token，以获得等价值的 LP Token。适配 SPL Token 与 Token2022，并自动处理转账费用。

---

## 🧩 账户结构说明

```rust
#[derive(Accounts)]
pub struct Deposit<'info> { ... }

💼 必要账户列表：

账户名	类型	描述
owner	Signer	用户签名者，支付和接收操作执行者
authority	UncheckedAccount	AMM 的 PDA 权限账户（构造转账和铸造授权）
pool_state	AccountLoader<PoolState>	池子全局状态
owner_lp_token	TokenAccount	用户用于接收 LP Token 的账户
token_0_account	TokenAccount	用户的 Token0 SPL Token 账户
token_1_account	TokenAccount	用户的 Token1 SPL Token 账户
token_0_vault	TokenAccount	池子中存储 Token0 的账户
token_1_vault	TokenAccount	池子中存储 Token1 的账户
token_program	Program<Token>	SPL Token v1 程序
token_program_2022	Program<Token2022>	SPL Token v2 程序（支持 Transfer Hook 等）
vault_0_mint	Mint	Token0 的 Mint
vault_1_mint	Mint	Token1 的 Mint
lp_mint	Mint	LP Token 的 Mint（输出）



⸻

🔁 指令函数逻辑：deposit

pub fn deposit(
    ctx: Context<Deposit>,
    lp_token_amount: u64,
    maximum_token_0_amount: u64,
    maximum_token_1_amount: u64
) -> Result<()>

📋 参数解释：
	•	lp_token_amount: 用户希望获得的 LP Token 数量
	•	maximum_token_0_amount: 用户可接受的最大 token_0 支付额度
	•	maximum_token_1_amount: 用户可接受的最大 token_1 支付额度（用于限制滑点）

⸻

🧠 执行步骤详解

✅ 1. 校验池子是否允许存入

if !pool_state.get_status_by_bit(PoolStatusBitIndex::Deposit) {
    return err!(ErrorCode::NotApproved);
}



⸻

📊 2. 计算所需 Token0 和 Token1 数量

使用池中总量（扣除费用）和当前 LP Supply，通过 CurveCalculator 得出添加 LP 所需的 token 数量：

let (token_0_amount, token_1_amount) = CurveCalculator::lp_tokens_to_trading_tokens(...);



⸻

💸 3. 计算手续费后用户需要实际支付的 Token 数量

通过 get_transfer_inverse_fee，反推转账后剩余为 token_amount 所需的原始转账数值：

let (transfer_token_0_amount, transfer_token_0_fee) = ...;
let (transfer_token_1_amount, transfer_token_1_fee) = ...;



⸻

🔍 4. 滑点检查（Slippage Check）

如果任何 Token 的实际支付金额超过 maximum_*_amount，报错终止：

if transfer_token_0_amount > maximum_token_0_amount 
    || transfer_token_1_amount > maximum_token_1_amount {
    return Err(ErrorCode::ExceededSlippage.into());
}



⸻

🔁 5. 实际转账 Token 到池子 Vault

通过封装函数 transfer_from_user_to_pool_vault 支持 SPL 和 Token2022 转账：

transfer_from_user_to_pool_vault(...);



⸻

🪙 6. 更新 LP 供应 & 铸造 LP Token

pool_state.lp_supply += lp_token_amount;

token_mint_to(...);

授权通过 PDA authority 完成 LP Token 的铸造。

⸻

🕒 7. 更新最近操作 Epoch

pool_state.recent_epoch = Clock::get()?.epoch;

用于奖励或状态管理的时间记录。

⸻

🧾 日志与事件

使用 emit! 发出 LpChangeEvent，记录存入前后变化情况：

emit!(LpChangeEvent { ... });



⸻

✅ 总结

此 deposit 指令完成如下任务：
	•	⛳ 验证池子状态
	•	🧮 根据 LP 数量计算所需 token 数量
	•	💸 自动计算并适配 SPL Token 手续费（支持 SPL 2022）
	•	🔁 执行双 Token 转账至 Vault
	•	🪙 铸造等值 LP Token
	•	🧾 发出日志事件、更新状态

⸻
