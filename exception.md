## 扩展中异常与错误处理

### 抛出异常
```
// Zend/zend_exception.h
ZEND_API ZEND_COLD zend_object *zend_throw_exception(zend_class_entry *exception_ce, const char *message, zend_long code);

// Zend/zend_exception.c
ZEND_API ZEND_COLD zend_object *zend_throw_exception(zend_class_entry *exception_ce, const char *message, zend_long code) /* {{{ */
{
	zval ex, tmp;

	if (exception_ce) {
		if (!instanceof_function(exception_ce, zend_ce_throwable)) {
			zend_error(E_NOTICE, "Exceptions must implement Throwable");
			exception_ce = zend_ce_exception;
		}
	} else {
		exception_ce = zend_ce_exception;
	}
	object_init_ex(&ex, exception_ce);


	if (message) {
		ZVAL_STRING(&tmp, message);
		zend_update_property_ex(exception_ce, &ex, ZSTR_KNOWN(ZEND_STR_MESSAGE), &tmp);
		zval_ptr_dtor(&tmp);
	}
	if (code) {
		ZVAL_LONG(&tmp, code);
		zend_update_property_ex(exception_ce, &ex, ZSTR_KNOWN(ZEND_STR_CODE), &tmp);
	}

	zend_throw_exception_internal(&ex);
	return Z_OBJ(ex);
}

// 先来看一看PHP中定义的Exception类:摘自PHP官网
Exception {
  /* 属性 */
  protected string $message ;
  protected int $code ;
  protected string $file ;
  protected int $line ;
  /* 方法 */
  public __construct ([ string $message = "" [, int $code = 0 [, Throwable $previous = NULL ]]] )
  final public string getMessage ( void )
  final public Throwable getPrevious ( void )
  final public int getCode ( void )
  final public string getFile ( void )
  final public int getLine ( void )
  final public array getTrace ( void )
  final public string getTraceAsString ( void )
  public string __toString ( void )
  final private void __clone ( void )
}

从上面代码中可以看出:
第一个参数 zend_class_entry *exception_ce 允许我们传一个异常类进去，这个类也可以是我们自定义的
第二个参数 const char *message 传递一个描述异常的字符串，也是给Exception的$message属性赋值，可以通过$e->getMessage() 来获取
第三个参数 zend_long code，在上面PHP中定义的Exception中有一个属性$code，这里就是同一个code，实例化后可以通过$e->getCode() 来获取

// 具体测试例子
PHP_FUNCTION(myexception) 
{
	zend_throw_exception(NULL, "This is my throw Exception", 601);
}

写测试用例:
//exception.php
<?php
try{
    myexception();
}catch(Exception $e){
    echo $e->getCode().PHP_EOL;
    echo $e->getMessage().PHP_EOL;
}

测试:
php exception.php
输出: 
601
This is my throw Exception
```

### 抛出错误
```
// Zend/zend.h
ZEND_API ZEND_COLD void zend_throw_error(zend_class_entry *exception_ce, const char *format, ...) ZEND_ATTRIBUTE_FORMAT(printf, 2, 3);

// Zend/zend.c
ZEND_API ZEND_COLD void zend_throw_error(zend_class_entry *exception_ce, const char *format, ...) /* {{{ */
{
	va_list va;
	char *message = NULL;
	
	if (exception_ce) {
		if (!instanceof_function(exception_ce, zend_ce_error)) {
			zend_error(E_NOTICE, "Error exceptions must be derived from Error");
			exception_ce = zend_ce_error;
		}
	} else {
		exception_ce = zend_ce_error;
	}

	va_start(va, format);
	zend_vspprintf(&message, 0, format, va);

	//TODO: we can't convert compile-time errors to exceptions yet???
	if (EG(current_execute_data) && !CG(in_compilation)) {
		zend_throw_exception(exception_ce, message, 0);
	} else {
		zend_error(E_ERROR, "%s", message);
	}

	efree(message);
	va_end(va);
}

在PHP7中，抛出的错误也是可以被捕获到的
从上面代码中可以看出:
第一个参数 zend_class_entry *exception_ce 允许我们传一个异常类进去，这个类也可以是我们自定义的
第二个参数 const char *format 传递一个描述错误的格式化字符串，类似于pintf()函数一样，可支持格式化符号
第三个参数 ...，这里可以看出是一个可变参数，具体由const char *format格式化需要传的参数个数来决定

// 具体测试例子
PHP_FUNCTION(myexception) 
{
	zend_throw_error(NULL, "This is my throw Error in (%s)", "myexception");
}

写测试用例1: 不捕获，程序会中断
php -r 'myexception();'
PHP Fatal error:  Uncaught Error: This is my throw Error in (myexception) in Command line code:1
Stack trace:
#0 Command line code(1): myexception()
#1 {main}
  thrown in Command line code on line 1
此时程序已经中断了


写测试用例2: 捕获，程序不会中断
//error.php
<?php
try{
    myexception();
}catch(Error $e){
    echo $e->getMessage().PHP_EOL;
}
测试:
php error.php
输出:
This is my throw Error in (myexception)
```
