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
#define ZVAL_UNDEF(z) do {				\
		Z_TYPE_INFO_P(z) = IS_UNDEF;	\
	} while (0)

#define ZVAL_NULL(z) do {				\
		Z_TYPE_INFO_P(z) = IS_NULL;		\
	} while (0)

#define ZVAL_FALSE(z) do {				\
		Z_TYPE_INFO_P(z) = IS_FALSE;	\
	} while (0)

#define ZVAL_TRUE(z) do {				\
		Z_TYPE_INFO_P(z) = IS_TRUE;		\
	} while (0)

#define ZVAL_BOOL(z, b) do {			\
		Z_TYPE_INFO_P(z) =				\
			(b) ? IS_TRUE : IS_FALSE;	\
	} while (0)

#define ZVAL_LONG(z, l) {				\
		zval *__z = (z);				\
		Z_LVAL_P(__z) = l;				\
		Z_TYPE_INFO_P(__z) = IS_LONG;	\
	}

#define ZVAL_DOUBLE(z, d) {				\
		zval *__z = (z);				\
		Z_DVAL_P(__z) = d;				\
		Z_TYPE_INFO_P(__z) = IS_DOUBLE;	\
	}

#define ZVAL_STR(z, s) do {						\
		zval *__z = (z);						\
		zend_string *__s = (s);					\
		Z_STR_P(__z) = __s;						\
		/* interned strings support */			\
		Z_TYPE_INFO_P(__z) = ZSTR_IS_INTERNED(__s) ? \
			IS_INTERNED_STRING_EX : 			\
			IS_STRING_EX;						\
	} while (0)

#define ZVAL_INTERNED_STR(z, s) do {				\
		zval *__z = (z);							\
		zend_string *__s = (s);						\
		Z_STR_P(__z) = __s;							\
		Z_TYPE_INFO_P(__z) = IS_INTERNED_STRING_EX;	\
	} while (0)

#define ZVAL_NEW_STR(z, s) do {					\
		zval *__z = (z);						\
		zend_string *__s = (s);					\
		Z_STR_P(__z) = __s;						\
		Z_TYPE_INFO_P(__z) = IS_STRING_EX;		\
	} while (0)

#define ZVAL_STR_COPY(z, s) do {						\
		zval *__z = (z);								\
		zend_string *__s = (s);							\
		Z_STR_P(__z) = __s;								\
		/* interned strings support */					\
		if (ZSTR_IS_INTERNED(__s)) {							\
			Z_TYPE_INFO_P(__z) = IS_INTERNED_STRING_EX;	\
		} else {										\
			GC_REFCOUNT(__s)++;							\
			Z_TYPE_INFO_P(__z) = IS_STRING_EX;			\
		}												\
	} while (0)

#define ZVAL_ARR(z, a) do {						\
		zval *__z = (z);						\
		Z_ARR_P(__z) = (a);						\
		Z_TYPE_INFO_P(__z) = IS_ARRAY_EX;		\
	} while (0)

#define ZVAL_NEW_ARR(z) do {									\
		zval *__z = (z);										\
		zend_array *_arr =										\
		(zend_array *) emalloc(sizeof(zend_array));				\
		Z_ARR_P(__z) = _arr;									\
		Z_TYPE_INFO_P(__z) = IS_ARRAY_EX;						\
	} while (0)

#define ZVAL_NEW_PERSISTENT_ARR(z) do {							\
		zval *__z = (z);										\
		zend_array *_arr =										\
		(zend_array *) malloc(sizeof(zend_array));				\
		Z_ARR_P(__z) = _arr;									\
		Z_TYPE_INFO_P(__z) = IS_ARRAY_EX;						\
	} while (0)

#define ZVAL_OBJ(z, o) do {						\
		zval *__z = (z);						\
		Z_OBJ_P(__z) = (o);						\
		Z_TYPE_INFO_P(__z) = IS_OBJECT_EX;		\
	} while (0)

#define ZVAL_RES(z, r) do {						\
		zval *__z = (z);						\
		Z_RES_P(__z) = (r);						\
		Z_TYPE_INFO_P(__z) = IS_RESOURCE_EX;	\
	} while (0)

#define ZVAL_NEW_RES(z, h, p, t) do {							\
		zend_resource *_res =									\
		(zend_resource *) emalloc(sizeof(zend_resource));		\
		zval *__z;												\
		GC_REFCOUNT(_res) = 1;									\
		GC_TYPE_INFO(_res) = IS_RESOURCE;						\
		_res->handle = (h);										\
		_res->type = (t);										\
		_res->ptr = (p);										\
		__z = (z);												\
		Z_RES_P(__z) = _res;									\
		Z_TYPE_INFO_P(__z) = IS_RESOURCE_EX;					\
	} while (0)

#define ZVAL_NEW_PERSISTENT_RES(z, h, p, t) do {				\
		zend_resource *_res =									\
		(zend_resource *) malloc(sizeof(zend_resource));		\
		zval *__z;												\
		GC_REFCOUNT(_res) = 1;									\
		GC_TYPE_INFO(_res) = IS_RESOURCE;						\
		_res->handle = (h);										\
		_res->type = (t);										\
		_res->ptr = (p);										\
		__z = (z);												\
		Z_RES_P(__z) = _res;									\
		Z_TYPE_INFO_P(__z) = IS_RESOURCE_EX;					\
	} while (0)

#define ZVAL_REF(z, r) do {										\
		zval *__z = (z);										\
		Z_REF_P(__z) = (r);										\
		Z_TYPE_INFO_P(__z) = IS_REFERENCE_EX;					\
	} while (0)

#define ZVAL_NEW_EMPTY_REF(z) do {								\
		zend_reference *_ref =									\
		(zend_reference *) emalloc(sizeof(zend_reference));		\
		GC_REFCOUNT(_ref) = 1;									\
		GC_TYPE_INFO(_ref) = IS_REFERENCE;						\
		Z_REF_P(z) = _ref;										\
		Z_TYPE_INFO_P(z) = IS_REFERENCE_EX;						\
	} while (0)

#define ZVAL_NEW_REF(z, r) do {									\
		zend_reference *_ref =									\
		(zend_reference *) emalloc(sizeof(zend_reference));		\
		GC_REFCOUNT(_ref) = 1;									\
		GC_TYPE_INFO(_ref) = IS_REFERENCE;						\
		ZVAL_COPY_VALUE(&_ref->val, r);							\
		Z_REF_P(z) = _ref;										\
		Z_TYPE_INFO_P(z) = IS_REFERENCE_EX;						\
	} while (0)

#define ZVAL_NEW_PERSISTENT_REF(z, r) do {						\
		zend_reference *_ref =									\
		(zend_reference *) malloc(sizeof(zend_reference));		\
		GC_REFCOUNT(_ref) = 1;									\
		GC_TYPE_INFO(_ref) = IS_REFERENCE;						\
		ZVAL_COPY_VALUE(&_ref->val, r);							\
		Z_REF_P(z) = _ref;										\
		Z_TYPE_INFO_P(z) = IS_REFERENCE_EX;						\
	} while (0)

#define ZVAL_NEW_AST(z, a) do {									\
		zval *__z = (z);										\
		zend_ast_ref *_ast =									\
		(zend_ast_ref *) emalloc(sizeof(zend_ast_ref));			\
		GC_REFCOUNT(_ast) = 1;									\
		GC_TYPE_INFO(_ast) = IS_CONSTANT_AST;					\
		_ast->ast = (a);										\
		Z_AST_P(__z) = _ast;									\
		Z_TYPE_INFO_P(__z) = IS_CONSTANT_AST_EX;				\
	} while (0)

#define ZVAL_INDIRECT(z, v) do {								\
		Z_INDIRECT_P(z) = (v);									\
		Z_TYPE_INFO_P(z) = IS_INDIRECT;							\
	} while (0)

#define ZVAL_PTR(z, p) do {										\
		Z_PTR_P(z) = (p);										\
		Z_TYPE_INFO_P(z) = IS_PTR;								\
	} while (0)

#define ZVAL_FUNC(z, f) do {									\
		Z_FUNC_P(z) = (f);										\
		Z_TYPE_INFO_P(z) = IS_PTR;								\
	} while (0)

#define ZVAL_CE(z, c) do {										\
		Z_CE_P(z) = (c);										\
		Z_TYPE_INFO_P(z) = IS_PTR;								\
	} while (0)

#define ZVAL_ERROR(z) do {				\
		Z_TYPE_INFO_P(z) = _IS_ERROR;	\
	} while (0)
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


