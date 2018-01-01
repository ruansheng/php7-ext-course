## 调用类中的方法
在扩展可以调用:

用户在.php文件中定义的类的方法
其他扩展中定义的类的方法
PHP内核中的类的方法

### call_user_function
call_user_function是PHP内核中定义的一个宏，可以调用函数也可以调用类的方法
```
#define call_user_function(function_table, object, function_name, retval_ptr, param_count, params)
使用call_user_function宏调用method同前面讲到的调用function的方式类似，不同点在于第二个参数object，
这时需要通过 object_init_ex 实例化一个类，然后才能调用其method

下面请看 object_init_ex 宏的定义
#define object_init_ex(arg, ce)	_object_init_ex((arg), (ce) ZEND_FILE_LINE_CC)
参数解释:
arg 为生成的对象，类型为zval*
ce 是要实例化的类
```

### zend_call_method
zend_call_method是PHP内核中定义的一个函数
```
ZEND_API zval* zend_call_method(zval *object_pp, zend_class_entry *obj_ce, zend_function **fn_proxy, const char *function_name, size_t function_name_len, zval *retval, int param_count, zval* arg1, zval* arg2);
参数解释:
zval *object_pp           类的实例化对象变量指针
zend_class_entry *obj_ce  类句柄指针变量
zend_function **fn_proxy
const char *function_name  方法名称字符串
size_t function_name_len 方法名称字符串长度
zval *retval  调用返回结果
int param_count 传参数个数
zval* arg1, zval* arg2  参数
```
下面例子是子类的__construct()中调用父类的__construct()，调用其他父类的其他method也是类似的
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
  zval *this_zval;
  this_zval = getThis();
  zend_call_method(&this_zval, my_test_a_ce, NULL, "__construct", sizeof("__construct") - 1, NULL, 0, NULL, NULL);
}
```

