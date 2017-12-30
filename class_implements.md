## 类实现接口
PHP实现接口有一些约束:
1. 接口只能定义方法原型，不能有方法的实现，方法名不能有重复，这点和java不同
2. 一个接口可以被多个不同的类实现
3. 实现接口的类，在实现的方法参数必须一致
4. 一个类可以实现多个不同的接口
5. 接口也可以继承，通过关键字extends来继承
6. 接口中可以定义常量，和类中定义类常量是一样的
7. 如果一个类实现了一个接口，那么:
  a. 这个类不实现接口方法 这个类被认为是abstract class，因而不能被实例化，其他子类又可以来继承它，再实现接口方法也是可以的
  a. 这个类只实现部分接口方法 这个类被认为是abstract class，因而不能被实例化，其他子类又可以来继承它，再实现接口方法也是可以的
  b. 这个类实现全部接口方法 这个类可以被实例化

### zend_class_implements函数声明类实现接口
```
// Zend/zend_API.c
ZEND_API void zend_class_implements(zend_class_entry *class_entry, int num_interfaces, ...) /* {{{ */
{
	zend_class_entry *interface_entry;
	va_list interface_list;
	va_start(interface_list, num_interfaces);

	while (num_interfaces--) {
		interface_entry = va_arg(interface_list, zend_class_entry *);
		zend_do_implement_interface(class_entry, interface_entry);
	}

	va_end(interface_list);
}
这里zend_class_implements函数要说明一下参数:
zend_class_entry *class_entry 类的指针变量
int num_interfaces 实现的接口数量，这个参数和后面的'...' 参数配合使用
... 这是一个可变参数，如果 num_interfaces=1，那后面就传一个接口的zend_class_entry *变量，num_interfaces=2，就传2个，这样一个类就可以实现多个接口
```

### ZEND_ABSTRACT_ME 宏
```
看到 ZEND_ABSTRACT_ME 宏是不是就想起 PHP_ME 宏，看名字就能猜出是做什么用的吧
在abstract class 中也是用 ZEND_ABSTRACT_ME 来定义abstract method 的

// Zend/zend_api.h
#define ZEND_ME(classname, name, arg_info, flags)	ZEND_FENTRY(name, ZEND_MN(classname##_##name), arg_info, flags)
#define ZEND_ABSTRACT_ME(classname, name, arg_info)	ZEND_FENTRY(name, NULL, arg_info, ZEND_ACC_PUBLIC|ZEND_ACC_ABSTRACT)
```

### 实现接口
```
zend_class_entry * my_test_a_ce;
zend_class_entry * my_test_b_ce;

PHP_METHOD(TestB, __construct)
{
	php_printf("TestB __construct\n");
}

PHP_METHOD(TestB, show)
{
	php_printf("TestB show\n");
}

PHP_METHOD(TestB, dance)
{
	php_printf("TestB dance\n");
}

PHP_METHOD(InterfaceA, show);
PHP_METHOD(InterfaceA, dance);
const zend_function_entry 	test_a_methods[] = {
	ZEND_ABSTRACT_ME(InterfaceA, show, NULL)
	ZEND_ABSTRACT_ME(InterfaceA, dance, NULL)
	{NULL, NULL, NULL}
};

const zend_function_entry 	test_b_methods[] = {
	PHP_ME(TestB, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
	PHP_ME(TestB, show, NULL, ZEND_ACC_PUBLIC)
	PHP_ME(TestB, dance, NULL, ZEND_ACC_PUBLIC)
	{NULL, NULL, NULL}
};

PHP_MINIT_FUNCTION(test_ext)
{
	zend_class_entry ce_a;
	INIT_CLASS_ENTRY(ce_a, "InterfaceA", test_a_methods);
	my_test_a_ce = zend_register_internal_interface(&ce_a); // 注册接口

	zend_class_entry ce_b;
	INIT_CLASS_ENTRY(ce_b, "TestB", test_b_methods);
	my_test_b_ce = zend_register_internal_class(&ce_b);  // 注册类
	zend_class_implements(my_test_b_ce, 1, my_test_a_ce); // 声明 my_test_b_ce 类 实现 my_test_a_ce接口

	return SUCCESS;
}

// 如果只是实现了接口，而没有完全实现接口内部的方法，那么这个类还是会被认为是一个abstract class
1. 任何一个接口方法都不实现
调试可以看到:
php -r 'new TestB();'
PHP Fatal error:  Uncaught Error: Cannot instantiate abstract class TestB in Command line code:1
Stack trace:
#0 {main}
  thrown in Command line code on line 1
  
2. 只实现部分接口方法 
调试可以看到:
php -r 'new TestB();'
PHP Fatal error:  Uncaught Error: Cannot instantiate abstract class TestB in Command line code:1
Stack trace:
#0 {main}
  thrown in Command line code on line 1

3. 实现接口的全部接口方法
调试可以看到:
php -r 'var_dump(new TestB());'
object(TestB)#1 (0) {
}
```
