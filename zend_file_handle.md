## zend_file_handle 解析
```
// Zend/zend_stream.h

typedef enum {
	ZEND_HANDLE_FILENAME,
	ZEND_HANDLE_FD,
	ZEND_HANDLE_FP,
	ZEND_HANDLE_STREAM,
	ZEND_HANDLE_MAPPED
} zend_stream_type;

typedef struct _zend_mmap {
	size_t      len;
	size_t      pos;
	void        *map;
	char        *buf;
	void                  *old_handle;
	zend_stream_closer_t   old_closer;
} zend_mmap;

typedef struct _zend_stream {
	void        *handle;
	int         isatty;
	zend_mmap   mmap;
	zend_stream_reader_t   reader;
	zend_stream_fsizer_t   fsizer;
	zend_stream_closer_t   closer;
} zend_stream;

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

zend_file_handle 作为承载执行php脚本的一个结构体载体，内部可容纳多种类型的文件模式，包含:fd、FILE、filename、fp

php内核中执行php脚本的函数，内部兼容了各种 zend_stream_type 的文件类型
PHPAPI int php_execute_script(zend_file_handle *primary_file){
 //......
}
可见这里传递的是一个 zend_file_handle*

下面看一下 zend_file_handle.type 的各种值的情况:
1. ZEND_HANDLE_FILENAME
    zend_file_handle file_handle;
    file_handle.type = ZEND_HANDLE_FILENAME;
    file_handle.filename = filepath; // filepath 是一个.php文件的全路径
    file_handle.free_filename = 0;
    file_handle.opened_path = NULL;

2. ZEND_HANDLE_FD
    zend_file_handle file_handle;
    file_handle->type = ZEND_HANDLE_FP;
    file_handle->handle.fp = fdopen(file_handle->handle.fd, "rb");
    
3. ZEND_HANDLE_FP
    zend_file_handle file_handle;
    file_handle->type = ZEND_HANDLE_FP;

4. ZEND_HANDLE_STREAM
    zend_file_handle file_handle;
    file_handle->type = ZEND_HANDLE_STREAM;
    
5. ZEND_HANDLE_MAPPED
    zend_file_handle file_handle;
    file_handle->type = ZEND_HANDLE_MAPPED;
    file_handle->handle.stream.mmap.pos = 0;
    *buf = file_handle->handle.stream.mmap.buf;
    *len = file_handle->handle.stream.mmap.len;
```
