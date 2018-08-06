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
```
