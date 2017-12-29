## 类常量
1. 类常量在PHP中可以通过 const 定义
2. 类常量和普通常量一样，定义之后就不能被修改

### 类常量\接口常量的访问
1. ClassName::ConstantName InteraceName::ConstantName
2. self::ConstantName 这种访问方式只能在类自身内部

### 定义类常量/接口常量
注册类常量和注册接口常量是一样的
```
zend_class_entry * my_test_a_ce;

PHP_METHOD(TestA, __construct)
{
	php_printf("TestA __construct\n");
}

const zend_function_entry 	test_a_methods[] = {
		PHP_ME(TestA,__construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
		{NULL, NULL, NULL}
};

PHP_MINIT_FUNCTION(test_ext)
{
  zend_class_entry ce_a;
	INIT_CLASS_ENTRY(ce_a, "TestA", test_a_methods);
	my_test_a_ce = zend_register_internal_class(&ce_a);
  
  // 注册类常量
  zval zv_es_version;
  ZVAL_PSTRING(&zv_es_version, "1.0.0");
  zend_declare_class_constant(my_test_a_ce, "VERSION", sizeof("VERSION") - 1, &zv_es_version);
}

测试:
php -r 'var_dump(TestA::VERSION);'
输出:
string(5) "1.0.0"
```

