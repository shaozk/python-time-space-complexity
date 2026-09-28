---
source_sha: c3c73ab839a5b77b3fca60844a3f5248278dab5ecf214d168681092967a90985
translated: machine
---

# bytearray() 函数的复杂度

`bytearray()` 函数构造一个可变的字节序列：一块可扩缩的缓冲区，就地编辑。它拥有 `bytes`
的查找、拆分与转换方法——这些方法都不触碰缓冲区：拆分和转换从它构建新对象，查找和判断只读取
它——以及 `list` 的变更方法：`append()`、`extend()`、`insert()`、`pop()`、`remove()`、
`clear()`、`reverse()` 和切片赋值。

缓冲区在增长时会过量分配，因此追加是均摊 O(1)；它还记录内容从哪里开始，因此删除前缀只需移动
该偏移而不必移动尾部——无论是用 `del` 还是在头部赋一个更短的切片，后者仍要为复制进来的字节付
出代价。在其他任何位置改变长度都会使其后的所有内容移动。增长超出缓冲区时会把内容复制进更大的
缓冲区；降到不足其一半时同样如此，会把内容压缩进更小的缓冲区。这是改变长度时仅有的两处分配。

`n` 为 bytearray 中的字节数，`k` 为另一操作数中的字节数（复制进来的数据、覆盖的切片或可迭代
对象的元素），`m` 为模式（`sub`、`prefix`、`suffix`、`sep`）中的字节数，`p` 为交给
`join()` 的部分数，`w` 为结果中与 `n` 不同时的字节数。可迭代对象的元素是 `range(256)` 中的
一个 `int`，读取为 O(1)；编码相关行假设编解码器和错误处理器的成本与输出都与其输入成比例，面向
字节的编解码器正是如此。结构化编解码器的成本可能更高。

## 复杂度参考

### 构造

| 操作 | 时间 | 空间 | 备注 |
|-----------|------|-------|-------|
| `bytearray()` | O(1) | O(1) | 空，未分配缓冲区 |
| `bytearray(count)` | O(n) | O(n) | n = count 个零字节；负数会抛出 `ValueError` |
| `bytearray(bytes_like)` | O(k) | O(k) | 复制 `bytes`、`bytearray`、`memoryview` 或 `array.array` 的缓冲区；副本与其来源相互独立 |
| `bytearray(iterable)` | O(k) | O(k) | 每个元素一个字节，边遍历边增长；元素超出 `range(256)` 会抛出 `ValueError` |
| `bytearray(string, encoding, errors='strict')` | O(k) | O(k) | 先编码为 `bytes` 再复制进来；k = 字符串中的字符数，无论编码器输出什么都要遍历这些字符。`str` 不带编码器会抛出 `TypeError` |
| `bytearray.fromhex(string)` | O(k) | O(k) | k = 字符数，每对字符一个字节；对与对之间的空白会被跳过。类方法 |

### 序列操作

| 操作 | 时间 | 空间 | 备注 |
|-----------|------|-------|-------|
| `len(ba)` | O(1) | O(1) | |
| `ba[i]`, `ba[i] = v` | O(1) | O(1) | 两个方向的值都是 `range(256)` 中的 `int` |
| `ba[a:b]`, `ba[a:b:step]` | O(k) | O(k) | 覆盖的 k 个字节组成的新 `bytearray`；`memoryview(ba)` 是 O(1) 的替代 |
| `ba[a:b] = data` | O(n + k) | O(k)，缓冲区增长或压缩时 O(n) | 切片长度不变时 O(k)；长度变化会移动尾部，但在头部缩短切片时例外——此时移动的是起始偏移。除非 `data` 是另一个 `bytearray`，否则会先复制到临时对象 |
| `del ba[i]`, `del ba[a:b]` | O(n) | O(1)，压缩时 O(n) | 移动尾部；删除前缀则是推进起始偏移，均摊 O(1) |
| `x in ba` | O(n + m) | O(1) | `x` 是 `int` 或类字节对象 |
| `ba + other` | O(n + k) | O(n + k) | 一个新的 `bytearray` |
| `ba += other` | O(k) 均摊 | O(k) 均摊 | 原地进行；`other` 可以是任何类字节对象 |
| `ba * count`, `ba *= count` | O(n·count)，count ≤ 0 时 O(n) | O(n·count) | 两种情况结果长度都如此；`*=` 原地增长，count 为 0 或负数时清空缓冲区，像 `clear()` 一样释放它 |
| `ba == other`, `ba != other` | O(n) | O(1) | 长度不同时 O(1)；`<` 及其他比较走到第一个差异处 |
| `for x in ba` | O(n) | O(1) | 每个字节一个 `int`；单独的 `iter(ba)` 是 O(1) |
| `hash(ba)` | O(1) | O(1) | 抛出 `TypeError`；bytearray 不可哈希 |
| `bytes(ba)` | O(n) | O(n) | 一个不可变副本 |
| `memoryview(ba)` | O(1) | O(1) | 共享缓冲区并固定其长度：视图释放前任何扩缩都会抛出 `BufferError` |

### 变更

| 操作 | 时间 | 空间 | 备注 |
|-----------|------|-------|-------|
| `append(x)` | O(1) 均摊 | O(1) 均摊 | 缓冲区增长时按约八分之一过量分配，触发增长的那次追加分配整个新缓冲区 |
| `extend(iterable)` | O(k) 均摊 | O(k) 均摊 | 类字节操作数一步复制进来；其他可迭代对象先收集到一个临时 `bytearray` |
| `insert(i, x)` | O(n) | O(1) 均摊 | 移动尾部 |
| `pop()`, `pop(-1)` | O(1) 均摊 | O(1)，压缩时 O(n) | 内容降到不足缓冲区一半时压缩一次，复制余下的部分 |
| `pop(i)` | O(n) | O(1)，压缩时 O(n) | 移动尾部，i = 0 也一样；`del ba[0]` 则不会 |
| `remove(x)` | O(n) | O(1)，压缩时 O(n) | 找到第一个匹配并移动尾部；不存在时 `ValueError` |
| `clear()` | O(n) | O(1) | 释放缓冲区 |
| `copy()` | O(n) | O(n) | |
| `reverse()` | O(n) | O(1) | 原地 |
| `resize(size)` | O(n + size) | O(size) | Python 3.14+。新增的字节为零，丢弃的字节被释放，因此缩容要为让出的缓冲区付出代价；负数会抛出 `ValueError` |

### 查找

| 操作 | 时间 | 空间 | 备注 |
|-----------|------|-------|-------|
| `find(sub, start, end)`, `index(sub, start, end)` | O(n + m) | O(1) | `sub` 是类字节对象或 `int`；不存在时 `index()` 抛出 `ValueError` |
| `rfind(sub, start, end)`, `rindex(sub, start, end)` | O(n·m) | O(1) | 最坏情况，包括模式不存在时：反向查找没有线性时间的回退，正向查找则有 |
| `count(sub, start, end)` | O(n + m) | O(1) | 不重叠的匹配 |
| `startswith(prefix, start, end)`, `endswith(suffix, start, end)` | O(m + q) | O(1) | 与 n 无关。两者都接受 q 个候选组成的元组，此时 m 为它们的总长度 |

### 转换

这里的每个方法都构建新对象，无论是否有内容改变；没有一个会触碰接收者本身。

| 操作 | 时间 | 空间 | 备注 |
|-----------|------|-------|-------|
| `replace(old, new, count=-1)` | O(n + w) | O(w) | w = 结果长度 |
| `translate(table, /, delete=b'')` | O(n + d) | O(n) | `table` 是来自 `maketrans()` 的 256 字节表，或 `None`；d = `delete` 中的字节数，每个都在一遍扫描前标进一张表 |
| `bytearray.maketrans(from, to)` | O(k) | O(1) | 无论 k 是多少都返回 256 字节的 `bytes`；两个参数必须等长。静态方法 |
| `upper()`, `lower()`, `capitalize()`, `title()`, `swapcase()` | O(n) | O(n) | 仅限 ASCII 字母 |
| `strip(chars)`, `lstrip(chars)`, `rstrip(chars)` | O(n + (s + 1)·c) | O(n) | s = 剥离的字节数，c = `chars` 中的字节数，每个被剥离的字节搜索一次，停下前再搜一次。即使无所剥离也复制 |
| `removeprefix(prefix)`, `removesuffix(suffix)` | O(n) | O(n) | 前缀或后缀不存在时也复制 |
| `center(width, fillbyte)`, `ljust(width, fillbyte)`, `rjust(width, fillbyte)`, `zfill(width)` | O(w) | O(w) | w = max(n, width)：宽度低于 n 时仍复制 n 个字节 |
| `expandtabs(tabsize=8)` | O(n + w) | O(w) | 读每个字节并写出 w；`tabsize` 低于 1 会丢弃制表符，因此 w 可能小于 n |

### 拆分与连接

| 操作 | 时间 | 空间 | 备注 |
|-----------|------|-------|-------|
| `split(sep=None, maxsplit=-1)` | O(n + m) | O(n) | 一个由新 bytearray 组成的列表，总共至多持有 n 个字节；分隔符和跳过的空白被丢弃 |
| `splitlines(keepends=False)` | O(n) | O(n) | |
| `partition(sep)` | O(n + m) | O(n) | 三个新 bytearray 复制全部 n 个字节，无论命中与否 |
| `rsplit(sep=None, maxsplit=-1)`, `rpartition(sep)` | O(n·m) | O(n) | 最坏情况：多字节 `sep` 向后搜索，每次匹配都可能是 `rfind()` 的最坏情况。`None` 和单字节 `sep` 则是扫描 |
| `join(iterable)` | O(p + w) | O(p + w) | 每个部分都必须是类字节对象，且无论长短都要付出一个缓冲区描述符的代价，因此 p 个部分有它们自己的一项成本 |

### 判断与转换

| 操作 | 时间 | 空间 | 备注 |
|-----------|------|-------|-------|
| `isalnum()`, `isalpha()`, `isascii()`, `isdigit()`, `islower()`, `isspace()`, `istitle()`, `isupper()` | O(n) | O(1) | 在第一个能决定结果的字节处停止 |
| `decode(encoding='utf-8', errors='strict')` | O(n) | O(n) | |
| `hex(sep, bytes_per_sep)` | O(n) | O(n) | 每字节两个字符 |

## 增长 bytearray

小幅增长按约八分之一过量分配缓冲区，大幅增长则精确取用所求的大小，因此一连串 `append()` 或
`+=` 调用无论怎样拆分，总成本都随新增字节数线性增长。拼接 `bytes` 则每次都要复制已累积的
全部内容。

```python
import sys

buffer = bytearray()
sizes = set()
for value in range(1000):
    buffer.append(value % 256)  # O(1) amortized
    sizes.add(sys.getsizeof(buffer))

assert len(buffer) == 1000
assert len(sizes) < 50  # the buffer was reallocated a few dozen times, not 1000

buffer += b"tail"  # O(k) amortized - copied onto the end in place
buffer.extend([1, 2, 3])  # O(k) - gathered into a temporary, then copied in
assert buffer[-7:] == bytearray(b"tail\x01\x02\x03")
```

## 从头部删除

bytearray 记录其内容在缓冲区内的起点。用 `del ba[:k]` 删除前缀移动的是起点而非字节，且只有
当余下内容降到不足缓冲区一半时才压缩，因此从头部排空一个缓冲区的总成本是 O(n)。`pop(0)` 和
`insert(0, x)` 则要移动余下的每个字节，各为 O(n)。

```python
buffer = bytearray(b"header:payload:trailer")

del buffer[:7]  # O(1) amortized - the start offset advances
assert buffer == bytearray(b"payload:trailer")

first = buffer.pop(0)  # O(n) - every remaining byte shifts left
assert first == ord("p")

buffer.insert(0, ord("P"))  # O(n) - every byte shifts right
assert buffer.startswith(b"Payload")  # O(m)

del buffer[-8:]  # O(1) amortized - nothing after it to shift
assert buffer == bytearray(b"Payload")
```

## 切片会复制，视图不会

切片是一个持有其所覆盖字节的新 `bytearray`。`memoryview` 共享缓冲区，且只要它存活，
bytearray 就不能改变长度。

```python
data = bytearray(b"abcdefgh")

piece = data[2:5]  # O(k) - a new bytearray of the 3 bytes covered
piece[0] = ord("X")
assert data == bytearray(b"abcdefgh")  # the original is untouched

view = memoryview(data)  # O(1) - shares the buffer
view[0] = ord("A")
assert data[:1] == bytearray(b"A")

try:
    data.append(0)  # the view pins the length
except BufferError as error:
    assert "cannot be re-sized" in str(error)
else:
    raise AssertionError("a bytearray was resized under a live memoryview")

view.release()
data.append(0)  # O(1) amortized again
assert len(data) == 9
```

### 切片赋值

用相同字节数替换切片会把它们复制到位。替换成更多或更少则移动切片之后的所有内容，除非更短的
那个从头部开始：此时移动的是起始偏移，与 `del` 一样。

```python
data = bytearray(b"0123456789")

data[2:4] = b"AB"  # O(k) - same length, copied into place
assert data == bytearray(b"01AB456789")

data[2:4] = b"abcd"  # O(n + k) - longer, the tail shifts right
assert data == bytearray(b"01abcd456789")

data[2:6] = b""  # O(n) - shorter, the tail shifts left
assert data == bytearray(b"01456789")

data[:2] = b""  # O(1) - shorter at the front, so the start offset moves
assert data == bytearray(b"456789")
```

## 查找

`find()` 即使在最坏情况下也是线性的。`rfind()` 用更简单的算法向后搜索，在每个位置都可能
比较模式的大部分，因此在缓冲区的每个位置都可能付出模式长度的代价。`rsplit()` 和
`rpartition()` 以同样的方式搜索。

```python
haystack = bytearray(b"a" * 10_000)
needle = b"ab" + b"a" * 98  # mismatches only at its second byte

assert haystack.find(needle) == -1  # O(n + m) - linear in the worst case
assert haystack.rfind(needle) == -1  # O(n·m) - m comparisons at each of n positions
assert haystack.count(b"aa") == 5_000  # O(n + m) - non-overlapping
assert haystack.startswith(b"aaa")  # O(m) - the prefix's length, not the buffer's
assert 97 in haystack  # O(n) - an int is a byte value

try:
    haystack.index(b"b")  # O(n + m)
except ValueError as error:
    assert "not found" in str(error)
else:
    raise AssertionError("index() found a byte that is not there")
```

## 转换方法返回新对象

`bytes` 方法族中没有谁会编辑缓冲区，且每个转换方法都为它的结果分配一个新的 `bytearray`，即使
该结果与输入完全相同。因此用 `strip()` 或 `replace()` 检查一个缓冲区要付出一次复制，而
`startswith()`、`find()` 或判断方法则不用。

```python
data = bytearray(b"  Hello  ")

assert data.strip() == bytearray(b"Hello")  # O(n) - a new bytearray
assert data.strip(b"x") is not data  # O(n) - a copy even when nothing is stripped
assert data.replace(b"z", b"y") is not data  # O(n + w) - likewise
assert data.upper() == bytearray(b"  HELLO  ")  # O(n)
assert data.isspace() is False  # O(n) - stops at the first byte that settles it

table = bytearray.maketrans(b"lo", b"01")  # O(k) - always a 256-byte table
assert len(table) == 256
assert data.translate(table, delete=b" ") == bytearray(b"He001")  # O(n)
```

## 常见模式

### 消费流式缓冲区

```python
buffer = bytearray()
chunks = [b"GET / HTTP/1.1\r\nHost: ex", b"ample\r\n\r\nbody"]
lines = []

for chunk in chunks:
    buffer += chunk  # O(k) amortized
    while (end := buffer.find(b"\r\n")) != -1:  # O(n + m)
        lines.append(bytes(buffer[:end]))  # O(k)
        del buffer[: end + 2]  # O(1) amortized - the start offset advances

assert lines == [b"GET / HTTP/1.1", b"Host: example", b""]
assert buffer == bytearray(b"body")
```

### 构造消息

```python
payload = b"payload"

message = bytearray()
message.append(1)  # O(1) amortized - a message type
message += len(payload).to_bytes(2, "big")  # O(k) amortized
message += payload  # O(k) amortized
message.append(0)  # O(1) amortized

assert bytes(message) == b"\x01\x00\x07payload\x00"  # O(n) - freeze it once, at the end
```

## 性能最佳实践

✅ **应该**：

- 用 `append()`、`+=` 或 `extend()` 构建，最后调用一次 `bytes()`；总成本保持线性，而 `bytes + bytes` 每次都要复制已积累的全部内容
- 用 `del buffer[:k]` 丢弃已消费的前缀，它是均摊 O(1)；循环里的 `pop(0)` 每次都移动整个缓冲区
- 不需要复制时通过 `memoryview(ba)` 读取区域；切片要复制它的 k 个字节
- 在你无法控制的输入上，优先用正向的 `find()`、`index()`、`split()` 和 `partition()` 而非它们的 `r` 版本；只有正向查找在最坏情况下是线性的

❌ **避免**：

- 在循环中使用 `insert(0, x)` 和 `pop(0)`——每次都移动编辑点之后的每个字节
- 用 `strip()`、`replace()` 或 `upper()` 检查缓冲区——即使没有变化每个也要复制全部 n 个字节；`startswith()`、`find()` 和判断方法不会
- 在缓冲区的 `memoryview` 存活期间扩缩——这会抛出 `BufferError` 而非复制
- 把 bytearray 用作字典键——它不可哈希，且 `bytes(ba)` 是一次 O(n) 复制

## 版本说明

- **Python 3.14+**：新增 `resize()`；`fromhex()` 除 `str` 外也接受类字节输入
- **所有 Python 3**：bytearray 不可哈希；它的存活 `memoryview` 会用 `BufferError` 阻止每一次长度改变

## 相关函数

- **[bytes()](bytes_func.md)** - 不可变的对应物，以相同的方式构造
- **[Bytes 与 ByteArray 操作](bytes.md)** - 两个类型的方法并排对比
- **[memoryview()](memoryview_func.md)** - bytearray 缓冲区上的 O(1) 视图
- **[str](str.md)** - `encode()` 产出 `bytearray(text, encoding)` 复制进来的字节
- **[list](list.md)** - 同一套变更 API 作用于任意对象，但没有均摊 O(1) 的头部删除
