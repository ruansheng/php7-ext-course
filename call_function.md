## 调用函数
在扩展可以调用:
1. 用户在.php文件中定义的函数
2. 其他扩展中定义的函数
3. PHP内核中的函数 如 array_merge

### call_user_function
看到这个是不是很熟悉，想起PHP中的call_user_function()函数，但call_user_function在PHP内核中是一个宏
```
// Zend/zend_API.h
#define call_user_function(function_table, object, function_name, retval_ptr, param_count, params) \
	_call_user_function_ex(object, function_name, retval_ptr, param_count, params, 1)
#define call_user_function_ex(function_table, object, function_name, retval_ptr, param_count, params, no_separation, symbol_table) \
	_call_user_function_ex(object, function_name, retval_ptr, param_count, params, no_separation)
  
// Zend/end_execute_API.c 
int _call_user_function_ex(zval *object, zval *function_name, zval *retval_ptr, uint32_t param_count, zval params[], int no_separation) /* {{{ */
{
	zend_fcall_info fci;

	fci.size = sizeof(fci);
	fci.object = object ? Z_OBJ_P(object) : NULL;
	ZVAL_COPY_VALUE(&fci.function_name, function_name);
	fci.retval = retval_ptr;
	fci.param_count = param_count;
	fci.params = params;
	fci.no_separation = (zend_bool) no_separation;

	return zend_call_function(&fci, NULL);
}  
```

### 扩展中调用.php文件中定义的函数
