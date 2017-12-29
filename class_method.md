## 类成员方法
类成员方法修饰词 public、private、protected、static、final、abstract，默认的是 public
1. public method 当类的成员方法被声明为public的访问修饰符时，该成员能被外部代码访问和操作
2. private method 被定义为private的成员方法，对于类内部所有成员都可见，没有访问限制。对类外部不允许访问
3. protected method 被声明为protected的成员方法，只允许该类的子类进行访问
4. static method 静态方法，不需要实例化就可以使用 ClassName::staticMethodName()
5. final method 表示方法不能被重写
6. abstract method 表示方法不能有实现，只有声明，其所在的类也不能被实例化

### 定义成员方法
```
// main/php.h
#define PHP_ME          ZEND_ME
#define PHP_ABSTRACT_ME ZEND_ABSTRACT_ME

// Zend/zend_API.h
#define ZEND_ME(classname, name, arg_info, flags)	ZEND_FENTRY(name, ZEND_MN(classname##_##name), arg_info, flags)
#define ZEND_ABSTRACT_ME(classname, name, arg_info)	ZEND_FENTRY(name, NULL, arg_info, ZEND_ACC_PUBLIC|ZEND_ACC_ABSTRACT)

这里 ZEND_ABSTRACT_ME 是用来定义abstract method 的
```

### 成员方法修饰符
```
// Zend/zend_compile.h
#define ZEND_ACC_STATIC			0x01
#define ZEND_ACC_ABSTRACT		0x02
#define ZEND_ACC_FINAL			0x04

#define ZEND_ACC_PUBLIC		0x100
#define ZEND_ACC_PROTECTED	0x200
#define ZEND_ACC_PRIVATE	0x400

#define ZEND_ACC_CTOR		0x2000
#define ZEND_ACC_DTOR		0x4000
```

### 成员方法参数定义
```
ZEND_BEGIN_ARG_INFO_EX(arginfo_public_func_, 0, 0, 3)
	ZEND_ARG_INFO(0, arg1)
    ZEND_ARG_INFO(0, arg2)
    ZEND_ARG_INFO(0, arg3)
ZEND_END_ARG_INFO()

// Zend/zend_API.h
#define ZEND_BEGIN_ARG_INFO_EX(name, _unused, return_reference, required_num_args)	\
	static const zend_internal_arg_info name[] = { \
		{ (const char*)(zend_uintptr_t)(required_num_args), 0, return_reference, 0 },
#define ZEND_BEGIN_ARG_INFO(name, _unused)	\
	ZEND_BEGIN_ARG_INFO_EX(name, 0, ZEND_RETURN_VALUE, -1)
#define ZEND_END_ARG_INFO()		};

#define ZEND_ARG_INFO(pass_by_ref, name)                             { #name, 0, pass_by_ref, 0},

宏展开:
static const zend_internal_arg_info name[] = {
		{ (const char*)(zend_uintptr_t)(required_num_args), 0, return_reference, 0 },
        { #name, 0, pass_by_ref, 0},
};
可见也是对结构体数组的初始化，这样的操作在PHP中实在是太多了
    
使用的时候:
PHP_ME(test, public_func, arginfo_public_func_, ZEND_ACC_PUBLIC)
```

### 具体用法
```
const zend_function_entry 	test_methods[] = {
    PHP_ME(test, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
    PHP_ABSTRACT_ME(test, abstract_func, NULL)
    PHP_ME(test, public_func, NULL, ZEND_ACC_PUBLIC)
    PHP_ME(test, private_func, NULL, ZEND_ACC_PRIVATE)
    PHP_ME(test, protected_func, NULL, ZEND_ACC_PROTECTED)
    PHP_ME(test, static_func, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_STATIC)  // 这里可以有几种修饰方式共用
    PHP_ME(test, abstract_func, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_ABSTRACT) // 这里可以有几种修饰方式共用
    PHP_ME(test, abstract_func, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_FINAL) // 这里可以有几种修饰方式共用
    PHP_ME(test, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_DTOR)
    {NULL, NULL, NULL}
};
```
