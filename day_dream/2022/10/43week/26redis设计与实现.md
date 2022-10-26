# 技术
## redis设计与实现
2.5小时
### intset
```c
typedef struct intset {
    //编码方式
    uint32_t encoding;
    //集合包含的元素数量
    uint32_t length;
    //保存元素的数组
    int8_t contents[];
} intset;
```

虽然intset结构将contents属性声明为int8_t类型的数组，但实际上contents数组并不保存任何int8_t类型的值，contents数组的真正类型取决于encoding属性的值：

- 如果encoding属性的值为INTSET_ENC_INT16，那么contents就是一个int16_t类型的数组，数组里的每个项都是一个int16_t类型的整数值（最小值为-32768，最大值为32767）。
- 如果encoding属性的值为INTSET_ENC_INT32，那么contents就是一个int32_t类型的数组，数组里的每个项都是一个int32_t类型的整数值（最小值为-2147483648，最大值为2147483647）。
- 如果encoding属性的值为INTSET_ENC_INT64，那么contents就是一个int64_t类型的数组，数组里的每个项都是一个int64_t类型的整数值（最小值为-9223372036854775808，最大值为9223372036854775807）。


`((int64_t*)is->contents)+pos` 直接找源代码看了一下，发现是会强转指针类型，然后再去加pos下标，这样就可以跳转指定类型大小了，存的时候也进行了指针的强行转换: `((int64_t*)is->contents)[pos] = value;`

证据来源: https://github.com/redis/redis/blob/unstable/src/intset.c

# 运动
30俯卧撑，60仰卧起坐

# 总结
+2.5自信币和自信点