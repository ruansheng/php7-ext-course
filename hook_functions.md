## 扩展中的钩子函数
这里以test扩展为例

### 钩子函数
创建好的test扩展，test.c文件中可以看到如下钩子函数的定义代码
```
// 模块初始化的时候调用
PHP_MINIT_FUNCTION(test)
{
	return SUCCESS;
}

// 请求初始化的时候调用
PHP_RINIT_FUNCTION(test)
{
#if defined(COMPILE_DL_TEST) && defined(ZTS)
	ZEND_TSRMLS_CACHE_UPDATE();
#endif
	return SUCCESS;
}

// 请求结束关闭的时候调用
PHP_RSHUTDOWN_FUNCTION(test)
{
	return SUCCESS;
}

// 模块结束关闭的时候调用
PHP_MSHUTDOWN_FUNCTION(test)
{
  return SUCCESS;
}

// 在phpinfo() 或 php-i 的时候调用
PHP_MINFO_FUNCTION(test)
{
	php_info_print_table_start();
	php_info_print_table_header(2, "test support", "enabled");
	php_info_print_table_end();
}

```
