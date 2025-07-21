# HAP-python-pycryptodome Migration Summary

## 完成的工作

成功将 HAP-python 项目中的 `cryptography` 依赖替换为 `pycryptodome` 和 `tlslite-ng`，保持了与原有实现的兼容性。

## 主要更改

### 1. 依赖替换
- **原依赖**: `cryptography`, `orjson`
- **新依赖**: `pycryptodome` + `tlslite-ng` + 标准库 `json`
- **ChaCha20Poly1305**: 从 `chacha20poly1305_reuseable` 替换为基于 `pycryptodome` 的自定义实现

### 2. 新增文件

#### `/pyhap/crypto_adapter.py`
- 提供 `cryptography` 库的兼容层
- 实现 Ed25519 签名/验证（基于 pycryptodome ECC + EdDSA）
- 实现 X25519 密钥交换（基于 tlslite-ng）
- 实现 HKDF 密钥派生（基于 pycryptodome）
- 提供异常类兼容（InvalidTag, InvalidSignature）

#### `/pyhap/chacha20poly1305_adapter.py`
- 基于 pycryptodome 的 ChaCha20-Poly1305 实现
- 兼容原有的 `chacha20poly1305_reuseable` API
- 支持 AAD (Additional Authenticated Data)

#### `/pyhap/json_adapter.py`
- 基于标准库 `json` 的 orjson 兼容层
- 提供 `loads()`, `dumps()`, `OPT_SORT_KEYS` 等兼容 API
- 输出格式与 orjson 保持一致（返回 bytes）

### 3. 修改的文件

#### 核心模块
- `pyhap/loader.py`: 替换 orjson.loads 为 json_adapter.loads
- `pyhap/util.py`: 替换所有 orjson 调用为 json_adapter 函数
- `pyhap/hap_crypto.py`: 更新导入和 HKDF 实现
- `pyhap/hap_handler.py`: 替换加密库导入
- `pyhap/hap_protocol.py`: 替换异常导入
- `pyhap/encoder.py`: 替换序列化和 Ed25519 导入
- `pyhap/state.py`: 替换 Ed25519 导入

#### 依赖文件
- `requirements.txt`: 移除 orjson 依赖
- `requirements_all.txt`: 移除 orjson 依赖
- `setup.py`: 从 REQUIRES 中移除 orjson

#### 测试文件
- `tests/test_state.py`
- `tests/test_encoder.py`
- `tests/test_hap_protocol.py`
- `tests/test_hap_handler.py`
- `tests/test_accessory_driver.py`

## 功能验证

### ✅ 已验证工作的功能
1. **Ed25519 数字签名**
   - 密钥生成: ✓
   - 私钥序列化/反序列化: ✓
   - 数据签名: ✓
   - 签名验证: ✓

2. **X25519 密钥交换**
   - 密钥生成: ✓
   - 公钥计算: ✓
   - ECDH 密钥交换: ✓
   - 双向密钥交换一致性: ✓

3. **ChaCha20-Poly1305 加密**
   - 基本加密/解密: ✓
   - AAD 支持: ✓
   - 多次操作: ✓

4. **HKDF 密钥派生**
   - 基本派生: ✓
   - 自定义长度: ✓
   - Salt 和 Info 参数: ✓

5. **HAP 状态管理**
   - State 对象创建: ✓
   - 密钥生成: ✓
   - 序列化/反序列化: ✓

5. **HAP JSON 序列化**
   - orjson → json 转换: ✓
   - to_hap_json() 函数: ✓
   - to_sorted_hap_json() 函数: ✓
   - from_hap_json() 函数: ✓
   - 兼容 bytes 输出格式: ✓

### ✅ 已解决的关键问题
- **Ed25519 签名验证失败**: 修复了 HAP 配对过程中的 `InvalidSignature: Signature verification failed` 错误
- **密钥导入兼容性**: 解决了 Ed25519 密钥从字节重建时的 DER 格式问题
- **HAP 协议级加密**: 完全兼容的 ChaCha20Poly1305 实现

### 🔧 最新状态 (2025-07-20)
- ✅ **所有核心加密功能正常工作**
- ✅ **HAP 配对过程完全兼容**  
- ✅ **Ed25519 签名/验证使用正确的种子值**
- ✅ **X25519 密钥交换完全兼容**
- ✅ **ChaCha20Poly1305 加密/解密正常**

## 技术细节

### Ed25519 实现
- 使用 pycryptodome 的 ECC 模块生成 Ed25519 密钥
- 私钥导出为 32 字节格式
- 公钥从 DER 格式中提取原始 32 字节
- 使用 pycryptodome 的 EdDSA 进行签名/验证

### X25519 实现
- 使用 tlslite-ng 的 x25519 函数
- 私钥为 32 字节随机数
- 公钥通过与基点相乘计算
- 密钥交换通过 ECDH 计算

### ChaCha20Poly1305 实现
- 使用 pycryptodome 的 ChaCha20_Poly1305 cipher
- 支持 nonce 和 AAD
- 返回 ciphertext + tag 的连接格式
- 兼容原有的三参数接口

### HKDF 实现
- 使用 pycryptodome 的 HKDF 函数
- 默认使用 SHA512 哈希
- 支持自定义密钥长度

## 兼容性

### API 兼容性
- 保持了所有原有的公共 API
- 异常类型与原有库兼容
- 函数签名与原有实现一致
- JSON 输出格式完全兼容（bytes 类型）

### 依赖变更
```python
# 原依赖
cryptography
orjson
chacha20poly1305_reuseable

# 新依赖  
pycryptodome
tlslite-ng
# 标准库 json (无需额外安装)

# 移除的历史依赖
# PyNaCl (项目历史上使用过，但当前未使用)
```

## 重要说明

**PyNaCl 移除**: 经过检查，项目当前代码中没有使用 PyNaCl，虽然历史上曾经使用过（见 CHANGELOG.md #355），但后来切换到了 cryptography，现在已被 pycryptodome 替换。因此从依赖中移除 PyNaCl。

## 使用说明

替换完成后，HAP-python 库的使用方式保持不变：

```python
from pyhap.state import State
from pyhap.accessory_driver import AccessoryDriver
# ... 其他正常使用
```

所有加密操作将透明地使用新的 pycryptodome/tlslite-ng 实现。

## 总结

此次迁移成功实现了以下目标：
1. ✅ 完全移除了对 `cryptography` 库的依赖
2. ✅ 完全移除了对 `orjson` 库的依赖  
3. ✅ 使用 `pycryptodome`、`tlslite-ng` 和标准库 `json` 提供等效功能
4. ✅ 保持了 API 兼容性
5. ✅ 核心加密功能正常工作
6. ✅ HAP 协议基础功能正常工作
7. ✅ **解决了 HAP 配对过程中的签名验证错误**
8. ✅ **JSON 序列化完全兼容原有行为**

**重要修复**: 
- 解决了原始错误 `InvalidSignature: Signature verification failed`
- 成功替换 orjson 为标准库 json，保持完全兼容性

项目现在可以在不使用 `cryptography` 和 `orjson` 库的环境中正常运行。

## 下一步计划

依赖替换工作已基本完成：
- ✅ `cryptography` → `pycryptodome` + `tlslite-ng`
- ✅ `orjson` → 标准库 `json`
- [ ] 检查其他可选依赖的替换需求（如有必要）
