# iOS-Demo

some demo objc code

1. 整体结构

   ```plaintext
   ====================
   Main VC
   =============
   demo list
   ---
   demo 1 --> subVC
   ---
   demo 2 --> subVC
   ---
   demo 3 --> subVC
   ---
   ====================
   ```
2. 考虑挂载的方式

* sub VC 如何挂载
  1. 继承，添加到 mountArray中
  2. 接口，实例方法，添加到 mountArray中
  3. 全局方法， 静态容器，接口，实例方法
  4. 接口，类方法
* 是否使用注入，以及注入方式
  1. 提供serviceProvider, 获取方式，泛型提供，类方法提供，初始化实例方法提供，全局方法提供，宏提供
  2. 多协议注入的能力，一个实例注入为多个Protocol,
