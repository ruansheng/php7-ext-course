## 函数/方法返回值
函数或者方法的返回值都是通用的

### Example
```
# php -r 'echo test();'
```

### 返回值方式一
注意: 这种方式不建议使用
```
还记得之前创建函数中的 PHP_FUNCTION(test);
宏展开之后是 void zif_test(zend_execute_data *execute_data, zval *return_value);
这里就传入了一个 zval *return_value 指针，如果函数\方法有返回值，直接设置此指针即可
注意: 这里需要特别注意引用计数问题

PHP_FUNCTION(test)
{
   ZVAL_LONG(return_value, 10);
}

// ZVAL_LONG 宏 Zend/zend_types.h
#define ZVAL_LONG(z, l) {				\
		zval *__z = (z);				\
		Z_LVAL_P(__z) = l;				\
		Z_TYPE_INFO_P(__z) = IS_LONG;	\
	}
```

### 返回值方式二
PHP提供了很多专门用于设置返回值的宏，这些宏定义在 Zend/zend_API.h
```
#define RETURN_BOOL(b) 					{ RETVAL_BOOL(b); return; }
#define RETURN_NULL() 					{ RETVAL_NULL(); return;}
#define RETURN_LONG(l) 					{ RETVAL_LONG(l); return; }
#define RETURN_DOUBLE(d) 				{ RETVAL_DOUBLE(d); return; }
#define RETURN_STR(s) 					{ RETVAL_STR(s); return; }
#define RETURN_INTERNED_STR(s)			{ RETVAL_INTERNED_STR(s); return; }
#define RETURN_NEW_STR(s)				{ RETVAL_NEW_STR(s); return; }
#define RETURN_STR_COPY(s)				{ RETVAL_STR_COPY(s); return; }
#define RETURN_STRING(s) 				{ RETVAL_STRING(s); return; }
#define RETURN_STRINGL(s, l) 			{ RETVAL_STRINGL(s, l); return; }
#define RETURN_EMPTY_STRING() 			{ RETVAL_EMPTY_STRING(); return; }
#define RETURN_RES(r) 					{ RETVAL_RES(r); return; }
#define RETURN_ARR(r) 					{ RETVAL_ARR(r); return; }
#define RETURN_OBJ(r) 					{ RETVAL_OBJ(r); return; }
#define RETURN_ZVAL(zv, copy, dtor)		{ RETVAL_ZVAL(zv, copy, dtor); return; }
#define RETURN_FALSE  					{ RETVAL_FALSE; return; }
#define RETURN_TRUE   					{ RETVAL_TRUE; return; }

#define RETVAL_BOOL(b)					ZVAL_BOOL(return_value, b)
#define RETVAL_NULL() 					ZVAL_NULL(return_value)
#define RETVAL_LONG(l) 					ZVAL_LONG(return_value, l)
#define RETVAL_DOUBLE(d) 				ZVAL_DOUBLE(return_value, d)
#define RETVAL_STR(s)			 		ZVAL_STR(return_value, s)
#define RETVAL_INTERNED_STR(s)	 		ZVAL_INTERNED_STR(return_value, s)
#define RETVAL_NEW_STR(s)		 		ZVAL_NEW_STR(return_value, s)
#define RETVAL_STR_COPY(s)		 		ZVAL_STR_COPY(return_value, s)
#define RETVAL_STRING(s)		 		ZVAL_STRING(return_value, s)
#define RETVAL_STRINGL(s, l)		 	ZVAL_STRINGL(return_value, s, l)
#define RETVAL_EMPTY_STRING() 			ZVAL_EMPTY_STRING(return_value)
#define RETVAL_RES(r)			 		ZVAL_RES(return_value, r)
#define RETVAL_ARR(r)			 		ZVAL_ARR(return_value, r)
#define RETVAL_OBJ(r)			 		ZVAL_OBJ(return_value, r)
#define RETVAL_ZVAL(zv, copy, dtor)		ZVAL_ZVAL(return_value, zv, copy, dtor)
#define RETVAL_FALSE  					ZVAL_FALSE(return_value)
#define RETVAL_TRUE   					ZVAL_TRUE(return_value)

可见这些宏最终还是设置了return_value的值

```
