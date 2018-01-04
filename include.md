## 扩展中加载执行php文件
在PHP中提供了包含文件的语法: include、include_once、require、require_one ,下面看一下每个语句的作用与特点
include : 包含并运行指定文件，未找到文件则 include 结构会发出一条警告，脚本会继续运行
include_once: 功能和include相同，唯一区别是 PHP 会检查该文件是否已经被包含过，如果是则不会再次包含。
require : 包含并运行指定文件，基本同include，只是在出错时产生 E_COMPILE_ERROR 级别的错误，将导致脚本中止
require_once : 功能和require相同，唯一区别是 PHP 会检查该文件是否已经被包含过，如果是则不会再次包含。

### PHP7内核执行PHP文件的流程
```
PHP执行一个.php文件的流程:
读取文件->词法分析->语法分析->抽象语法树->编译成opcode->执行opcode

包含文件的原理:
include之类的包含文件也是一样的原理，一个大的项目中，一次执行流程可能包含多个文件，依次按照顺序依次处理每个被包含的文件
对于include_once这样的语句，因为其检查了重复包含，可想而知就是在一次执行流程中，往一个全局变量中保存了这次执行流程中包含过了所有文件
在重复包含的时候就从这个全局变量中判断是否重复包含
```

### 存储执行流程包含的所有文件
```
Zend引擎在执行Opcode的时候，需要记录一些执行过程中的状态。如，当前执行的类作用域，当前已经加载了那些文件，等；这些数据就保存在EG中。
EG的含义是 executor_globals。Zend执行器相关的全局变量

ZEND_API zend_executor_globals executor_globals;

# define EG(v) (executor_globals.v)

typedef struct _zend_executor_globals zend_executor_globals;

struct _zend_executor_globals {
	zval uninitialized_zval;
	zval error_zval;

	/* symbol table cache */
	zend_array *symtable_cache[SYMTABLE_CACHE_SIZE];
	zend_array **symtable_cache_limit;
	zend_array **symtable_cache_ptr;

	zend_array symbol_table;		/* main symbol table */

	HashTable included_files;	/* files already included */

	JMP_BUF *bailout;

	int error_reporting;
	int exit_status;

	HashTable *function_table;	/* function symbol table */
	HashTable *class_table;		/* class table */
	HashTable *zend_constants;	/* constants table */

	zval          *vm_stack_top;
	zval          *vm_stack_end;
	zend_vm_stack  vm_stack;

	struct _zend_execute_data *current_execute_data;
	zend_class_entry *fake_scope; /* used to avoid checks accessing properties */

	zend_long precision;

	int ticks_count;

	HashTable *in_autoload;
	zend_function *autoload_func;
	zend_bool full_tables_cleanup;

	/* for extended information support */
	zend_bool no_extensions;

	zend_bool vm_interrupt;
	zend_bool timed_out;
	zend_long hard_timeout;

#ifdef ZEND_WIN32
	OSVERSIONINFOEX windows_version_info;
#endif

	HashTable regular_list;
	HashTable persistent_list;

	int user_error_handler_error_reporting;
	zval user_error_handler;
	zval user_exception_handler;
	zend_stack user_error_handlers_error_reporting;
	zend_stack user_error_handlers;
	zend_stack user_exception_handlers;

	zend_error_handling_t  error_handling;
	zend_class_entry      *exception_class;

	/* timeout support */
	zend_long timeout_seconds;

	int lambda_count;

	HashTable *ini_directives;
	HashTable *modified_ini_directives;
	zend_ini_entry *error_reporting_ini_entry;

	zend_objects_store objects_store;
	zend_object *exception, *prev_exception;
	const zend_op *opline_before_exception;
	zend_op exception_op[3];

	struct _zend_module_entry *current_module;

	zend_bool active;
	zend_uchar flags;

	zend_long assertions;

	uint32_t           ht_iterators_count;     /* number of allocatd slots */
	uint32_t           ht_iterators_used;      /* number of used slots */
	HashTableIterator *ht_iterators;
	HashTableIterator  ht_iterators_slots[16];

	void *saved_fpu_cw_ptr;
#if XPFPA_HAVE_CW
	XPFPA_CW_DATATYPE saved_fpu_cw;
#endif

	zend_function trampoline;
	zend_op       call_trampoline_op;

	zend_bool each_deprecation_thrown;

	void *reserved[ZEND_MAX_RESERVED_RESOURCES];
};

可以看到 executor_globals 中保存了很多信息，其中就有之前熟知的 function_table、class_table、zend_constants，
这里我们重点看的是 	HashTable included_files;	/* files already included */
这个就是用来做重复包含文件的，是一个HashTable类型的变量，可想而知我们可以使用操作HashTable的一些函数来操作它
```

### cli模式执行文件的流程
```
首先来看一下php_cli中是怎样执行一个.php文件的，在main/php_cli.c中可以看到这样的代码
static int do_cli(int argc, char **argv)
{
  ...
  zend_file_handle file_handle;
  ...
  file_handle.type = ZEND_HANDLE_FP;
  file_handle.opened_path = NULL;
	file_handle.free_filename = 0;
  ...
  php_execute_script(&file_handle);
  ...
}

php_execute_script函数的原型:
PHPAPI int php_execute_script(zend_file_handle *primary_file)

看下zend_file_handle结构体的定义:
typedef struct _zend_file_handle {
	union {
		int           fd;
		FILE          *fp;
		zend_stream   stream;
	} handle;
	const char        *filename;
	zend_string       *opened_path;
	zend_stream_type  type;
	zend_bool free_filename;
} zend_file_handle;

可见输入的是 php_execute_script 函数的是一个文件信息

PHPAPI int php_execute_script(zend_file_handle *primary_file)
{
  ...
  retval = (zend_execute_scripts(ZEND_REQUIRE, NULL, 3, prepend_file_p, primary_file, append_file_p) == SUCCESS);
  ... 
}

ZEND_API int zend_execute_scripts(int type, zval *retval, int file_count, ...)
{
  zend_op_array *op_array;
  ...
    op_array = zend_compile_file(file_handle, type);
		if (file_handle->opened_path) {
			zend_hash_add_empty_element(&EG(included_files), file_handle->opened_path);
		}
		zend_destroy_file_handle(file_handle);
		if (op_array) {
			zend_execute(op_array, retval);
			zend_exception_restore();
			zend_try_exception_handler();
			if (EG(exception)) {
				zend_exception_error(EG(exception), E_ERROR);
			}
			destroy_op_array(op_array);
			efree_size(op_array, sizeof(zend_op_array));
		} else if (type==ZEND_REQUIRE) {
			va_end(files);
			return FAILURE;
		}
  ...
}

从上面的 op_array = zend_compile_file(file_handle, type) 可以看出 zend_compile_file 函数就是把php代码编译成opcode的
函数输入一个文件信息，输出抽象语法树的产物( zend_op_array)

typedef struct _zend_op_array zend_op_array;
struct _zend_op_array {
  ...
  zend_op *opcodes;
  ...
};

typedef struct _zend_op zend_op;
struct _zend_op {
	const void *handler;
	znode_op op1;
	znode_op op2;
	znode_op result;
	uint32_t extended_value;
	uint32_t lineno;
	zend_uchar opcode;
	zend_uchar op1_type;
	zend_uchar op2_type;
	zend_uchar result_type;
};

编译完成得到opcode，然后调用zend_execute(op_array, retval);执行opcode
```
