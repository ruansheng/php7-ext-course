## 创建内部函数
这里添加一个test()方法

### Example
```
# php -r 'test("hello world!");'
```

### 声明
自定义扩展中
```
PHP_FUNCTION(test);
```

** 解析说明 **
```
// 宏展开
void zif_test(zend_execute_data *execute_data, zval *return_value);
```

### 定义
自定义扩展中
```
PHP_FUNCTION(test)
{
    // do something
}
```

** 解析说明 **
```
// main/php.h
#define PHP_FUNCTION			ZEND_FUNCTION
// Zend/zend_API.h
#define ZEND_FUNCTION(name)				ZEND_NAMED_FUNCTION(ZEND_FN(name))
#define ZEND_FN(name) zif_##name
#define ZEND_NAMED_FUNCTION(name)		void name(INTERNAL_FUNCTION_PARAMETERS)

// 宏展开的结果
void zif_test(zend_execute_data *execute_data, zval *return_value)
{
    // do something
}
```

### 传参
扩展中定义函数的参数
```
ZEND_BEGIN_ARG_INFO_EX(arginfo_test_index, 0, 0, 1)
	ZEND_ARG_INFO(0, data)
ZEND_END_ARG_INFO()
```

** 解析说明 **
```
// Zend/zend_API.h
#define ZEND_BEGIN_ARG_INFO_EX(name, _unused, return_reference, required_num_args)	\
	static const zend_internal_arg_info name[] = { \
		{ (const char*)(zend_uintptr_t)(required_num_args), 0, return_reference, 0 },

// Zend/zend_API.h
#define ZEND_ARG_INFO(pass_by_ref, name)                             { #name, 0, pass_by_ref, 0},

// Zend/zend_API.h
#define ZEND_END_ARG_INFO()		};

// zend_internal_arg_info 结构体定义  Zend/zend_compile.h
typedef struct _zend_internal_arg_info {
	const char *name;
	zend_type type;
	zend_uchar pass_by_reference;
	zend_bool is_variadic;
} zend_internal_arg_info;

// 宏展开的结果
static const zend_internal_arg_info arginfo_test_index[] = {
		{ (const char*)(zend_uintptr_t)(1), 0, 0, 0 },
    { "data", 0, 0, 0},
};

```

### 添加到扩展函数数组中
扩展中把函数添加到zend_function_entry数组中
```
const zend_function_entry test_functions[] = {
	PHP_FE(test,	arginfo_test_index)
	PHP_FE_END  // 注意: 不可省略
};

```

** 解析说明 **
```
// PHP_FE(test,	arginfo_test_index) 宏展开 Zend/zend_API.h
{ "es_dump", zif_test, arginfo_test_index, 1, 0 },

// 其中 PHP_FE_END 宏展开  Zend/zend_API.h
#define ZEND_FE_END            { NULL, NULL, NULL, 0, 0 }

// 其中 zend_function_entry 结构体定义 Zend/zend_API.h
typedef struct _zend_function_entry {
	const char *fname;  // 函数名称
	zif_handler handler; // handler实现
	const struct _zend_internal_arg_info *arg_info;  // 参数信息
	uint32_t num_args;  // 参数数目
	uint32_t flags;
} zend_function_entry;
```
