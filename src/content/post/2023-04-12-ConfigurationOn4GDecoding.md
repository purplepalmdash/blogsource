+++
title= "ConfigurationOn4GDecoding"
date = "2023-04-12T09:01:24+08:00"
description = "ConfigurationOn4GDecoding"
keywords = ["Technology"]
categories = ["Technology"]
+++
Above 4G Decoding (Available if the system supports 64-bit PCI decoding) Select Enabled to decode a PCI device that supports 64-bit in the space above 4G Address. The options are Enabled and Disabled.    

翻译:    

G以上解码（如果系统支持64位PCI解码，则可用）选择 "已启用 "来解码一个支持64位的PCI设备，在4G地址以上的空间。选项是已启用和已禁用。   

禁用的情况:     

```
root@mac:~# lspci -v | grep "Memory.*64-bit"
	Memory at d0000000 (64-bit, prefetchable) [disabled] [size=256M]
	Memory at e0000000 (64-bit, prefetchable) [disabled] [size=2M]
	Memory at fcf60000 (64-bit, non-prefetchable) [size=16K]
	Memory at fce04000 (64-bit, non-prefetchable) [size=4K]
	Memory at fce00000 (64-bit, non-prefetchable) [size=16K]
	Memory at b0000000 (64-bit, prefetchable) [size=256M]
	Memory at c0000000 (64-bit, prefetchable) [size=2M]
	Memory at fca00000 (64-bit, non-prefetchable) [size=1M]
	Memory at fc900000 (64-bit, non-prefetchable) [size=1M]
```
开启的情况:     

```
idv@idv-TC-9070:~$ sudo lspci -v | grep "Memory.*64-bit"
	Memory at de000000 (64-bit, non-prefetchable) [size=16M]
	Memory at c0000000 (64-bit, prefetchable) [size=256M]
	Memory at df110000 (64-bit, non-prefetchable) [size=64K]
	Memory at df12d000 (64-bit, non-prefetchable) [size=4K]
	Memory at df120000 (64-bit, non-prefetchable) [size=16K]
	Memory at df100000 (64-bit, non-prefetchable) [size=64K]
	Memory at df12a000 (64-bit, non-prefetchable) [size=256]
	Memory at df000000 (64-bit, non-prefetchable) [size=4K]
	Memory at d0000000 (64-bit, prefetchable) [size=16K]

```

### Other options in bios
peci:   

```
PECI是用于监测CPU及芯片组温度的一线总线(one-wire bus)，全称是Platform Environment Control Interface。
```
