## 类的继承
PHP的类只允许继承一个父类

### 类继承
之前在看定义类的时候用的是 zend_register_internal_class 函数，PHP中类只允许继承一个类
这里有一个与之类似的函数来处理类继承关系 zend_register_internal_class_ex
```
ZEND_API zend_class_entry *zend_register_internal_class(zend_class_entry *orig_class_entry)
ZEND_API zend_class_entry *zend_register_internal_class_ex(zend_class_entry *class_entry, zend_class_entry *parent_ce)

可见 zend_register_internal_class_ex 的第二个参数是传一个父类 zend_class_entry * 变量
那么可以定义2个类，第一个类作为父类，第二个类就设置第一个类作为其父类
```
看下面代码
```
zend_class_entry * my_test_a_ce;
zend_class_entry * my_test_b_ce;

PHP_METHOD(TestA, __construct)
{
	php_printf("TestA __construct\n");
}

PHP_METHOD(TestB, __construct)
{
  	php_printf("TestB __construct\n");
}

const zend_function_entry 	test_a_methods[] = {
	PHP_ME(TestA, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
	{NULL, NULL, NULL}
};

const zend_function_entry 	test_b_methods[] = {
	PHP_ME(TestB, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
	{NULL, NULL, NULL}
};

PHP_MINIT_FUNCTION(test_ext)
{
	zend_class_entry ce_a;
	INIT_CLASS_ENTRY(ce_a, "TestA", test_a_methods);
	my_test_a_ce = zend_register_internal_class(&ce_a);

	zend_class_entry ce_b;
	INIT_CLASS_ENTRY(ce_b, "TestB", test_b_methods);
	my_test_b_ce = zend_register_internal_class_ex(&ce_b, my_test_a_ce);

	return SUCCESS;
}

测试:
php -r 'var_dump(new TestB());'
输出: TestB __construct
      object(TestB)#1 (0) {
      }
```

### 调用父类的 __construct 方法
在PHP中默认如果在子类中不主动调用父类的 __construct，那么父类的 __construct 是不会被调用的
```
PHP_METHOD(TestA, __construct)
{
	php_printf("TestA __construct\n");
}

PHP_METHOD(TestB, __construct)
{
	php_printf("TestB __construct\n");
	
	zval *this_zval;
    	this_zval = getThis();
    	zend_call_method(&this_zval, my_test_a_ce, NULL, "__construct", sizeof("__construct") - 1, NULL, 0, NULL, NULL);
}

测试:
php -r 'var_dump(new TestB());'
输出: TestB __construct
      TestA __construct
      object(TestB)#1 (0) {
      }
```
