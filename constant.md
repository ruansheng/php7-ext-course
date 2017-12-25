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
```

### 在扩展中注册常量的用法
扩展中的常量在模块初始化钩子函数中注册，Example:
```
PHP_MINIT_FUNCTION(test)
{
  REGISTER_LONG_CONSTANT("TEST_VERSION", 100, CONST_CS | CONST_PERSISTENT)
}
```
