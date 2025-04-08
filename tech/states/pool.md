当然，这里是你提供内容的 Markdown 格式版本：

---

# 🛠️ PoolState 状态设置原理说明

`PoolState` 中的 `status` 字段用于通过 **位运算（bitwise operations）** 控制流动池的功能是否启用，包括 **Deposit（存款）**、**Withdraw（取款）** 和 **Swap（交易）**。

---

## 📐 状态字段定义

```rust
/// bit0: Deposit（1）
/// bit1: Withdraw（2）
/// bit2: Swap（4）
pub status: u8,

	•	status 是一个 8 位的无符号整数，每一位代表一个功能的开关状态。
	•	使用 位掩码 来表示某个功能是否启用。

⸻

📊 状态位说明

功能	位索引（Bit Index）	掩码值	状态含义
Deposit	0	1	1 表示禁用，0 启用
Withdraw	1	2	1 表示禁用，0 启用
Swap	2	4	1 表示禁用，0 启用

示例：
	•	status = 0：所有功能启用。
	•	status = 5（0b101）：禁用 Deposit 和 Swap，仅启用 Withdraw。

⸻

🧩 设置状态方法

set_status_by_bit

pub fn set_status_by_bit(&mut self, bit: PoolStatusBitIndex, flag: PoolStatusBitFlag)

	•	bit：要设置的功能位。
	•	flag：
	•	Enable：启用此功能（将该位清零）
	•	Disable：禁用此功能（将该位置 1）

✅ 示例

// 禁用 Swap 功能
pool.set_status_by_bit(PoolStatusBitIndex::Swap, PoolStatusBitFlag::Disable);

// 启用 Deposit 功能
pool.set_status_by_bit(PoolStatusBitIndex::Deposit, PoolStatusBitFlag::Enable);



⸻

🔍 获取状态方法

get_status_by_bit

pub fn get_status_by_bit(&self, bit: PoolStatusBitIndex) -> bool

	•	返回值：true 表示功能启用，false 表示功能禁用。

✅ 示例

if pool.get_status_by_bit(PoolStatusBitIndex::Withdraw) {
    // Withdraw 功能是启用的
}



⸻

🧠 原理小结
	•	使用位运算节省空间和操作效率：一个 u8 可以管理多达 8 个状态。
	•	修改某一功能状态不会影响其他功能。
	•	设置状态：
	•	启用：& !mask
	•	禁用：| mask
	•	读取状态：status & mask == 0 表示启用。

⸻
