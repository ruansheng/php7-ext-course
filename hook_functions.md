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

### PHP_MINIT_FUNCTION(test) 钩子函数
```

```


### PHP_RINIT_FUNCTION(test) 钩子函数
```

```


### PHP_RSHUTDOWN_FUNCTION(test) 钩子函数
```

```


### PHP_MSHUTDOWN_FUNCTION(test) 钩子函数
```

```


### PHP_MINFO_FUNCTION(test) 钩子函数
PHP内核提供了一系列的函数来方便控制扩展信息的输出展示，这些函数定义在 ext/standard/info.h 中
```
PHPAPI zend_string *php_info_html_esc(char *string);
PHPAPI void php_info_html_esc_write(char *string, int str_len);
PHPAPI void php_print_info_htmlhead(void);
PHPAPI void php_print_info(int flag);
PHPAPI void php_print_style(void);
PHPAPI void php_info_print_style(void);
PHPAPI void php_info_print_table_colspan_header(int num_cols, char *header);
PHPAPI void php_info_print_table_header(int num_cols, ...);
PHPAPI void php_info_print_table_row(int num_cols, ...);
PHPAPI void php_info_print_table_row_ex(int num_cols, const char *, ...);
PHPAPI void php_info_print_table_start(void);
PHPAPI void php_info_print_table_end(void);
PHPAPI void php_info_print_box_start(int bg);
PHPAPI void php_info_print_box_end(void);
PHPAPI void php_info_print_hr(void);
PHPAPI void php_info_print_module(zend_module_entry *module);
PHPAPI zend_string *php_get_uname(char mode);
```
可以看一看这里函数内部的具体实现，下面代码以 php_info_print_table_colspan_header 为例来展示
```
PHPAPI void php_info_print_table_colspan_header(int num_cols, char *header) /* {{{ */
{
	int spaces;

	if (!sapi_module.phpinfo_as_text) {
		php_info_printf("<tr class=\"h\"><th colspan=\"%d\">%s</th></tr>\n", num_cols, header );
	} else {
		spaces = (int)(74 - strlen(header));
		php_info_printf("%*s%s%*s\n", (int)(spaces/2), " ", header, (int)(spaces/2), " ");
	}
}

```
