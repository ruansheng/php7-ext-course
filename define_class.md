## 定义类
在扩展中定义的类，在我们经常写的.php文件中，可以调用，这里以test扩展为例

### 类的修饰
```
PHP中class可以指定一些修饰符，这些修饰符分别是: final、abstract
1. 常规类
class test{}
	默认不加修饰词的class，可以继承基类、实现接口
2. 最终类
final class test{}
	当final修饰class的时候，此类被限定为不可继承类，也就是其他类无法继承此类，被称为最终类
      	当你不想让别人继承自己的编写的类时只需要在前面加上final关键字即可
3. 抽象类	
abstract class test{}
	当abstract修饰class的时候，此类被限定为抽象类，只能用于继承，而无法实例化对象，也就是不能new class。     
```

### 扩展中定义常规类class
```
// 在test.c中定义一个 zend_class_entry 结构体指针变量，作为定义的class的句柄
zend_class_entry * my_test_ce;

// 定义类的method数组，每一个类的method都要在这个数组中初始化定义，这里先不添加method
const zend_function_entry 	test_methods[] = {
       //PHP_ME(test, __construct, NULL, ZEND_ACC_PUBLIC | ZEND_ACC_CTOR)
	{NULL, NULL, NULL}
};

// 在 PHP_MINIT_FUNCTION(test) 钩子函数中注册类，初始化 my_test_ce
PHP_MINIT_FUNCTION(test)
{
	zend_class_entry ce;
	INIT_CLASS_ENTRY(ce, "Mytest", test_methods);
	my_test_ce = zend_register_internal_class(&ce);  // 注册这个class到zend engine

	return SUCCESS;
}

** 解释说明 **
// INIT_CLASS_ENTRY(ce, "Mytest", test_methods);
ce  是zend_class_entry变量
"Mytest" 是类的名称
test_methods 是一个定义类的成员方法数组

编译加载完扩展就才能测试
php -r 'var_dump(new Mytest());'
输出:
object(Mytest)#1 (0) {
}
```

### 扩展中定义 abstract class
```
在扩展中注册抽象类还是和常规类一样的方式，区别只是在: 注册这个class到zend engine 之后需要设置一下这个类的类型
举报代码是 my_test_ce->ce_flags = ZEND_ACC_EXPLICIT_ABSTRACT_CLASS; 

// 在 PHP_MINIT_FUNCTION(test) 钩子函数中注册类，初始化 my_test_ce
PHP_MINIT_FUNCTION(test)
{
	zend_class_entry ce;
	INIT_CLASS_ENTRY(ce, "Mytest", test_methods);
	my_test_ce = zend_register_internal_class(&ce);  
	my_test_ce->ce_flags = ZEND_ACC_EXPLICIT_ABSTRACT_CLASS; 

	return SUCCESS;
}

编译加载完扩展就才能测试
php -r 'var_dump(new Mytest());'
输出:
PHP Fatal error:  Uncaught Error: Cannot instantiate abstract class Mytest in Command line code:1
Stack trace:
#0 {main}
  thrown in Command line code on line 1
// 由上测试可以看出抽象类是不能被实例化的，否则会报致命错误

php -r 'class AAA extends Mytest{};var_dump(new AAA());'
输出:
object(AAA)#1 (0) {
}
// 由上测试可以看出抽象类是可以被继承的
```

### 扩展中定义 final class
```
我们可以找到一个 ZEND_ACC_FINAL_CLASS 宏，是用来注册设置final class的
这里有个特别的点需要注意，类似注册抽象类一样的，那么注册final class是不是也是my_test_ce->ce_flags = ZEND_ACC_FINAL_CLASS;呢
这里有点疑惑，在PHP5.x的时候是这样的，但是在PHP7中被移除了
如果在PHP7的扩展里面这样写，编译的时候会报错:
错误：‘ZEND_ACC_FINAL_CLASS’未声明(在此函数内第一次使用)
```
