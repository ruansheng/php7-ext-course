## 扩展内存泄露检查
内存泄漏（Memory Leak）是指程序中己动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果。
PHP是由C语言编写，PHP扩展中涉及到很多变量的操作都是对C语言代码的编写，一个变量的初始化从堆上分配内存用的malloc，如果只分配了内存，没有与之对应的free，
那就会导致内存泄露

### 内存泄露的影响
```
单次内存泄露或许不会产生什么影响，但是在常见的项目中可就不是一次了，那是成千上万次，服务器的内存是有限的，如果服务器内存被过度泄露到一定程度，触发到系统OOM，那后果可就不简单了
```

### PHP开启内存泄露报告
```
PHP源码编译的时候可以添加一个选项 --enable-debug，这样可以方便用GDB调试PHP内核代码
PHP扩展编译可以添加选项--enable-debug，这样可以方便GDB调试扩展代码

上面两个都开启之后，在执行PHP代码的时候，如果发现扩展内存有泄露的地方，会输出提示
值得注意的是不同的PHP执行模式，输出的信息会在不同的地方，在"错误日志"或者"标准输出"中
```

### 内存泄露的例子
```
PHP_FUNCTION(mypow)
{
	zval name;
	ZVAL_STRING(&name, "pow");

  RETURN_TRUE;
}

测试:
php -r 'var_dump(mypow());'
输出:
bool(true)
[Sat Dec 30 00:58:34 2017]  Script:  'Standard input code'
/usr/local/php7/include/php/Zend/zend_string.h(134) :  Freeing 0x00007fcbfd401800 (32 bytes), script=Standard input code
=== Total 1 memory leaks detected ===
```

### 修复内存泄露
```
PHP_FUNCTION(mypow)
{
	zval name;
	ZVAL_STRING(&name, "pow");

	zval_ptr_dtor(&call_func_name);
	ZVAL_NULL(&call_func_name);
	
  RETURN_TRUE;
}

测试:
php -r 'var_dump(mypow());'
输出:
bool(true)
```
