## 定义全局常量

### 注册常量的宏
PHP提供了很多对于不同类型常量注册的宏，宏定义在Zend/zend_constants.h中
```
// 注册常量时只会绑定到当前模块，一旦注册这个模块的常量从内存中卸载，那么这个常量也就会随即消逝
#define REGISTER_NULL_CONSTANT(name, flags)  zend_register_null_constant((name), sizeof(name)-1, (flags), module_number)
#define REGISTER_BOOL_CONSTANT(name, bval, flags)  zend_register_bool_constant((name), sizeof(name)-1, (bval), (flags), module_number)
#define REGISTER_LONG_CONSTANT(name, lval, flags)  zend_register_long_constant((name), sizeof(name)-1, (lval), (flags), module_number)
#define REGISTER_DOUBLE_CONSTANT(name, dval, flags)  zend_register_double_constant((name), sizeof(name)-1, (dval), (flags), module_number)
#define REGISTER_STRING_CONSTANT(name, str, flags)  zend_register_string_constant((name), sizeof(name)-1, (str), (flags), module_number)
#define REGISTER_STRINGL_CONSTANT(name, str, len, flags)  zend_register_stringl_constant((name), sizeof(name)-1, (str), (len), (flags), module_number)

// 注册带命名空间的常量
#define REGISTER_NS_NULL_CONSTANT(ns, name, flags)  zend_register_null_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (flags), module_number)
#define REGISTER_NS_BOOL_CONSTANT(ns, name, bval, flags)  zend_register_bool_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (bval), (flags), module_number)
#define REGISTER_NS_LONG_CONSTANT(ns, name, lval, flags)  zend_register_long_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (lval), (flags), module_number)
#define REGISTER_NS_DOUBLE_CONSTANT(ns, name, dval, flags)  zend_register_double_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (dval), (flags), module_number)
#define REGISTER_NS_STRING_CONSTANT(ns, name, str, flags)  zend_register_string_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (str), (flags), module_number)
#define REGISTER_NS_STRINGL_CONSTANT(ns, name, str, len, flags)  zend_register_stringl_constant(ZEND_NS_NAME(ns, name), sizeof(ZEND_NS_NAME(ns, name))-1, (str), (len), (flags), module_number)

// 注册的变量将会独立于该模块，始终保存在符号表中
#define REGISTER_MAIN_NULL_CONSTANT(name, flags)  zend_register_null_constant((name), sizeof(name)-1, (flags), 0)
#define REGISTER_MAIN_BOOL_CONSTANT(name, bval, flags)  zend_register_bool_constant((name), sizeof(name)-1, (bval), (flags), 0)
#define REGISTER_MAIN_LONG_CONSTANT(name, lval, flags)  zend_register_long_constant((name), sizeof(name)-1, (lval), (flags), 0)
#define REGISTER_MAIN_DOUBLE_CONSTANT(name, dval, flags)  zend_register_double_constant((name), sizeof(name)-1, (dval), (flags), 0)
#define REGISTER_MAIN_STRING_CONSTANT(name, str, flags)  zend_register_string_constant((name), sizeof(name)-1, (str), (flags), 0)
#define REGISTER_MAIN_STRINGL_CONSTANT(name, str, len, flags)  zend_register_stringl_constant((name), sizeof(name)-1, (str), (len), (flags), 0)

#define CONST_CS				(1<<0)				/* Case Sensitive */
#define CONST_PERSISTENT		(1<<1)				/* Persistent */
```

### 在扩展中注册常量
扩展中的常量在模块初始化钩子函数中注册，Example:
```
PHP_MINIT_FUNCTION(test)
{
  REGISTER_LONG_CONSTANT("TEST_VERSION", 100, CONST_CS | CONST_PERSISTENT);
}
```

### 宏展开
```
int zm_startup_test(int type, int module_number)
{
  zend_register_long_constant("TEST_VERSION", sizeof("TEST_VERSION")-1, 1, ((1<<0) | (1<<1)), module_number)	
}
```

### 注册常量究竟做了什么
```
ZEND_API void zend_register_long_constant(const char *name, size_t name_len, zend_long lval, int flags, int module_number)
{
	zend_constant c;

	ZVAL_LONG(&c.value, lval);
	c.flags = flags;
	c.name = zend_string_init(name, name_len, flags & CONST_PERSISTENT);
	c.module_number = module_number;
	zend_register_constant(&c);
}

ZEND_API int zend_register_constant(zend_constant *c)
{
   ...
   zend_hash_add_constant(EG(zend_constants), name, c)
   ...
}
// 可见注册常量其实定义一个zend_constant变量，添加到 EG(zend_constants) 中
```


### 常量的其他操作
```
// 看上面的宏只涉及到null、bool、long、double、string 类型常量的注册，如果要注册其他类型的常量，可以使用:
ZEND_API int zend_register_constant(zend_constant *c);

// 获取其他扩展或内涵中注册的常量值 通过zend_string
ZEND_API zval *zend_get_constant(zend_string *name);

// 获取其他扩展或内涵中注册的常量值 通过chan *
ZEND_API zval *zend_get_constant_str(const char *name, size_t name_len);
```



