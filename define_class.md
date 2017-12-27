## 定义类
在扩展中定义的类，在我们经常写的.php文件中，可以调用，这里以test扩展为例

### 类的修饰
```
PHP中class可以指定一些修饰符，这些修饰符分别是: final、abstract
final: 当final修饰class的时候，此类被限定为不可继承类，也就是其他类无法继承此类，被称为最终类
       当你不想让别人继承自己的编写的类时只需要在前面加上final关键字即可
abstract: 当abstract修饰class的时候，此类被限定为抽象类，只能用于继承，而无法实例化对象，也就是不能new class。     
```

### 扩展中定义class
```
// 在test.c中定义一个 zend_class_entry 结构体指针变量，作为定义的class的句柄
zend_class_entry * my_test_ce;

// 定义类的method数组，每一个类的method都要在这个数组中初始化定义，这里先不添加method
const zend_function_entry 	test_methods[] = {
       //PHP_ME(test, __construct, NULL, ZEND_ACC_PUBLIC | ZEND_ACC_CTOR)
	{NULL, NULL, NULL}
};

// 在 PHP_MINIT_FUNCTION(test) 钩子函数中注册类，初始化 my_test_ce
PHP_MINIT_FUNCTION(test_ext)
{
	zend_class_entry ce;
	INIT_CLASS_ENTRY(ce, "Mytest", test_methods);
	my_test_ce = zend_register_internal_class(&ce);

	return SUCCESS;
}

** 解释说明 **
// INIT_CLASS_ENTRY(ce, "Mytest", test_methods);
ce  是zend_class_entry变量
"Mytest" 是类的名称
test_methods 是一个定义类的成员方法数组
```

### 测试
编译加载完扩展就才能测试
```
php -r 'var_dump(new Mytest());'
输出:
object(Mytest)#1 (0) {
}
```
