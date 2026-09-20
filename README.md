
<p align="center">
  <h1>lyzcrypto</h1>
  <a href="https://pypi.org/project/lyzcrypto/"><img src="https://img.shields.io/pypi/v/lyzcrypto.svg" alt="PyPI version"></a>
  <a href="https://pypi.org/project/lyzcrypto/"><img src="https://img.shields.io/badge/Python-3.8~3.14-3776AB?logo=python&logoColor=white" alt="Python"></a>
  <a href="https://github.com/zhenzi0322-package/lyzcrypto/blob/master/LICENSE"><img src="https://img.shields.io/pypi/l/lyzcrypto.svg" alt="License"></a>
          <a href="https://tool.long920.cn/lyzcrypto"><img src="https://app.readthedocs.org/projects/zhenzi0322-tool/badge/?version=latest" alt="Documentation Status"></a>
</p>

> 纯标准库实现的 `.lyz` 格式加解密库（`AES-256-CBC` + `PKCS7` + 可选 `HMAC-SHA256`）。


## 特性

- **零第三方依赖** — 仅使用 `Python` 标准库（`os`、`hmac`、`hashlib`、`struct`、`json`）
- **AES 纯 Python 实现** — 按 `FIPS-197` 标准，查表法（`T-table`）优化
- **Windows 自动加速** — 通过 `ctypes` 调用 `bcrypt.dll`，提速两个数量级（失败自动回退纯 Python）
- **完整格式兼容** — 与旧实现生成的 `.lyz` 文件 100% 兼容

## 安装

```bash
pip install lyzcrypto
```

## 快速开始

### 字符串加解密

```python
from lyzcrypto import LYZStringCrypto

# 初始化（密钥必须是 16/24/32 字节）
crypto = LYZStringCrypto(key=b'your-32-byte-key-here-xxxxxxxx')

# 加密
encrypted = crypto.encrypt("Hello, 世界!")
print(type(encrypted))  # <class 'bytes'>

# 解密
decrypted = crypto.decrypt(encrypted)
print(decrypted)  # Hello, 世界!
```

### 字典加解密

```python
from lyzcrypto import LYZStringCrypto

crypto = LYZStringCrypto(key=b'your-32-byte-key-here-xxxxxxxx')

# 加密字典 → 字节流
data = {"name": "张三", "age": 25, "tags": ["python", "crypto"]}
encrypted = crypto.encrypt_dict(data)

# 解密字节流 → 字典
decrypted = crypto.decrypt_to_dict(encrypted)
print(decrypted)  # {'name': '张三', 'age': 25, 'tags': ['python', 'crypto']}
```

### 文件加解密

```python
from lyzcrypto import LYZStringCrypto

crypto = LYZStringCrypto(key=b'your-32-byte-key-here-xxxxxxxx')

# 加密字典并写入文件
data = {"secret": "value"}
crypto.encrypt_dict_to_file(data, "secret.lyz")

# 从文件读取并解密
result = crypto.decrypt_file_to_dict("secret.lyz")
```

### 兼容明文 JSON 与加密格式

```python
# load_json 自动识别明文 JSON 或 .lyz 加密格式
result = crypto.load_json(raw_bytes)
```

## 文件格式

```
| MAGIC(4B) | VERSION(1B) | IV(16B) | HMAC(32B, 可选) | ENCRYPTED_DATA |
```

- `MAGIC`: 固定为 `LYZ1`
- `VERSION`: 当前版本号 `1`
- `IV`: 每次加密随机生成的 16 字节初始化向量
- `HMAC`: SHA-256 校验（`use_hmac=False` 时省略）
- `ENCRYPTED_DATA`: AES-256-CBC 加密后的数据（PKCS7 填充）

## API 参考

### `LYZStringCrypto(key, use_hmac=True)`

| 参数 | 类型 | 说明 |
|------|------|------|
| `key` | `bytes` | 加密密钥，长度必须为 16/24/32 字节 |
| `use_hmac` | `bool` | 是否启用 HMAC-SHA256 校验（默认 `True`） |

### 方法

| 方法 | 说明 |
|------|------|
| `encrypt(data)` | 加密字符串或字节，返回 `.lyz` 格式字节流 |
| `decrypt(lyz_data)` | 解密 `.lyz` 格式数据，返回原始字符串 |
| `encrypt_dict(data_dict)` | 加密字典，返回 `.lyz` 格式字节流 |
| `decrypt_to_dict(lyz_data)` | 解密 `.lyz` 格式数据，返回字典 |
| `encrypt_dict_to_file(data_dict, file_path)` | 加密字典并写入文件 |
| `decrypt_file_to_dict(file_path)` | 从文件读取并解密为字典 |
| `load_json(raw)` | 自动识别明文 JSON 或 `.lyz` 加密格式并解析 |

