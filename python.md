## 函数

函数是**组织好的可以重复使用的，用来实现特定功能**的**代码块**

> 函数定义

![image-20220830135758490](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20220830135758490.png)

- 函数使用步骤：先定义后使用
- 注意事项
  - 参数不需要的，可以省略
  - 返回值不需要的可以省略

> 函数的传入参数

- 功能：在函数计算的时候，接受外部调用时提供的数据

- 函数定义中，提供的x和y，称之为形式参数（形参），表示函数的声明需要使用两个参数
  
  - 参数之间使用逗号分隔

- 函数调用之中，提供5和6，称之为实参，表示函数调用的时候真正使用的参数值
  
  - 传入参数的时候，按照顺序传入数据，使用逗号分隔

> 函数返回值

程序中函数完成任务后，返回的结果

- 无返回值的函数，返回的是None
  - None作为一个特殊的字面量，用于表示：空，无意义，
  - 用在函数无返回值上
  - 用在if判断中，None等同于False,
  - ![image-20220830144110689](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20220830144110689.png)
  - 用于声明无内容的变量上，定义变量，但是暂时不需要变量有具体值，可以用None来代替

> 嵌套调用

在一个函数中调用另一个函数

执行流程：

函数A执行到调用函数B的语句，会将函数B全部执行完成之后，继续执行函数A的剩余内容

> 变量在函数中额作用域

- 变量的作用域指的是变量的作用范围分为局部变量和全局变量

- 局部变量指的是函数内部生效的变量，

- 全局变量指的是在

- global关键字，可以将函数内定义的变量声明为全局变量

>  函数综合案例

## 数据容器

list 列表

tuple 元组

str 字符串

set 集合

dict 字典

> list 列表

![image-20220830172047721](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20220830172047721.png)

```python
name_list=['aa','bb','cc']
print(name_list)
print(type(name_list))
```

- 列表之间可以嵌套
- 元素之间的类型可以不同

> 列表的下标索引

- 如何取出特定位置的数据，通过下标索引
  - 或者可反向索引，-1 -2 -3 -4一次递减
- 对于嵌套的列表 list\[0][1]
- 下标取值不能越界

![image-20220830173510003](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20220830173510003.png)

> 列表的常用操作

- 查询
  
  - 查找指定元素在列表中的位置，找不到会报错valueerror
  
  - ```
    
    ```

- 插入

- 删除

- 追加

- 修改

- 清空列表

- ![image-20220830214315277](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20220830214315277.png)

> list列表的循环

- for循环

```python
for 临时变量 in 数据容器:
    print(临时变量)
```

> 元组

- 列表的数据是可以修改的，有泄露的风险
- 元组一旦==定义==就不能修改了
- 元组的定义：使用小括号，使用逗号隔开，数据可以是不同类型。
- 元组也支持嵌套

![image-20220830220855888](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20220830220855888.png)

> 元组操作

![image-20220830221347014](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20220830221347014.png)

![image-20220830223332746](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20220830223332746.png)

## Python类和模块（Class，Moudle）

每个类都有自己的属性和方法

类的定义以Class开头，类名的首字母推荐大写，冒号之后换行缩进紧跟着属性和方法的定义

```python
class Person
    name="jack"
    age=45
    def getet(self):
        print("hi myname id +self.name")

p1=Person()
p1.getet()
```
