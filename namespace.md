## 命名空间
PHP 命名空间提供了一种将相关的类、函数和常量组合到一起的途径
在PHP中，命名空间用来解决在编写类库或应用程序时创建可重用的代码如类或函数时碰到的两类问题：
1. 用户编写的代码与PHP内部的类/函数/常量或第三方类/函数/常量之间的名字冲突。
2. 为很长的标识符名称(通常是为了缓解第一类问题而定义的)创建一个别名（或简短）的名称，提高源代码的可读性。

### 命名空间的作用范围
1. 类
2. 函数
3. 常量

### 定义命名空间的类

### 定义命名空间的函数

### 定义命名空间的常量
前面讲定义全局常量的时候也提过注册带命名空间的常量
```
// Zend/zend_constant.h
#define REGISTER_NS_NULL_CONSTANT(ns, name, flags)  zend_register_null_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (flags), module_number)
#define REGISTER_NS_BOOL_CONSTANT(ns, name, bval, flags)  zend_register_bool_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (bval), (flags), module_number)
#define REGISTER_NS_LONG_CONSTANT(ns, name, lval, flags)  zend_register_long_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (lval), (flags), module_number)
#define REGISTER_NS_DOUBLE_CONSTANT(ns, name, dval, flags)  zend_register_double_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (dval), (flags), module_number)
#define REGISTER_NS_STRING_CONSTANT(ns, name, str, flags)  zend_register_string_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (str), (flags), module_number)
#define REGISTER_NS_STRINGL_CONSTANT(ns, name, str, len, flags)  zend_register_stringl_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (str), (len), (flags), module_number)

// Zend/zend_API.h
#define ZEND_NS_NAME(ns, name)			ns "\\" name

拿注册Long类型的常量来看
#define REGISTER_LONG_CONSTANT(name, lval, flags)  zend_register_long_constant((name), sizeof(name)-1, (lval), (flags), module_number)
#define REGISTER_NS_LONG_CONSTANT(ns, name, lval, flags)  zend_register_long_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (lval), (flags), module_number)

由上面可以看出，命名空间只是给常量名前面加了一个前缀，用 "\\" 分隔，其他的都和不带命名空间的常量一样的处理方式

看一下具体怎么注册的:
PHP_MINIT_FUNCTION(test)
{
    REGISTER_LONG_CONSTANT("CODE", 10, CONST_CS | CONST_PERSISTENT);
	  REGISTER_NS_LONG_CONSTANT("test", "NSCODE", 11, CONST_CS | CONST_PERSISTENT);
  	
    return SUCCESS;
}

测试:
php -r 'var_dump(CODE);'
输出: int(10)

php -r 'var_dump(test\NSCODE);'
输出: int(11)

```
