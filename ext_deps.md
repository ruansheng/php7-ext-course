## 扩展之间的依赖
在扩展开发，有些扩展之间存在一定的依赖关系，在依赖的扩展中要调用被依赖的扩展中的某些符号，这时在配置\运行的时候就需要检测是否存在被依赖扩展
模块依赖可以在2个时刻进行处理:
1. 配置时
2. 运行时

### 配置时依赖检查
比如mysqli扩展，依赖于mysqlnd扩展
```
if test "$PHP_MYSQLI" = "yes" || test "$PHP_MYSQLI" = "mysqlnd"; then
    PHP_ADD_EXTENSION_DEP(mysqli, mysqlnd)
    AC_DEFINE([MYSQLI_USE_MYSQLND], 1, [Whether mysqlnd is enabled])
    PHP_INSTALL_HEADERS([ext/mysqli/mysqli_mysqlnd.h])
  else
    PHP_INSTALL_HEADERS([ext/mysqli/mysqli_libmysql.h])
  fi
```

### 运行时依赖检查
在扩展中 zend_module_entry 结构体是描述一个扩展的信息的，扩展的依赖关系也是注册进此结构体的
```
struct _zend_module_entry {
  ...
	const struct _zend_module_dep *deps;
	...
};

由上可见初始化需要传一个 struct _zend_module_dep 指针变量

struct _zend_module_dep {
	const char *name;		/* module name */
	const char *rel;		/* version relationship: NULL (exists), lt|le|eq|ge|gt (to given version) */
	const char *version;	/* version */
	unsigned char type;		/* dependency type */
};

可这样初始化
zend_module_dep deps[] = {
        {"standard", NULL, NULL, MODULE_DEP_REQUIRED},
        { NULL, NULL, NULL, 0},
};

当然伟大的Zend内核还提供了人性化的处理方式
static const  zend_module_dep deps[] = {
	  ZEND_MOD_REQUIRED("standard")
	  ZEND_MOD_END
};

// 在 Zend/zend_modules.h 中
#define ZEND_MOD_REQUIRED_EX(name, rel, ver)	{ name, rel, ver, MODULE_DEP_REQUIRED  },
#define ZEND_MOD_REQUIRED(name)		ZEND_MOD_REQUIRED_EX(name, NULL, NULL)
#define ZEND_MOD_END { NULL, NULL, NULL, 0 }

// 注册依赖数组到 zend_module_entry 中
	test_module_entry.deps = deps;
或者
	zend_module_entry test_module_entry = {
		STANDARD_MODULE_HEADER,
		...
	};
	改成
	zend_module_entry test_module_entry = {
		STANDARD_MODULE_HEADER_EX, NULL,
		deps,
		...
	};
2中写法看着不同，但本质是一样的
```
