project('mecab', 'cpp', version : '0.996')

add_project_arguments('-DHAVE_CONFIG_H', language : 'cpp')

compiler = meson.get_compiler('cpp')

conf = configuration_data()
conf.set_quoted('PACKAGE', meson.project_name())
conf.set_quoted('VERSION', meson.project_version())
conf.set('DIC_VERSION', 102)

conf.set('SIZEOF_CHAR', compiler.sizeof('char'))
conf.set('SIZEOF_INT', compiler.sizeof('int'))
conf.set('SIZEOF_LONG', compiler.sizeof('long'))
conf.set('SIZEOF_LONG_LONG', compiler.sizeof('long long'))
conf.set('SIZEOF_SHORT', compiler.sizeof('short'))
conf.set('SIZEOF_SIZE_T', compiler.sizeof('size_t'))

required_headers = [
  'vector',
  'list',
  'map',
  'set',
  'queue',
  'functional',
  'algorithm',
  'string',
  'iostream',
  'sstream',
  'fstream',
]
foreach header : required_headers
  assert(compiler.has_header(header))
endforeach

optional_headers = [
  ['HAVE_CTYPE_H', 'ctype.h'],
  ['HAVE_STDINT_H', 'stdint.h'],
  ['HAVE_MEMORY_H', 'memory.h'],
  ['HAVE_DIRENT_H', 'dirent.h'],
  ['HAVE_DLFCN_H', 'dlfcn.h'],
  ['HAVE_FCNTL_H', 'fcntl.h'],
  ['HAVE_INTTYPES_H', 'inttypes.h'],
  ['HAVE_IO_H', 'io.h'],
  ['HAVE_PTHREAD_H', 'pthread.h'],
  ['HAVE_STRINGS_H', 'strings.h'],
  ['HAVE_STRING_H', 'string.h'],
  ['HAVE_SYS_MMAN_H', 'sys/mman.h'],
  ['HAVE_SYS_PARAM_H', 'sys/param.h'],
  ['HAVE_SYS_STAT_H', 'sys/stat.h'],
  ['HAVE_SYS_TIMES_H', 'sys/times.h'],
  ['HAVE_SYS_TYPES_H', 'sys/types.h'],
  ['HAVE_UNISTD_H', 'unistd.h'],
  ['HAVE_WINDOWS_H', 'windows.h'],
]

foreach header : optional_headers
  conf.set(header.get(0), compiler.has_header(header.get(1)))
endforeach

check_functions = [
  ['HAVE_GETENV', 'getenv', '#include <stdlib.h>'],
  ['HAVE_OPENDIR', 'opendir', '#include <stdlib.h>'],
  ['HAVE_GETPAGESIZE', 'getpagesize', '#include <unistd.h>'],
  ['HAVE_MMAP', 'mmap', '#include <sys/mman.h>'],
  ['HAVE_ICONV', 'iconv', '#include <iconv.h>']
]

foreach f : check_functions
  conf.set(f.get(0), compiler.has_function(f.get(1), prefix : f.get(2)))
endforeach

# idk how to set this correctly in the general case
conf.set('ICONV_CONST', '')

conf.set(
  'HAVE_UNSIGNED_LONG_LONG_INT',
  compiler.compiles('unsigned long long int a = 1;', name : 'has unsigned long long int'),
)

assert(
  compiler.compiles(
    '''
    template <class T> T foo (T &i) { return i++; };
    int main() {
        int i = 0;
        double d = 0.0;
        foo(i);
        foo(d);
    }
    ''',
    name : 'supports template <class T>',
  ),
)

assert(
  compiler.compiles(
    '''
    class foo;
    int main() {
        const foo *c=0;
        foo *c1=const_cast<foo*>(c);
    }
    ''',
    name : 'supports const_cast<>',
  ),
)

assert(
  compiler.compiles(
    '''
    class foo;
    int main() {
        foo *c = 0;
        void *c1 = static_cast<void *>(c);
    }
    ''',
    name : 'supports static_cast<>',
  ),
)

assert(
  compiler.compiles(
    '''
    int main() {
        int *a = 0;
        char *p = reinterpret_cast<char *>(a);
    }
    ''',
    name : 'supports reinterpret_cast<>',
  ),
)

assert(
  compiler.compiles(
    '''
    namespace Foo { struct A {}; }
    using namespace Foo;
    A a;
    ''',
    name : 'supports namespaces',
  ),
)

have_tls_keyword = compiler.compiles(
  '''
  __thread int a = 0;
  int main() {
      a = 10;
  }
  ''',
  name : 'supports __thread',
)

conf.set('HAVE_TLS_KEYWORD', have_tls_keyword)
if not have_tls_keyword
  warning(
    '''__thread keyword is not supported on this environment.
Error handling of MeCab, e.g., MeCab::getLastError(), is not thread safe.
''')
endif

conf.set(
  'HAVE_GCC_ATOMIC_OPS',
  compiler.compiles(
    '''
    #include <sched.h>
    int main() {
        int a = 10;
        __sync_fetch_and_add(&a, 10);
        __sync_val_compare_and_swap(&a, 0, 10);
        sched_yield();
    }
    ''',
    name : 'supports GCC native atomic operations',
  ),
)

conf.set(
  'HAVE_OSX_ATOMIC_OPS',
  compiler.compiles(
    '''
    #include <libkern/OSAtomic.h>
    int main() {
        int a = 10;
        OSAtomicAdd32(10, &a);
        OSAtomicCompareAndSwapInt(10, 0, &a);
    }
    ''',
    name : 'supports OSX native atomic operations',
  ),
)

if host_machine.system().startswith('mingw') or host_machine.system().startswith('os2')
  conf.set_quoted(
    'MECAB_DEFAULT_RC',
    'c:\\\\Program Files\\\\mecab\\\\etc\\\\mecabrc',
  )
else
  conf.set_quoted('MECAB_DEFAULT_RC', 'mecabrc')
endif

thread_dep = dependency('threads', required : false)
conf.set('HAVE_LIBPTHREAD', thread_dep.found())

if get_option('utf8-only')
  conf.set('MECAB_USE_UTF8_ONLY', true)
endif

configure_file(
  output : 'config.h',
  configuration : conf,
)
config_inc = include_directories('.')

subdir('src')
