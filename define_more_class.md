## 扩展中定义多个类

### 在同一个.c文件中定义多个类
```
这种方式其实就是在和定义一个类的方式一样，无非就是在同样的地方，一样的代码来两份，改个变量名，改个类名
同理可以定义2、3、4 ... 类都是一样的方式

zend_class_entry * my_test_a_ce;
zend_class_entry * my_test_b_ce;

const zend_function_entry 	test_a_methods[] = {
		//PHP_ME(TestA, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
		{NULL, NULL, NULL}
};

const zend_function_entry 	test_b_methods[] = {
		//PHP_ME(TestB, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
		{NULL, NULL, NULL}
};

PHP_MINIT_FUNCTION(test_ext)
{
	zend_class_entry ce_a;
	INIT_CLASS_ENTRY(ce_a, "TestA", test_a_methods);
	my_test_a_ce = zend_register_internal_class(&ce_a);

	zend_class_entry ce_b;
	INIT_CLASS_ENTRY(ce_b, "TestB", test_b_methods);
	my_test_b_ce = zend_register_internal_class(&ce_b);

	return SUCCESS;
}

// 测试
php -r 'var_dump(new TestA);var_dump(new TestB());'
输出:
object(TestA)#1 (0) {
}
object(TestB)#1 (0) {
}

```

### 更优雅的定义方式
