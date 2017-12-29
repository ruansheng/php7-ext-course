## 类成员方法
类成员方法修饰词 public、private、protected、static、final、abstract，默认的是 public
1. public method 当类的成员方法被声明为public的访问修饰符时，该成员能被外部代码访问和操作
2. private method 被定义为private的成员方法，对于类内部所有成员都可见，没有访问限制。对类外部不允许访问
3. protected method 被声明为protected的成员方法，只允许该类的子类进行访问
4. static method 静态方法，不需要实例化就可以使用 ClassName::staticMethodName()
5. final method 表示方法不能被重写
6. abstract method 表示方法不能有实现，只有声明

### 定义成员方法
```
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

### 具体用法
```
const zend_function_entry 	test_methods[] = {
		PHP_ME(test, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
		ZEND_ABSTRACT_ME(test, abstract_func, NULL)
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
