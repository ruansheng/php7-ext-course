## 变量的操作
```
变量是一门语言的基础，PHP内核中的变量是为了描述高级PHP语言中的变量而实现的
PHP语言中的变量有类型，同样在PHP内核中变量也是有类型的
变量有三部分组成: 变量名、变量值、变量类型

都知道PHP7相对于PHP5，有比较大的变更，在变量的表示方面也做了较大调整，即zval的变更
```

### PHP7 zval
```
// Zend/zend_types.h
typedef struct _zval_struct     zval;

struct _zval_struct {
	zend_value        value;			/* value */
	union {
		struct {
			ZEND_ENDIAN_LOHI_4(
				zend_uchar    type,			/* active type */
				zend_uchar    type_flags,
				zend_uchar    const_flags,
				zend_uchar    reserved)	    /* call info for EX(This) */
		} v;
		uint32_t type_info;
	} u1;
	union {
		uint32_t     next;                 /* hash collision chain */
		uint32_t     cache_slot;           /* literal cache slot */
		uint32_t     lineno;               /* line number (for ast nodes) */
		uint32_t     num_args;             /* arguments number for EX(This) */
		uint32_t     fe_pos;               /* foreach position */
		uint32_t     fe_iter_idx;          /* foreach iterator index */
		uint32_t     access_flags;         /* class constant access flags */
		uint32_t     property_guard;       /* single property guard */
		uint32_t     extra;                /* not further specified */
	} u2;
};

typedef union _zend_value {
	zend_long         lval;				/* long value */
	double            dval;				/* double value */
	zend_refcounted  *counted;
	zend_string      *str;
	zend_array       *arr;
	zend_object      *obj;
	zend_resource    *res;
	zend_reference   *ref;
	zend_ast_ref     *ast;
	zval             *zv;
	void             *ptr;
	zend_class_entry *ce;
	zend_function    *func;
	struct {
		uint32_t w1;
		uint32_t w2;
	} ww;
} zend_value;

由上可看出PHP7中把zval的名和值分开了，分别是 zval、zend_value
可以看zend_value中包含有 zend_long、double、zend_string、zend_array、zend_object、zend_resource ...

// 定义变量
zval value1;
zval *value1;
zend_string *zstr;
zend_array *zarr;
...
```

### 变量初始化
下面这些宏定义就是为zval变量赋值，虽然可以手动来初始化，但还是推荐用内核提供的方式来处理
因为初始化并非只是设置zend_value，还好对zval的其它字段设置
```
// Zend/zend_types.h
ZVAL_UNDEF(z)
ZVAL_NULL(z)
ZVAL_FALSE(z)
ZVAL_TRUE(z) 
ZVAL_BOOL(z, b)
ZVAL_LONG(z, l)
ZVAL_DOUBLE(z, d)
ZVAL_STR(z, s)
ZVAL_INTERNED_STR(z, s)
ZVAL_NEW_STR(z, s)
ZVAL_STR_COPY(z, s)
ZVAL_ARR(z, a)
ZVAL_NEW_ARR(z)
ZVAL_NEW_PERSISTENT_ARR(z)
ZVAL_OBJ(z, o)
ZVAL_RES(z, r)
ZVAL_NEW_RES(z, h, p, t)
ZVAL_NEW_PERSISTENT_RES(z, h, p, t)
ZVAL_REF(z, r)
ZVAL_NEW_EMPTY_REF(z)
ZVAL_NEW_REF(z, r)
ZVAL_NEW_PERSISTENT_REF(z, r)
ZVAL_NEW_AST(z, a)
ZVAL_INDIRECT(z, v)
ZVAL_PTR(z, p) 
ZVAL_FUNC(z, f)
ZVAL_CE(z, c) 
ZVAL_ERROR(z)

// Zend/zend_API.h
ZVAL_STRINGL(z, s, l)
ZVAL_STRING(z, s)
ZVAL_EMPTY_STRING(z)
ZVAL_PSTRINGL(z, s, l)
ZVAL_PSTRING(z, s)
ZVAL_EMPTY_PSTRING(z)
ZVAL_ZVAL(z, zv, copy, dtor)

上面这些宏的第一个参数都是zval *
```

### 分析ZVAL_STRING(z, s)
```
zval name;
ZVAL_STRING(&name, "hello world");

看下 ZVAL_STRING 具体做了什么事

#define ZVAL_STRING(z, s) do {					\
		const char *_s = (s);					\
		ZVAL_STRINGL(z, _s, strlen(_s));		\
	} while (0)
	
#define ZVAL_STRINGL(z, s, l) do {				\
		ZVAL_NEW_STR(z, zend_string_init(s, l, 0));		\
	} while (0)

#define ZVAL_NEW_STR(z, s) do {					\
		zval *__z = (z);						\
		zend_string *__s = (s);					\
		Z_STR_P(__z) = __s;						\
		Z_TYPE_INFO_P(__z) = IS_STRING_EX;		\
	} while (0)

#define Z_STR(zval)				(zval).value.str
#define Z_STR_P(zval_p)				Z_STR(*(zval_p))

static zend_always_inline zend_string *zend_string_init(const char *str, size_t len, int persistent)
{
	zend_string *ret = zend_string_alloc(len, persistent);

	memcpy(ZSTR_VAL(ret), str, len);
	ZSTR_VAL(ret)[len] = '\0';
	return ret;
}

static zend_always_inline zend_string *zend_string_alloc(size_t len, int persistent)
{
	zend_string *ret = (zend_string *)pemalloc(ZEND_MM_ALIGNED_SIZE(_ZSTR_STRUCT_SIZE(len)), persistent);

	GC_REFCOUNT(ret) = 1;
#if 1
	/* optimized single assignment */
	GC_TYPE_INFO(ret) = IS_STRING | ((persistent ? IS_STR_PERSISTENT : 0) << 8);
#else
	GC_TYPE(ret) = IS_STRING;
	GC_FLAGS(ret) = (persistent ? IS_STR_PERSISTENT : 0);
	GC_INFO(ret) = 0;
#endif
	zend_string_forget_hash_val(ret);
	ZSTR_LEN(ret) = len;
	return ret;
}

由上可看出zval和zend_value是分开的，ZVAL_STRING(z, s)其实就是:
1. 创建一个zend_string*变量
2. 从堆上为zend_string*分配内存
3. 把zend_string*赋值给zval的value，具体表现为 (zval).value.str
```

### zval内存释放
```
对与 long、double、bool 类型的数据，直接在zend_value中存储内容，不会存在从堆上分配内存
但是对于string、array、obj...之类的数据必须要注意内存的释放
在C语言中有malloc就必须有free，否则会造成内存泄露，特别是在php-fpm模型下，内存泄露会产生很大的影响

zval_ptr_dtor(zval_ptr) 的宏参数是一个zval *

// 内存释放宏
#define zval_ptr_dtor(zval_ptr)   _zval_ptr_dtor((zval_ptr) ZEND_FILE_LINE_CC)

ZEND_API void _zval_ptr_dtor_wrapper(zval *zval_ptr)
{

	i_zval_ptr_dtor(zval_ptr ZEND_FILE_LINE_CC);
}

static zend_always_inline void i_zval_ptr_dtor(zval *zval_ptr ZEND_FILE_LINE_DC)
{
	if (Z_REFCOUNTED_P(zval_ptr)) {
		zend_refcounted *ref = Z_COUNTED_P(zval_ptr);
		if (!--GC_REFCOUNT(ref)) {
			_zval_dtor_func(ref ZEND_FILE_LINE_RELAY_CC);
		} else {
			gc_check_possible_root(ref);
		}
	}
}

#define GC_REFCOUNT(p)		(p)->gc.refcount

ZEND_API void ZEND_FASTCALL _zval_dtor_func(zend_refcounted *p ZEND_FILE_LINE_DC)
{
	switch (GC_TYPE(p)) {
		case IS_STRING:
		case IS_CONSTANT: {
				zend_string *str = (zend_string*)p;
				CHECK_ZVAL_STRING_REL(str);
				zend_string_free(str);
				break;
			}
		case IS_ARRAY: {
				zend_array *arr = (zend_array*)p;
				zend_array_destroy(arr);
				break;
			}
		case IS_CONSTANT_AST: {
				zend_ast_ref *ast = (zend_ast_ref*)p;

				zend_ast_destroy_and_free(ast->ast);
				efree_size(ast, sizeof(zend_ast_ref));
				break;
			}
		case IS_OBJECT: {
				zend_object *obj = (zend_object*)p;

				zend_objects_store_del(obj);
				break;
			}
		case IS_RESOURCE: {
				zend_resource *res = (zend_resource*)p;

				/* destroy resource */
				zend_list_free(res);
				break;
			}
		case IS_REFERENCE: {
				zend_reference *ref = (zend_reference*)p;

				i_zval_ptr_dtor(&ref->val ZEND_FILE_LINE_RELAY_CC);
				efree_size(ref, sizeof(zend_reference));
				break;
			}
		default:
			break;
	}
}

可见释放内存的时候
1. 首先获取zval的引用计数
2. 引用计数减1，如果减完之后，refcount == 0，就认为是垃圾，就释放内存
3. 释放内存的时候判断了zval变量的烈类型，不同类型做不同的处理
```

### 变量类型判断
```
// Zend/zend_types
/* regular data types */
#define IS_UNDEF					0
#define IS_NULL						1
#define IS_FALSE					2
#define IS_TRUE						3
#define IS_LONG						4
#define IS_DOUBLE					5
#define IS_STRING					6
#define IS_ARRAY					7
#define IS_OBJECT					8
#define IS_RESOURCE					9
#define IS_REFERENCE				10

/* constant expressions */
#define IS_CONSTANT					11
#define IS_CONSTANT_AST				12

/* fake types */
#define _IS_BOOL					13
#define IS_CALLABLE					14
#define IS_ITERABLE					19
#define IS_VOID						18

/* internal types */
#define IS_INDIRECT             	15
#define IS_PTR						17
#define _IS_ERROR					20

// PHP内涵提供了一些宏来获取变量的类型，然后再和上面的宏值做相等对比
#define Z_TYPE(zval)				zval_get_type(&(zval))
#define Z_TYPE_P(zval_p)			Z_TYPE(*(zval_p))

// 具体使用
zval *zvalp;
...
if(Z_TYPE_P(zvalp) != IS_ARRAY) {
  ...
}
```


