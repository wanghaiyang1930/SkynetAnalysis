### Skynet 源码分析 —— cservice

Skynet设计兼容了灵活性与可扩展性，但是这种松散的插件设计让源码变得有些晦涩难懂，需要一定的时间把握整体设计思路才能融汇贯通。这一章我们来了解一下Skynet下的service-sr与cservice。

service-src中包含了四个标准Skynet C服务插件，而cservice是service-src中服务插件编译后生成的插件动态库，这里不做过多解释。下面我们来逐个分析这四个标准Skynet C服务插件（这里的 C服务插件，特指提供给Skynet内部C代码调用的服务插件，也正是基于这种插件服务，Skynet也可以使用Python写外围业务逻辑）。

```
    service_gate.c
    service_harbor.c
    service_logger.c
    service_snlua.c
```

#### service 插件规范

作为插件肯定需要满足一定的调用规范才能在内部机制中统一管理。Skynet使用skynet_moudle(./skynet-src/skynet_module.h/c)模块管理C服务插件，每个插件按照规范格式实现导出功能函数。

```
    struct module* module_create( void );
    void module_release( struct module* );
    int module_init( struct module*, struct skynet_context *, const char* );
    void module_signal( struct module*, int);
```

#### skynet_module.h/c

skynet_module模块中定义了C服务插件接口，同时也定义了插件操作接口。Skynet使用配置文件中的capth项存储C服务插件目录。Skynet并没有在程序启动时加载所有的插件，而是等到真正用到某个插件时，再加载某个插件，该插件模块为全局唯一，Skynet使用了SPIN_LOCK进行多线程同步。

>C服务插件接口（该接口与上文插件规范一致）

```
    typedef void * (*skynet_dl_create)(void);
    typedef int (*skynet_dl_init)(void * inst, struct skynet_context *, const char * parm);
    typedef void (*skynet_dl_release)(void * inst);
    typedef void (*skynet_dl_signal)(void * inst, int signal);
```

>插件操作接口

```
    struct skynet_module * skynet_module_query(const char * name);
    void * skynet_module_instance_create(struct skynet_module *);
    int skynet_module_instance_init(struct skynet_module *, void * inst, struct skynet_context *ctx, const char * parm);
    void skynet_module_instance_release(struct skynet_module *, void *inst);
    void skynet_module_instance_signal(struct skynet_module *, void *inst, int signal);
```

#### service_snlua.c

#### service_logger.c

#### service_harbor.c

#### service_gate.c