# Protobuf 标量类型在各语言中的映射

| Proto 类型 | C++ 类型 | Java/Kotlin 类型[^1] | Python 类型[^3] | Go 类型 | Ruby 类型 | C# 类型 | PHP 类型 | Dart 类型 | Rust 类型 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| double | double | double | float | float64 | Float | double | float | double | f64 |
| float | float | float | float | float32 | Float | float | float | double | f32 |
| int32 | int32_t | int | int | int32 | Fixnum 或 Bignum（根据需要） | int | integer | int | i32 |
| int64 | int64_t | long | int/long[^4] | int64 | Bignum | long | integer/string[^6] | Int64 | i64 |
| uint32 | uint32_t | int[^2] | int/long[^4] | uint32 | Fixnum 或 Bignum（根据需要） | uint | integer | int | u32 |
| uint64 | uint64_t | long[^2] | int/long[^4] | uint64 | Bignum | ulong | integer/string[^6] | Int64 | u64 |
| sint32 | int32_t | int | int | int32 | Fixnum 或 Bignum（根据需要） | int | integer | int | i32 |
| sint64 | int64_t | long | int/long[^4] | int64 | Bignum | long | integer/string[^6] | Int64 | i64 |
| fixed32 | uint32_t | int[^2] | int/long[^4] | uint32 | Fixnum 或 Bignum（根据需要） | uint | integer | int | u32 |
| fixed64 | uint64_t | long[^2] | int/long[^4] | uint64 | Bignum | ulong | integer/string[^6] | Int64 | u64 |
| sfixed32 | int32_t | int | int | int32 | Fixnum 或 Bignum（根据需要） | int | integer | int | i32 |
| sfixed64 | int64_t | long | int/long[^4] | int64 | Bignum | long | integer/string[^6] | Int64 | i64 |
| bool | bool | boolean | bool | bool | TrueClass/FalseClass | bool | boolean | bool | bool |
| string | std::string | String | str/unicode[^5] | string | String (UTF-8) | string | string | String | ProtoString |
| bytes | std::string | ByteString | str（Python 2）、bytes（Python 3） | []byte | String (ASCII-8BIT) | ByteString | string | List | ProtoBytes |

## 脚注

[^1]: Kotlin 使用与 Java 对应的类型，无符号类型也是如此，以保证 Java/Kotlin 混合代码库的兼容性。

[^2]: 在 Java 中，无符号 32 位和 64 位整数用对应的有符号类型表示，最高位直接存放在符号位上。

[^3]: 在所有语言中，为字段赋值时都会做类型检查，以确保取值合法。

[^4]: 64 位或无符号 32 位整数解码时总是表示为 `long`；但如果在设置字段时传入的是 `int`，也可以是 `int`。任何情况下，取值都必须落在所设置类型的范围内。参见 [^2]。

[^5]: Python 字符串解码后表示为 `unicode`，但如果传入的是 ASCII 字符串，也可以是 `str`（此行为可能变更）。

[^6]: 64 位机器上使用 `integer`，32 位机器上使用 `string`。

## 备注

- `int32`、`int64`、`uint32`、`uint64`、`sint32`、`sint64` 使用可变长度编码（varint）；其中 `int32`/`int64` 编码负数效率低，字段可能出现负值时建议改用 `sint32`/`sint64`。
- `sint32`/`sint64` 为有符号整数，对负数的编码效率高于普通的 `int32`/`int64`。
- `fixed32` 恒为 4 字节，当取值经常大于 2²⁸ 时比 `uint32` 更高效；`fixed64` 恒为 8 字节，当取值经常大于 2⁵⁶ 时比 `uint64` 更高效。
- `double` 使用 IEEE 754 双精度格式，`float` 使用 IEEE 754 单精度格式。
- `string` 必须始终是 UTF-8 编码或 7 位 ASCII 文本，且长度不得超过 2³²。
- `bytes` 可包含任意字节序列，长度不得超过 2³²。

> 来源：<https://protobuf.dev/programming-guides/proto3/#scalar>
