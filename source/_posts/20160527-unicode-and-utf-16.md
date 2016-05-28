title: Unicode and UTF-16
date: 2016-05-27 16:19:30
tags:
---

## Unicode

[Unicode](http://unicode.org) 做为一个国际统一的标准，目的是给世界上的每个字符都分配一个唯一的数字。这样一来，只要支持 Unicode 的系统，每一个数字都唯一的表示一个字符。例如：`0x0041` 表示大写字母 `A`，0x4E25 表示汉字 `严`。

`U+xxxx` 是 Unicode 的表示形式，xxxx 是数值的十六进制格式。`U+0041` `U+4E25`

每个表示 Unicode 字符的数值都成为`码点`(code point)。最初 Unicode 用 2 个字节来表示一个码点，提供了 65,535 个码点，也就是说可以容纳 65,535 个字符。后来考虑到要容纳中日韩以及历史上的一些字符，Unicode 扩充到了 21 位。从 0x0000 到 0x10FFFF，提供了 1,114,112 个码点。目前大概使用了 10% 的编码空间。

<!--more-->

Unicode 将 1,114,112 个编码空间分为17个`平面`(plane)。第 0 号平面叫做`基本多文本平面`(Basic Multilinguar Plane, BMP)，而其它 16 个平面称为`辅助平面`(Supplementary Planes)

Plane | Code Point Range
--- | ---
0 | 0x00 0000 - 0x00 FFFF
1 | 0x01 0000 - 0x01 FFFF
2 | 0x02 0000 - 0x02 FFFF
... | ... - ...
15 | 0x0F 0000 - 0x0F FFFF
16 | 0x10 0000 - 0x10 FFFF

## UTF-16

Unicode 的码点使用了 4 个字节来表示一个字符。如果文件存储的时候直接使用码点的话，那对于只有一个字节的 ASCII 码字符来说，是非常浪费空间的。同样对于网络传输来说也是一样。

解决方案就是 `Unicode Transformation Format -- UTF`。UTF 是一种编码方案，或者也可以想象成一种 Mapping 算法。它将 Unicode 的码点用另外一种 byte 序列来表示，以方便存储和传输。每个 Unicode 字符对应的 UTF 编码称为 `CodeUnit`

UTF 编码可以分为 UTF-32，UTF-16 和 UTF-8 三种。因为了解到 [Swift](https://swift.org) 的 String 类型是以 UTF-16 编码的，所以特别补一下 UTF-16 的编码知识。

UTF-16 是一种变长的编码方案。即每个 CodePoint 编码之后的 CodeUnit 是 `2` 个或 `4` 个字节。0 号基本面用 2 个字节编码。其它基本面用 4 个字节编码。

0 号基本面的码点空间是从 `0x00,0000 ~ 0x00,FFFF`，它的两个高字节都是零。所以 0 号基本面的 CodePoint 直接就是 2 字节的 UTF-16 编码。既 CodePoint 和 UTF-16 编码是一模一样，一一对应(不完全，中间有一段特殊区域，下文详解)。

那么其它16个码点平面的码点要怎么编码呢？刚说到 2 字节的 UTF-16 编码空间里面，中间有一段区域是有特殊作用的。`0xD800 ~ 0xDFFF` 的码点是用来表示一种叫做`代理对`(surrogate pair)的东西。每个代理对由4个字节组成。其中前面的两个字节称为 `lead` 后面两个字节称为 `trail`

代理对的编码规则是：

* CodePoint U+xx,xxxx 减掉 0x1,0000。例如：0x1,0437 - 0x1,0000 = 0x0437
* 0x0,0437 二进制是 0000 0000 0100 0011 0111，总共 20 bits，分为前 10 bits 和 后 10 bits
* 前 10 个 bits 加上 0xD800 形成 lead，0xD800 + 0x0001 = 0xD801
* 后 10 个 bits 加上 0xDC00 形成 trail，0xDC00 + 0x0037 = 0xDC37
* 最终的代理对就是 `0xD801DC37`

解码就是上述的逆操作。解码器的逻辑大概就是一次读 2 个字节，如果不是 D8 或 DC 开头，这两个字节就是一个基本面的 CodePoint。反之，这是一个代理对，接着读出下两个字节，形成 4 字节的代理对，安装上面算法的逆运算解码出 CodePoint。

可以参考 Golang 的标准库里面 UTF-16 代理对的编码解码过程来

```go
// EncodeRune returns the UTF-16 surrogate pair r1, r2 for the given rune.
// If the rune is not a valid Unicode code point or does not need encoding,
// EncodeRune returns U+FFFD, U+FFFD.
func EncodeRune(r rune) (r1, r2 rune) {
	if r < 0x10000 || r > maxRune {
		return replacementChar, replacementChar
	}
	r -= 0x10000
	return 0xd800 + (r>>10)&0x3ff, 0xdc00 + r&0x3ff
}
```

`rune` 是一个 32 bits 的 CodePoint。If 语句先检查 CodePoint 有没有越界。`0xd800 + (r>>10)&0x3ff` 右移 10 位加上 0xD800 形成 lead。`0xdc00 + r&0x3ff` 低 10 位直接加上 0xDC00 形成 trail。

```go
// DecodeRune returns the UTF-16 decoding of a surrogate pair.
// If the pair is not a valid UTF-16 surrogate pair, DecodeRune returns
// the Unicode replacement code point U+FFFD.
func DecodeRune(r1, r2 rune) rune {
	if 0xd800 <= r1 && r1 < 0xdc00 && 0xdc00 <= r2 && r2 < 0xe000 {
		return (r1-0xd800)<<10 | (r2 - 0xdc00) + 0x10000
	}
	return replacementChar
}
```

解码的过程也很简单。同样先检查是否是正确的代理对，lead 减掉 0xD800 之后左移 10 位。trail 直接减掉 0xDC00 再和 lead 的解码结果做 `|` 相当于拼接在一起形成 20 bits 的长度。最后再加上 0x10000 就得到最后的 CodePoint。

编码算法里面要先减掉 0x1,000 然后解码的时候再加回来的原因是：4 个字节的代理对只能容纳 20 bits，提供 FFFFF 个码点。然而最后一个辅助平面 0x10,0000 ~ 0x10,FFFF 已经超过了 FFFFF。同时，基本面是不会编码成代理对的。所以减掉 0x10000 相当于把每个编码面减掉 0x10000。这样第 16 号编码面就小于 FFFFF 了。可以假象一个编码面位为一个 bit，这就类似于位的右移操作。

下面用一个例子来一步步演示一下编码过程，我们用 0x02,14FE 来作为例子

```txt
减掉 0x1,0000 
0x02,14FE - 0x01,0000 = 0x00,14FE

0x0,14FE 的二进制形式
0000 0001 0100 1111 1110
以 10 bits 分隔后就是：(00 0000 0101) (00 1111 1110)

0xD800 的二进制 1101 1000 0000 0000 加上高位的 10 bits 00 0000 0101
结果是 1101 1000 0000 0101 = 0xD805

0xDC00 的二进制 1101 1100 0000 0000 加上低位的 10 bits 00 1111 1110
结果是 1101 1100 1111 1110 = 0xDCFE

编码后的代理对就是：0xD805DCFE
```

如果要编码的是最后一个编码面的字符 `0x10,1234`，减掉 0x01,0000 之后是 0x0F,1234，小于 FFFFF。仍然适用上述例子的算法进行编码。


