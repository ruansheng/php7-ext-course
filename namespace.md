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
```
前面也讲过注册不带命名空间的类
PHP_MINIT_FUNCTION(test)
{
	zend_class_entry ce;
	INIT_CLASS_ENTRY(ce, "Mytest", test_methods);
	my_test_ce = zend_register_internal_class(&ce);  

	return SUCCESS;
}

其实带命名空间的类和不带命名空间的类，在注册的时候差别很小，把 INIT_CLASS_ENTRY 换成 INIT_NS_CLASS_ENTRY，加个命名空间参数即可

// Zend/zend_API.h
#define INIT_NS_CLASS_ENTRY(class_container, ns, class_name, functions) \
	INIT_CLASS_ENTRY(class_container, ZEND_NS_NAME(ns, class_name), functions)
#define INIT_CLASS_ENTRY(class_container, class_name, functions) \
	INIT_OVERLOADED_CLASS_ENTRY(class_container, class_name, functions, NULL, NULL, NULL)
#define ZEND_NS_NAME(ns, name)			ns "\\" name

INIT_NS_CLASS_ENTRY 最终也调用 INIT_CLASS_ENTRY 
由上面可以看出，命名空间只是给常量名前面加了一个前缀，用 "\\" 分隔，其他的都和不带命名空间的类一样的处理方式

// 具体例子
PHP_MINIT_FUNCTION(test)
{
	zend_class_entry ce;
	INIT_NS_CLASS_ENTRY(ce, "test", "Mytest", test_methods);
	my_test_ce = zend_register_internal_class(&ce);  

	// 这里顺便注册一个类常量，可以看到类常量也是用带命名空间的类来访问，同理:类成员、成员方法也是一样的
	zval zv_version;
  	ZVAL_PSTRING(&zv_version, "1.0.0");
  	zend_declare_class_constant(ce, "VERSION", sizeof("VERSION") - 1, &zv_version);

	return SUCCESS;
}

测试:
php -r 'var_dump(new test\Mytest());'
输出:  Mytest __construct
      object(test\Mytest)#1 (0) {
      }
      
php -r 'var_dump(test\Test::VERSION);'
输出:  string(5) "1.0.0"      
```

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
