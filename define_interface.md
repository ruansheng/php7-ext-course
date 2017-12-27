## 定义接口
扩展中可以定义接口，定义的接口可以在本扩展中、其他扩展中、或者用户.php文件中被实现

### 在扩展中定义接口
```
在扩展中注册接口还是和常规类的代码差不多，区别只是在: 
1. 注册这个interface到zend engine是用zend_register_internal_interface，而不是zend_register_internal_class
2. 设置类型是 my_test_ce->ce_flags = ZEND_ACC_INTERFACE; 

const zend_function_entry 	test_methods[] = {
		{NULL, NULL, NULL}
};

// 在 PHP_MINIT_FUNCTION(test) 钩子函数中注册类，初始化 my_test_ce
PHP_MINIT_FUNCTION(test)
{
	zend_class_entry ce;
	INIT_CLASS_ENTRY(ce, "Myinterface", test_methods);
	my_test_ce = zend_register_internal_interface(&ce);
	my_test_ce->ce_flags = ZEND_ACC_INTERFACE; 

	return SUCCESS;
}


编译加载完扩展就才能测试
php -r 'class AAA implements Myinterface{};var_dump(new AAA());'
输出:
object(AAA)#1 (0) {
}
```
