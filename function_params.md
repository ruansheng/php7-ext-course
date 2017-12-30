## 函数内部获取传递的参数
这里解析test($data)中传的参数

### Example
```
# php -r 'test("hello world!");'
```

### PHP5/PHP7中通用的解析参数方式
```
zval *data;

if (zend_parse_parameters(ZEND_NUM_ARGS(), "z", &data) == FAILURE) {
	return;
}
```

** 解析说明 **
先来回顾一下定义函数的函数原型
```
PHP_FUNCTION(test);
宏替换结果
void zif_test(zend_execute_data *execute_data, zval *return_value);
```
再看看 zend_execute_data 结构体的定义
```
typedef struct _zend_execute_data    zend_execute_data;
struct _zend_execute_data {
	const zend_op       *opline;           /* executed opline                */
	zend_execute_data   *call;             /* current call                   */
	zval                *return_value;
	zend_function       *func;             /* executed function              */
	zval                 This;             /* this + call_info + num_args    */
	zend_execute_data   *prev_execute_data;
	zend_array          *symbol_table;
#if ZEND_EX_USE_RUN_TIME_CACHE
	void               **run_time_cache;   /* cache op_array->run_time_cache */
#endif
#if ZEND_EX_USE_LITERALS
	zval                *literals;         /* cache op_array->literals       */
#endif
};
```
函数参数解析
```
#define ZEND_NUM_ARGS()		EX_NUM_ARGS()
#define EX_NUM_ARGS()		ZEND_CALL_NUM_ARGS(execute_data)   
#define ZEND_CALL_NUM_ARGS(call)    (call)->This.u2.num_args

ZEND_API int zend_parse_parameters(int num_args, const char *type_spec, ...) /* {{{ */
{
	va_list va;
	int retval;
	int flags = 0;

	va_start(va, type_spec);
	retval = zend_parse_va_args(num_args, type_spec, &va, flags);
	va_end(va);

	return retval;
}

static int zend_parse_va_args(int num_args, const char *type_spec, va_list *va, int flags) /* {{{ */
{
	...
	for (spec_walk = type_spec; *spec_walk; spec_walk++) {
		c = *spec_walk;
		switch (c) {
			case 'l': case 'd':
			case 's': case 'b':
			case 'r': case 'a':
			case 'o': case 'O':
			case 'z': case 'Z':
			...
			...
		}
	}
	...
	// 获取实际传参数
	arg_count = ZEND_CALL_NUM_ARGS(EG(current_execute_data));
	...
	while (num_args-- > 0) {
		...
		// 逐个解析参数
		arg = ZEND_CALL_ARG(EG(current_execute_data), i + 1);
		'''
	}
	...
}

由上面解析参数的函数中可以看出:
在调用每个函数的时候，函数参数是由内核统一解析，然后统一存储在 EG(current_execute_data) 中
然后解析执行到这个函数的时候会把execute_data传递到函数中，解析参数也就是从execute_data中解析
```

### PHP7新增的高效的解析参数方式
```
zval *data;
	
ZEND_PARSE_PARAMETERS_START(1, 1)
	Z_PARAM_ZVAL(data)
ZEND_PARSE_PARAMETERS_END();
```
先看一下宏定义
```
#define ZEND_PARSE_PARAMETERS_START(min_num_args, max_num_args) \
	ZEND_PARSE_PARAMETERS_START_EX(0, min_num_args, max_num_args)

#define ZEND_PARSE_PARAMETERS_START_EX(flags, min_num_args, max_num_args) do { \
		const int _flags = (flags); \
		int _min_num_args = (min_num_args); \
		int _max_num_args = (max_num_args); \
		int _num_args = EX_NUM_ARGS(); \
		int _i; \
		zval *_real_arg, *_arg = NULL; \
		zend_expected_type _expected_type = IS_UNDEF; \
		char *_error = NULL; \
		zend_bool _dummy; \
		zend_bool _optional = 0; \
		int error_code = ZPP_ERROR_OK; \
		((void)_i); \
		((void)_real_arg); \
		((void)_arg); \
		((void)_expected_type); \
		((void)_error); \
		((void)_dummy); \
		((void)_optional); \
		\
		do { \
			if (UNEXPECTED(_num_args < _min_num_args) || \
			    (UNEXPECTED(_num_args > _max_num_args) && \
			     EXPECTED(_max_num_args >= 0))) { \
				if (!(_flags & ZEND_PARSE_PARAMS_QUIET)) { \
					zend_wrong_parameters_count_error(_flags & ZEND_PARSE_PARAMS_THROW, _num_args, _min_num_args, _max_num_args); \
				} \
				error_code = ZPP_ERROR_FAILURE; \
				break; \
			} \
			_i = 0; \
			_real_arg = ZEND_CALL_ARG(execute_data, 0);

#define ZEND_PARSE_PARAMETERS_END_EX(failure) \
		} while (0); \
		if (UNEXPECTED(error_code != ZPP_ERROR_OK)) { \
			if (!(_flags & ZEND_PARSE_PARAMS_QUIET)) { \
				if (error_code == ZPP_ERROR_WRONG_CALLBACK) { \
					zend_wrong_callback_error(_flags & ZEND_PARSE_PARAMS_THROW, E_WARNING, _i, _error); \
				} else if (error_code == ZPP_ERROR_WRONG_CLASS) { \
					zend_wrong_parameter_class_error(_flags & ZEND_PARSE_PARAMS_THROW, _i, _error, _arg); \
				} else if (error_code == ZPP_ERROR_WRONG_ARG) { \
					zend_wrong_parameter_type_error(_flags & ZEND_PARSE_PARAMS_THROW, _i, _expected_type, _arg); \
				} \
			} \
			failure; \
		} \
	} while (0)

#define ZEND_PARSE_PARAMETERS_END() \
	ZEND_PARSE_PARAMETERS_END_EX(return)
	
// 由上可以看出也是从 ZEND_CALL_ARG(execute_data, 0) 里面解析参数的
```
