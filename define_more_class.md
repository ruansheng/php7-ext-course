## 扩展中定义多个类
这里还是以test扩展为例来说明

### 在同一个.c文件中定义多个类
```
这种方式其实就是在和定义一个类的方式一样，无非就是在同样的地方，一样的代码来两份，改个变量名，改个类名
同理可以定义2、3、4 ... 类都是一样的方式

zend_class_entry * my_test_a_ce;
zend_class_entry * my_test_b_ce;

const zend_function_entry 	test_a_methods[] = {
		//PHP_ME(TestA, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
		{NULL, NULL, NULL}
};

const zend_function_entry 	test_b_methods[] = {
		//PHP_ME(TestB, __construct, NULL, ZEND_ACC_PUBLIC|ZEND_ACC_CTOR)
		{NULL, NULL, NULL}
};

PHP_MINIT_FUNCTION(test_ext)
{
	zend_class_entry ce_a;
	INIT_CLASS_ENTRY(ce_a, "TestA", test_a_methods);
	my_test_a_ce = zend_register_internal_class(&ce_a);

	zend_class_entry ce_b;
	INIT_CLASS_ENTRY(ce_b, "TestB", test_b_methods);
	my_test_b_ce = zend_register_internal_class(&ce_b);

	return SUCCESS;
}

// 测试
php -r 'var_dump(new TestA);var_dump(new TestB());'
输出:
object(TestA)#1 (0) {
}
object(TestB)#1 (0) {
}

```

### 更优雅的定义方式
```
// 在php_test.h 中定义几个宏，用来实现多文件关联
#define TEST_STARTUP_FUNCTION(module) ZEND_MINIT_FUNCTION(pmc_##module)
#define TEST_STARTUP(module) ZEND_MODULE_STARTUP_N(pmc_##module)(INIT_FUNC_ARGS_PASSTHRU)
#define TEST_RINIT_FUNCTION(module) ZEND_RINIT_FUNCTION(pmc_##module)
```

```
// 创建第一个类的test_util.h文件
#ifndef TEST_UTIL_H
#define TEST_UTIL_H
extern zend_class_entry *test_util_ce;
TEST_STARTUP_FUNCTION(util);
#endif //TEST_UTIL_H

// 创建第一个类的test_util.c文件
#ifdef HAVE_CONFIG_H
#include "config.h"
#endif

#include "php.h"

#include "php_test.h"
#include "test_util.h"

zend_class_entry *test_util_ce;

PHP_METHOD(test_util, show);

/** {{{ test_util_methods
*/
zend_function_entry test_util_methods[] = {
        PHP_ME(test_util, show, NULL, ZEND_ACC_PUBLIC)
        {NULL, NULL, NULL}
};
/* }}} */

/** {{{ proto public TestUtil::show()
*/
PHP_METHOD(test_util, show) {
    php_printf("hello util!");
}
/* }}} */

PMC_STARTUP_FUNCTION(util){
        zend_class_entry ce;

        INIT_CLASS_ENTRY(ce, "TestUtil", test_util_methods);
        test_util_ce = zend_register_internal_class(&ce);

        return SUCCESS;
}
```
上面第一个类就和扩展主文件分离了

```
// 在php_test.c 中引入test_util.h文件
#include "test_util.h"
```

然后只需要在扩展主文件test.c中通过前面定义的宏来加载这个分离的类
```
PHP_MINIT_FUNCTION(pmc)
{
	TEST_STARTUP(util);

	return SUCCESS;
}
```

最后需要注意的是: 如果一个扩展中使用了多个.c文件，则需要在config.m4中做相应的修改
```
PHP_NEW_EXTENSION(pmc,
    test.c \
    test_util.c,
    $ext_shared,, -DZEND_ENABLE_STATIC_TSRMLS_CACHE=1)
    
// 这里如果有多个.c文件，则需要都加进去，不加进去在运行时则会报错
// .c文件直接以空格分隔
```
需要多个类的话依葫芦画瓢去添加对应的.h、.c文件就可以了
