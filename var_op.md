## 变量的操作
```
变量是一门语言的基础，PHP内核中的变量是为了描述高级PHP语言中的变量而实现的
PHP语言中的变量有类型，同样在PHP内核中变量也是有类型的
变量有三部分组成: 变量名、变量值、变量类型

都知道PHP7相对于PHP5，有比较大的变更，在变量的表示方面也做了较大调整，即zval的变更
```

### PHP7 zval
```
// Zend/zend_types
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

