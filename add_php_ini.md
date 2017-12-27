## 为扩展新增php.ini配置项
php.ini配置文件的规则是:
1. 以';'开始的行会被忽略
2. [xxx]的行也被忽略
3. 配置标识符大小写敏感
4. 配置项以'.'来区分为不同的节
5. 配置项的值可以是 数字、字符串、PHP常量、位运算表达式

### 扩展解析配置项
这里以test扩展为例来说明
```
新创建的扩展，会有一些关于php.ini配置项的代码被注释起来
/* Remove comments and fill if you need to have entries in php.ini
PHP_INI_BEGIN()
    STD_PHP_INI_ENTRY("test.global_value",      "42", PHP_INI_ALL, OnUpdateLong, global_value, zend_test_ext_globals, test_ext_globals)
    STD_PHP_INI_ENTRY("test.global_string", "foobar", PHP_INI_ALL, OnUpdateString, global_string, zend_test_ext_globals, test_ext_globals)
PHP_INI_END()
*/
这里是描述当前扩展需要解析配置项的配置
PHP_INI_BEGIN()
  ...
PHP_INI_END()

// 在Zend/zend_ini.h 中
#define ZEND_INI_BEGIN()		static const zend_ini_entry_def ini_entries[] = {
#define ZEND_INI_END()		{ NULL, NULL, NULL, NULL, NULL, NULL, NULL, 0, 0, 0} };

宏展开结果:
static const zend_ini_entry_def ini_entries[] = {
    { NULL, NULL, NULL, NULL, NULL, NULL, NULL, 0, 0, 0} 
};

可以看出其实也是 zend_ini_entry_def 结构体变量数组的初始化赋值

// 中间的内容就是设置要解析配置项的描述，通过 STD_PHP_INI_ENTRY()宏定义
// 在Zend/zend_ini.h 中
#define STD_ZEND_INI_ENTRY(name, default_value, modifiable, on_modify, property_name, struct_type, struct_ptr) \
	ZEND_INI_ENTRY2(name, default_value, modifiable, on_modify, (void *) XtOffsetOf(struct_type, property_name), (void *) &struct_ptr)

#define ZEND_INI_ENTRY2(name, default_value, modifiable, on_modify, arg1, arg2) \
	ZEND_INI_ENTRY2_EX(name, default_value, modifiable, on_modify, arg1, arg2, NULL)

#define ZEND_INI_ENTRY2_EX(name, default_value, modifiable, on_modify, arg1, arg2, displayer) \
	ZEND_INI_ENTRY3_EX(name, default_value, modifiable, on_modify, arg1, arg2, NULL, displayer)
  
#define ZEND_INI_ENTRY3_EX(name, default_value, modifiable, on_modify, arg1, arg2, arg3, displayer) \
	{ name, on_modify, arg1, arg2, arg3, default_value, displayer, modifiable, sizeof(name)-1, sizeof(default_value)-1 },  

// 参数说明
STD_ZEND_INI_ENTRY(name, default_value, modifiable, on_modify, property_name, struct_type, struct_ptr)
1. name php.ini中的配置项，例如: "test.count"
2. default_value 默认值，无论转化后是什么类型，这里必须是字符串，例如: "32" "test"
3. modifiable 可修改等级，取值有:
  ZEND_INI_USER  可以在php脚本中设置
  ZEND_INI_SYSTEM 可以在php.ini、httpd.conf配置文件中设置
  ZEND_INI_PERDIR 可以在php.ini、.htaccess、httpd.conf中设置
  ZEND_INI_ALL 表示上面三中都可以
4. on_modify 解析配置项的函数，当发现配置项的时候会调用，默认提供了5个调用函数，也可以自己定义函数来处理
  OnUpdateBool、OnUpdateLong、OnUpdateLongGEZero、OnUpdateReal、OnUpdateString、OnUpdateStringUnempty
5. property_name 要解析到的struct_type结构中的成员
6. struct_type 解析到的结构类型
7. struct_ptr 解析到的结构的地址

```

### 扩展注册配置项
上面的步骤只是初始化了 zend_ini_entry_def 结构体数组，只是初始化了，没有注册到扩展中是不能用的
注册到扩展是在模块初始化阶段完成的，接下来看一下怎么注册:
```
在 PHP_MINIT_FUNCTION(test) 中也有被注释起来的代码
/* If you have INI entries, uncomment these lines
	REGISTER_INI_ENTRIES();
	*/
  
// 在 Zend/zend_ini.h 中  
#define REGISTER_INI_ENTRIES() zend_register_ini_entries(ini_entries, module_number)  
  
// 在 Zend/zend_ini.c 中  
ZEND_API int zend_register_ini_entries(const zend_ini_entry_def *ini_entry, int module_number) /* {{{ */
{
	zend_ini_entry *p;
	zval *default_value;
	HashTable *directives = registered_zend_ini_directives;

#ifdef ZTS
	if (directives != EG(ini_directives)) {
		directives = EG(ini_directives);
	}
#endif

	while (ini_entry->name) {
		p = pemalloc(sizeof(zend_ini_entry), 1);
		p->name = zend_string_init(ini_entry->name, ini_entry->name_length, 1);
		p->on_modify = ini_entry->on_modify;
		p->mh_arg1 = ini_entry->mh_arg1;
		p->mh_arg2 = ini_entry->mh_arg2;
		p->mh_arg3 = ini_entry->mh_arg3;
		p->value = NULL;
		p->orig_value = NULL;
		p->displayer = ini_entry->displayer;
		p->modifiable = ini_entry->modifiable;

		p->orig_modifiable = 0;
		p->modified = 0;
		p->module_number = module_number;

		if (zend_hash_add_ptr(directives, p->name, (void*)p) == NULL) {
			if (p->name) {
				zend_string_release(p->name);
			}
			zend_unregister_ini_entries(module_number);
			return FAILURE;
		}
		if (((default_value = zend_get_configuration_directive(p->name)) != NULL) &&
            (!p->on_modify || p->on_modify(p, Z_STR_P(default_value), p->mh_arg1, p->mh_arg2, p->mh_arg3, ZEND_INI_STAGE_STARTUP) == SUCCESS)) {

			p->value = zend_string_copy(Z_STR_P(default_value));
		} else {
			p->value = ini_entry->value ?
				zend_string_init(ini_entry->value, ini_entry->value_length, 1) : NULL;

			if (p->on_modify) {
        // 调用定义的handler处理
				p->on_modify(p, p->value, p->mh_arg1, p->mh_arg2, p->mh_arg3, ZEND_INI_STAGE_STARTUP);
			}
		}
		ini_entry++;
	}
	return SUCCESS;
} 
 
去掉注释之后就可以了:
PHP_MINIT_FUNCTION(test)
{
   REGISTER_INI_ENTRIES();
   return SUCCESS;
}

到这来，php.ini的解析就已经完成了
```
