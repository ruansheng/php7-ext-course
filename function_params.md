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


### PHP7新增的高效的解析参数方式
```
zval *data;
	
ZEND_PARSE_PARAMETERS_START(1, 1)
	Z_PARAM_ZVAL(data)
ZEND_PARSE_PARAMETERS_END();
```
