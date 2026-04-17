```Makefile
obj-m += lkm.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

```


```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/umh.h> // The magic header for user-mode helpers

MODULE_LICENSE("GPL");
MODULE_AUTHOR("The Architect");

static int __init exec_init(void) {
    /* Set up the path and arguments */
    // char *argv[] = { "/tmp/test.sh", NULL };

    /*
     * cat /usr/local/bin/test.sh 
     * #!/bin/bash
     * sudo /bin/bash -c 'whoami > /tmp/WHOAMI'
    */
    char *argv[] = { "/usr/local/bin/test.sh", NULL };

    static char *envp[] = {
        "HOME=/",
        "TERM=linux",
        "PATH=/sbin:/usr/sbin:/bin:/usr/bin",
        NULL
    };

    printk(KERN_INFO "exec_init: Attempting to execute\n");

    /* * call_usermodehelper(path, argv, envp, wait_flag)
     * UMH_WAIT_PROC: Wait for the process to finish
     */
    return call_usermodehelper(argv[0], argv, envp, UMH_WAIT_PROC);
}

static void __exit exec_exit(void) {
    printk(KERN_INFO "exec_exit: Executor unloaded.\n");
}

module_init(exec_init);
module_exit(exec_exit);
```
