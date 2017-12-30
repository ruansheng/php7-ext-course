## 类成员变量
类成员变量的修饰符有:
1. public  默认的 被修饰的成员能被外部代码访问和操作
2. private 对于类内部所有成员都可见，没有访问限制。对类外部不允许访问
3. protected 被声明为protected的成员，只允许该类的子类进行访问
4. static 静态成员变量属于静态存储方式，其存储空间为内存中的静态数据区，静态变量仅在局部函数域中存在

Zend提供了各种数据类型的函数来对类成员属性进行操作

### 定义类成员变量
// Zend/zend_API.h
```
ZEND_API int zend_declare_property(zend_class_entry *ce, const char *name, size_t name_length, zval *property, int access_type);
ZEND_API int zend_declare_property_null(zend_class_entry *ce, const char *name, size_t name_length, int access_type);
ZEND_API int zend_declare_property_bool(zend_class_entry *ce, const char *name, size_t name_length, zend_long value, int access_type);
ZEND_API int zend_declare_property_long(zend_class_entry *ce, const char *name, size_t name_length, zend_long value, int access_type);
ZEND_API int zend_declare_property_double(zend_class_entry *ce, const char *name, size_t name_length, double value, int access_type);
ZEND_API int zend_declare_property_string(zend_class_entry *ce, const char *name, size_t name_length, const char *value, int access_type);
ZEND_API int zend_declare_property_stringl(zend_class_entry *ce, const char *name, size_t name_length, const char *value, size_t value_len, int access_type);

// 定义普通成员和静态成员都使用 zend_declare_property* 函数来实现，最后一个参数传递的是成员变量的类型，请看类型的枚举定义:
#define ZEND_ACC_PUBLIC		0x100
#define ZEND_ACC_PROTECTED	0x200
#define ZEND_ACC_PRIVATE	0x400
#define ZEND_ACC_STATIC			0x01

// 看具体使用
zend_declare_property_string(test_ce, "message", sizeof("message") - 1, "", ZEND_ACC_PUBLIC);
zend_declare_property_long(test_ce, "timeout", sizeof("timeout") - 1, 1L, ZEND_ACC_PUBLIC|ZEND_ACC_STATIC);
```

### 读取类成员变量
// Zend/zend_API.h
```
// 读取普通成员变量
ZEND_API zval *zend_read_property_ex(zend_class_entry *scope, zval *object, zend_string *name, zend_bool silent, zval *rv);
ZEND_API zval *zend_read_property(zend_class_entry *scope, zval *object, const char *name, size_t name_length, zend_bool silent, zval *rv);

// 读取静态成员变量
ZEND_API zval *zend_read_static_property(zend_class_entry *scope, const char *name, size_t name_length, zend_bool silent);

// 看具体使用
zval *message;
message = zend_read_property(test_ce, getThis(), "message", sizeof("message") -1, 0, message);

zval *timeout;
timeout = zend_read_static_property(test_ce, "timeout", sizeof("timeout") -1, 0);
```

### 更新类成员变量
// Zend/zend_API.h
```
ZEND_API void zend_update_property_ex(zend_class_entry *scope, zval *object, zend_string *name, zval *value);
ZEND_API void zend_update_property(zend_class_entry *scope, zval *object, const char *name, size_t name_length, zval *value);
ZEND_API void zend_update_property_null(zend_class_entry *scope, zval *object, const char *name, size_t name_length);
ZEND_API void zend_update_property_bool(zend_class_entry *scope, zval *object, const char *name, size_t name_length, zend_long value);
ZEND_API void zend_update_property_long(zend_class_entry *scope, zval *object, const char *name, size_t name_length, zend_long value);
ZEND_API void zend_update_property_double(zend_class_entry *scope, zval *object, const char *name, size_t name_length, double value);
ZEND_API void zend_update_property_str(zend_class_entry *scope, zval *object, const char *name, size_t name_length, zend_string *value);
ZEND_API void zend_update_property_string(zend_class_entry *scope, zval *object, const char *name, size_t name_length, const char *value);
ZEND_API void zend_update_property_stringl(zend_class_entry *scope, zval *object, const char *name, size_t name_length, const char *value, size_t value_length);
ZEND_API void zend_unset_property(zend_class_entry *scope, zval *object, const char *name, size_t name_length);

ZEND_API int zend_update_static_property(zend_class_entry *scope, const char *name, size_t name_length, zval *value);
ZEND_API int zend_update_static_property_null(zend_class_entry *scope, const char *name, size_t name_length);
ZEND_API int zend_update_static_property_bool(zend_class_entry *scope, const char *name, size_t name_length, zend_long value);
ZEND_API int zend_update_static_property_long(zend_class_entry *scope, const char *name, size_t name_length, zend_long value);
ZEND_API int zend_update_static_property_double(zend_class_entry *scope, const char *name, size_t name_length, double value);
ZEND_API int zend_update_static_property_string(zend_class_entry *scope, const char *name, size_t name_length, const char *value);
ZEND_API int zend_update_static_property_stringl(zend_class_entry *scope, const char *name, size_t name_length, const char *value, size_t value_length);

// 看具体使用
end_update_property_string(test_ce,  getThis(), "message", sizeof("message") - 1, "error");

zval *seconds;
....
zend_update_static_property_long(test_ce, "timeout", sizeof("timeout") - 1, Z_LVAL_P(seconds));
```
