## 调试扩展

### 扩展中定义一个函数
```
PHP_FUNCTION(echo_hello)
{
	char *arg = NULL;
	size_t arg_len, len;
	zend_string *strg;

	if (zend_parse_parameters(ZEND_NUM_ARGS(), "s", &arg, &arg_len) == FAILURE) {
		return;
	}

	strg = strpprintf(0, "Congratulations! You have successfully modified ext/%.78s/config.m4. Module %.78s is now compiled into PHP.", "test_ext", arg);

	RETURN_STR(strg);
}
```

### 编译扩展
gcc 编译的时候加上-g参数，在PHP扩展的编译中提供了--enable-debug参数，这样在编译时会往.so文件内部加入调试符号
```
./configure --with-php-config=/usr/local/php/bin/php-config --enable-debug
```

### 查看扩展内部的符号
```
# nm /usr/local/php7/lib/php/extensions/debug-non-zts-20170718/test.so
0000000000202108 B __bss_start
0000000000202108 b completed.6344
                 w __cxa_finalize@@GLIBC_2.2.5
0000000000000a10 t deregister_tm_clones
0000000000000a80 t __do_global_dtors_aux
0000000000201d90 t __do_global_dtors_aux_fini_array_entry
0000000000201da0 d __dso_handle
0000000000201e00 d _DYNAMIC
0000000000202108 D _edata
0000000000202110 B _end
0000000000000c28 T _fini
0000000000000ac0 t frame_dummy
0000000000201d88 t __frame_dummy_init_array_entry
0000000000000e78 r __FRAME_END__
0000000000000c1a T get_module
0000000000202000 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000000970 T _init
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000201d98 d __JCR_END__
0000000000201d98 d __JCR_LIST__
                 w _Jv_RegisterClasses
000000000020210c b le_test_ext
0000000000000c40 r long_min_digits
                 U php_info_print_table_end
                 U php_info_print_table_header
                 U php_info_print_table_start
0000000000000a40 t register_tm_clones
0000000000201dc0 D test_ext_functions
0000000000202060 D test_ext_module_entry
0000000000202108 d __TMC_END__
                 U zend_parse_parameters
                 U zend_strpprintf
0000000000000af5 T zif_echo_hello
0000000000000bc3 T zm_activate_test_ext
0000000000000bd4 T zm_deactivate_test_ext
0000000000000be5 T zm_info_test_ext
0000000000000bb2 T zm_shutdown_test_ext
0000000000000ba1 T zm_startup_test_ext
```

### 写php文件调用
```
vim /root/php/echo_hello.php
<?php
<?php
echo_hello("test");
```

### 使用gdb调试
```
# gdb /usr/local/php/bin/php
(gdb)break zif_echo_hello
(gdb)run -c /usr/local/php/lib/php.ini -q /root/php/echo_hello.php
The program being debugged has been started already.
Start it from the beginning? (y or n) y

Starting program: /usr/local/php7/bin/php -c /usr/local/php7/lib/php.ini -q /root/php/echo_hello.php
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".

Breakpoint 1, zif_echo_hello (execute_data=0x7ffff5e1e090, return_value=0x7fffffffaa50) at /root/php-7.2.0/ext/test_ext/test_ext.c:56
56		char *arg = NULL;
(gdb) n
60		if (zend_parse_parameters(ZEND_NUM_ARGS(), "s", &arg, &arg_len) == FAILURE) {
(gdb) n
64		strg = strpprintf(0, "Congratulations! You have successfully modified ext/%.78s/config.m4. Module %.78s is now compiled into PHP.", "test_ext", arg);
(gdb) p strg
$2 = (zend_string *) 0x6000000000
(gdb) p *strg
Cannot access memory at address 0x6000000000
(gdb) p strg
$3 = (zend_string *) 0x6000000000
(gdb) p arg_len
$4 = 4
(gdb)
$5 = 4
```
