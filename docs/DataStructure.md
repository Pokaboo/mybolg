## 数据结构

### 位运算
- 左移（<<）:

- 右移（>>）:

- 无符号右移（>>>）:

- 位与（&）:

- 位或（|）：

- 位异或（^）：

- 位非（~）：

  ```
  // 右移( >> ) 高位补符号位
  // 右移2位，高位补0
  // 5： 0000 0101
  // >>: 0000 0001
  System.out.println(5 >> 2);   // 1
  
  // 左移2位后，低位补0
  // 5： 0000 0101
  // <<: 0001 0100
  //   2^4+2^2  = 20
  System.out.println(5 << 2);  // 20
  
  // 无符号右移( >>> ) 高位补0
  System.out.println(5 >>> 3); //0
  
  // 5:  0000 0101
  // -5: 1111 1011    (5取反加1)
  // >>: 1111 1111
  System.out.println(-5 >> 3); // -1
  System.out.println(-5 >>> 3); //
  
  // 位与( & )
  // 位与：第一个操作数的的第n位于第二个操作数的第n位如果都是1，那么结果的第n为也为1，否则为0
  System.out.println(5 & 3); // 1
  
  // 位或( | )
  // 第一个操作数的的第n位于第二个操作数的第n位 只要有一个是1，那么结果的第n为也为1，否则为0
  System.out.println(5 | 3);// 结果为7
  
  // 位异或( ^ )
  // 第一个操作数的的第n位于第二个操作数的第n位 相反，那么结果的第n为也为1，否则为0
  System.out.println(5 ^ 3);//结果为6
  
  // 位非( ~ )
  // 操作数的第n位为1，那么结果的第n位为0，反之。
  System.out.println(~5);// 结果为-6
  ```

### 数组
- 数组是一种顺序存储的线性表，所有元素的内存地址是连续的。无法动态修改容量