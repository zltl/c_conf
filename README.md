c_conf - A simple configuration file parser implemented by C
---

c_conf is a simple configuration file parser implemented by C.
Configuration file example:

```
# comments

# [] will be ignore
[title1]
# bool value
is_daemon no

[title2]

# memory size
max_memory 1G

# time duration
timeout 10m5s

# string
log_path /tmp/logs.txt

```

Usage example:

```C
#include "conf_parse.h"

/**
 * 配置项数据结构
 */
struct my_conf {
    int is_daemon;
    long max_memory;
    char log_path[100];
};

int main(int argc, const char *argv[]) {
    static struct my_conf conf;
    struct my_conf *pconf = &conf;

    parse_command_t commands[] = {

        // CONF_CMD_BEGIN() 必须第一个
        CONF_CMD_BEGIN(commands),

        // 设置 is_daemon 配置项为 bool 类型，默认值为 yes
        // yes -> 1, no -> 0
        CONF_CMD_BOOL(pconf, is_daemon, "yes"),

        // 设置 max_memory 配置项为 字节 类型，默认值为 10Mib
        // 最终 max_meory=10*1024*1024
        CONF_CMD_MEM(pconf, max_memory, "10M"),

        // 设置 log_path 配置项为 字符串 类型，默认值为 /tmp/log.txt
        CONF_CMD_STR(pconf, log_path, "/tmp/log.txt"),

        // CONF_CMD_END() 必须最后
        CONF_CMD_END(),
    };

    /* 应用默认值 */
    if (conf_init(commands) < 0) {
        printf("conf_init return -1\n");
        return -1;
    }

    /* 使用环境变量覆盖值 */
    if (conf_parse_env(commands) < 0) {
        printf("conf_parse_env return -1\n");
        return -1;
    }

    const char *file = "/tmp/conf.ini";
    /* 使用配置文件覆盖默认值 */
    if (file && conf_parse_file(commands, file) < 0) {
        printf("conf_parse_file return -1\n");
        // 文件可能不能存在，也是可以接受的，使用默认值或命令参数
    }

    /* 使用 argc, argv 覆盖默认值 */
    if (conf_parse_args(commands, argc, argv) < 0) {
        printf("conf_parse_args return -1\n");
        return -1;
    }
    /* 打印最终配置值 */
    conf_print_conf(stdout, commands);

    // 使用配置项

    printf("is_daemon: %d\n", conf.is_daemon);

    return 0;
}
```
