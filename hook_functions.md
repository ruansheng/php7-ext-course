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

### 钩子函数和扩展相关联
上面的是钩子函数的定义，只是定义，没有和扩展关联在一起，在PHP内核加载扩展的时候才会被依次调用对应的钩子函数
下面我们来看一下钩子函数怎么和扩展关联在一起:
```
创建好的扩展，在test.c文件中会有 zend_module_entry 类型的一个结构体被初始化
zend_module_entry test_module_entry = {
	STANDARD_MODULE_HEADER,
	"test",
	test_functions,
	PHP_MINIT(test),
	PHP_MSHUTDOWN(test),
	PHP_RINIT(test),		/* Replace with NULL if there's nothing to do at request start */
	PHP_RSHUTDOWN(test),	/* Replace with NULL if there's nothing to do at request end */
	PHP_MINFO(test),
	PHP_TEST_VERSION,
	STANDARD_MODULE_PROPERTIES
};
```

下面是 zend_module_entry 结构体的定义 在 Zend/zend_module.h 中
```
typedef struct _zend_module_entry zend_module_entry;
struct _zend_module_entry {
	unsigned short size;
	unsigned int zend_api;
	unsigned char zend_debug;
	unsigned char zts;
	const struct _zend_ini_entry *ini_entry;
	const struct _zend_module_dep *deps;
	const char *name;
	const struct _zend_function_entry *functions;
	int (*module_startup_func)(INIT_FUNC_ARGS);
	int (*module_shutdown_func)(SHUTDOWN_FUNC_ARGS);
	int (*request_startup_func)(INIT_FUNC_ARGS);
	int (*request_shutdown_func)(SHUTDOWN_FUNC_ARGS);
	void (*info_func)(ZEND_MODULE_INFO_FUNC_ARGS);
	const char *version;
	size_t globals_size;
#ifdef ZTS
	ts_rsrc_id* globals_id_ptr;
#else
	void* globals_ptr;
#endif
	void (*globals_ctor)(void *global);
	void (*globals_dtor)(void *global);
	int (*post_deactivate_func)(void);
	int module_started;
	unsigned char type;
	void *handle;
	int module_number;
	const char *build_id;
};
```

可见zend_module_entry结构体的参数特别多，在扩展中初始化zend_module_entry结构体的时候通过几个定义的宏来替代了大部分不需要关注的参数
只留了一些在扩展中需要关注的
```
// 在 Zend/zend_module.h 中
#define STANDARD_MODULE_HEADER_EX sizeof(zend_module_entry), ZEND_MODULE_API_NO, ZEND_DEBUG, USING_ZTS
#define STANDARD_MODULE_HEADER \
	STANDARD_MODULE_HEADER_EX, NULL, NULL
#define STANDARD_MODULE_PROPERTIES_EX 0, 0, NULL, 0, ZEND_MODULE_BUILD_ID
#define STANDARD_MODULE_PROPERTIES \
	NO_MODULE_GLOBALS, NULL, STANDARD_MODULE_PROPERTIES_EX
```

接下来就看一下最关键的几行函数指针赋值
```
在zend_module_entry结构体中定义的函数指针是这样:
int (*module_startup_func)(INIT_FUNC_ARGS);
int (*module_shutdown_func)(SHUTDOWN_FUNC_ARGS);
int (*request_startup_func)(INIT_FUNC_ARGS);
int (*request_shutdown_func)(SHUTDOWN_FUNC_ARGS);
void (*info_func)(ZEND_MODULE_INFO_FUNC_ARGS);

扩展中函数指针赋值也是封装了几个宏来实现的，简化了操作，看上去只需要传一个扩展名称，很直观:
PHP_MINIT(test)
PHP_MSHUTDOWN(test)
PHP_RINIT(test)
PHP_RSHUTDOWN(test)
PHP_MINFO(test)

这几个宏定义是这样的:
#define ZEND_MODULE_STARTUP_N(module)       zm_startup_##module
#define ZEND_MODULE_SHUTDOWN_N(module)		zm_shutdown_##module
#define ZEND_MODULE_ACTIVATE_N(module)		zm_activate_##module
#define ZEND_MODULE_DEACTIVATE_N(module)	zm_deactivate_##module
#define ZEND_MODULE_INFO_N(module)			zm_info_##module

这几个宏展开是这样的:
zm_startup_test
zm_shutdown_test
zm_activate_test
zm_deactivate_test
zm_info_test

我们都知道宏替换是发生在预处理阶段，其实在扩展中也可以直接用宏展开的结果来替代PHP_MINIT(test)这样的地方，只是这样不够灵活，还是推荐使用官方的方式
```
