## Golang 随身笔记本
### 1.Golang 数据类型比较问题？
先梳理下 Golang 中哪些数据类型可以比较，哪些数据类型不可比较:
- 可比较：Integer，Float，String，Boolean，Complex，Pointer，Channel，Interface，Array
- 不可比较：Slice，Map，Function

扩展：Golang 之 struct 能不能比较，涉及到以下场景：
- 同一个 struct 的两个实例能不能比较？

结论：**同一个 struct 的两个实例可直接比较也不可直接比较，当结构体中不包含不可直接比较的成员变量时可直接比较，
否则不可直接比较**

- 两个不同 struct 的实例能不能比较？

结论：一样的，可直接比较（强制转换一下），也不可以直接比较（包含不可直接比较的成员变量）
>- 参考链接：https://juejin.cn/post/6881912621616857102