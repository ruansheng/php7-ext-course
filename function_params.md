### 函数内部获取传递的参数

这里PHP7中提供了一种高效的读取参数的方法
```
zval *data;
	
ZEND_PARSE_PARAMETERS_START(1, 1)
	Z_PARAM_ZVAL(data)
ZEND_PARSE_PARAMETERS_END();
```

PHP5/PHP7中都可以使用的方式
```
zval *data;

if (zend_parse_parameters(ZEND_NUM_ARGS(), "z", &data) == FAILURE) {
	return;
}
```

## 函数返回值
