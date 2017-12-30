## PHP7内核加载扩展
每个扩展中都会定义一个描述扩展信息的zend_module_entry结构体，这里就讲述一下扩展怎样被PHP内核加载

### 加载扩展钩子
每个扩展都会定义一个具有外部链接的扩展结构体变量，然后扩展中提供一个有特定规律的函数名称
当在内核加载每个扩展的时候依次来调用这类函数，这类函数的返回值就是这个扩展结构体变量的内存地址

### 扩展中定义
```
#ifdef COMPILE_DL_TEST
#ifdef ZTS
ZEND_TSRMLS_CACHE_DEFINE()
#endif
ZEND_GET_MODULE(test)
#endif

// 这里用了#ifdef预处理条件来控制不会重复加载同一个扩展
```
宏定义
```
// Zend/zend_API.h
#define ZEND_GET_MODULE(name) \
    BEGIN_EXTERN_C()\
	ZEND_DLEXPORT zend_module_entry *get_module(void) { return &name##_module_entry; }\
    END_EXTERN_C()

// Zend/zend_config.w32.h
#define ZEND_DLEXPORT		__declspec(dllexport)
    
// Zend/zend_portability.h
#ifdef __cplusplus
#define BEGIN_EXTERN_C() extern "C" {
#define END_EXTERN_C() }
#else
#define BEGIN_EXTERN_C()
#define END_EXTERN_C()
#endif    
```
宏展开结果
```
ZEND_GET_MODULE(test)

的宏展开:

extern "C" {
  __declspec(dllexport) zend_module_entry *get_module(void) 
  { 
    return &test_module_entry; 
  }
}
可见这里的确是返回了一个结构体变量的地址
```
