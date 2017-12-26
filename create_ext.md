## 创建一个扩展
这里新建一个名字叫test扩展

### PHP7代码项目目录结构
```
php-7.2.0
    TSRM
    Zend
    appveyor
    build
    ext
    	bcmath
	ba2
	...
    main
    pear
    sapi
    scripts
    tests
    travis
    win32
    ...
```

### 创建扩展工具
```
# cd ext
# ./ext_skel
./ext_skel --extname=module [--proto=file] [--stubs=file] [--xml[=file]]
           [--skel=dir] [--full-xml] [--no-help]

  --extname=module   module is the name of your extension
  --proto=file       file contains prototypes of functions to create
  --stubs=file       generate only function stubs in file
  --xml              generate xml documentation to be added to phpdoc-svn
  --skel=dir         path to the skeleton directory
  --full-xml         generate xml documentation for a self-contained extension
                     (not yet implemented)
  --no-help          don't try to be nice and create comments in the code
                     and helper functions to test if the module compiled
```
这个工具有几个很有意思的参数，来看一下这些参数都是干啥用的:
```
--extname=module  待创建扩展的名称，module是一个全为小写字母的标识符，仅包含字母和下划线，在 PHP 发行包的 ext/ 文件夹下是唯一的
--proto=file 指定一个函数描述文件，可以指定一个C语音的.h文件，这样创建的扩展中就自动生成了描述文件中指定的函数		
	     Example: 
	     	 echo 'int my_sum(int a,int b);' >> test.h
	     	./ext_skel --extname=mytest --proto=./test.h
		这样就可以看到扩展的.c文件中自动创建了一个函数
		/* {{{ proto int my_sum(int a, int b)
		   ; */
		PHP_FUNCTION(my_sum)
		{
			int argc = ZEND_NUM_ARGS();
			zend_long a;
			zend_long b;

			if (zend_parse_parameters(argc, "ll", &a, &b) == FAILURE) 
				return;

			php_error(E_WARNING, "my_sum: not yet implemented");
		}
		/* }}} */
--skel=dir  设置生成的扩展项目存储的位置，不指定该项则默认在ext/module中	
--no-help  指定生成的扩展项目的文件中不包含注释，否则会生成很多注释说明 		
```

### 创建扩展目录
```
# ./ext_skel --extname=test
// 创建之后在ext目录下生成一个项目目录结构
CREDITS
EXPERIMENTAL
test.c
test.php
README.md
config.m4
config.w32
php_test.h
tests
    001.phpt
```

### 修改config.m4
```
del PHP_ARG_ENABLE(test, whether to enable test support,
del Make sure that the comment is aligned:
del [  --enable-test           Enable test support])

改为

PHP_ARG_ENABLE(test, whether to enable test support,
del Make sure that the comment is aligned:
[  --enable-test           Enable test support])

注意: del 是config.m4文件的注释符号，就好比C语言中的//注释符号
```

### 修改 test.c
```
PHP_FUNCTION(confirm_php_hello_compiled)
{
	php_printf("hello world!\n");
	return;
}
```

### 编译扩展
```
cd ext/test
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make
make install  
```

### php.ini配置文件中加载扩展的动态链接库
```
vim php.ini
添加: extension=test.so
```

### 检测扩展是否加载成功
```
# php -m | grep test
```

### 运行测试
```
# php -r 'confirm_php_hello_compiled()'
```
