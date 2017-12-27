## 全局资源

### 声明全局资源的结构体
全局资源的用处: 用来存储解析的配置项的值
```
// 在test_php.h 文件中 有被注释起来的全局资源定义的代码
/*
Declare any global variables you may need between the BEGIN
and END macros here:

ZEND_BEGIN_MODULE_GLOBALS(test)
	zend_long  global_value;
	char *global_string;
ZEND_END_MODULE_GLOBALS(test)
*/

// 在 Zend/zend_API.h 中
#define ZEND_BEGIN_MODULE_GLOBALS(module_name)		\
	typedef struct _zend_##module_name##_globals {
#define ZEND_END_MODULE_GLOBALS(module_name)		\
	} zend_##module_name##_globals;

宏展开结果:
typedef struct _zend_test_globals {
	zend_long  global_value;
	char *global_string;
} zend_test_globals;

// 由上可见是定义了一个全局结构体变量来存储从php.ini中解析出来的内容
```

### 定义全局资源结构体
```
// 在test.c 文件中 有被注释起来的全局资源定义的代码
/* If you declare any globals in php_test_ext.h uncomment this:
ZEND_DECLARE_MODULE_GLOBALS(test)
*/

// 在 Zend/zend_API.h 中
#define ZEND_DECLARE_MODULE_GLOBALS(module_name)							\
	zend_##module_name##_globals module_name##_globals;

宏展开结果:
zend_test_globals test_globals;

// 这里可见是在.c文件中定义一个存储全局资源的结构体变量
```

### 访问全局资源结构体中的成员
定义了结构体，那怎么来访问这个结构体中成员是一个问题，如果是直接用 test_globals.global_value 访问，那当然是可以了
```
这里PHP内核做了一些像EG、CG 的访问全局资源的宏

// 在Zend/zend_API.h 中
#define ZEND_MODULE_GLOBALS_ACCESSOR(module_name, v) ZEND_TSRMG(module_name##_globals_id, zend_##module_name##_globals *, v)

// 这里可以在我们扩展中再做一层封装
#define TEST_G(v) ZEND_MODULE_GLOBALS_ACCESSOR(module_name, v)

接下来可以在扩展中通过 TEST_G(global_value)、TEST_G(global_string) 来对结构体成员进行读写
```

