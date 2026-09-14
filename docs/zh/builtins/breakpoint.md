---
source_sha: 4593911d19ceb356fd1cf4d2a5339c188f9731dd52fa952b317d38c67cd703b5
translated: machine
---

# breakpoint() 函数

`breakpoint()` 函数会进入 Python 调试器（默认为 pdb），用于交互式调试。

## 复杂度参考

| 操作 | 时间 | 空间 | 备注 |
|------|------|-------|-------|
| `breakpoint()` | O(1) | O(1) | 暂停执行 |
| 调试器会话 | O(n) | O(n) | n = 调试所花时间 |


## 相关模块

- [pdb 模块](../stdlib/pdb.md) - Python 调试器
- [sys 模块](../stdlib/sys.md) - 系统函数（breakpointhook）
- [logging 模块](../stdlib/logging.md) - 生产环境调试
