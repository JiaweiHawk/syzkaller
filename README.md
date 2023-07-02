# syzkaller - kernel fuzzer

参考[syzkaller教程](https://i-m.dev/posts/20200313-143737.html)

1. 配置go环境

```bash
wget -O go.tar.gz https://go.dev/dl/go1.20.5.linux-amd64.tar.gz
tar -C ~ -xvf go.tar.gz
echo "export GOROOT=$HOME/go
export PATH=\$GOROOT/bin:\$PATH" | tee -a ~/.bashrc
```

2. 编译syzkaller
```bash
cd `/path/to/syzkaller`
git clone https://github.com/JiaweiHawk/syzkaller.git
cd syzkaller
make
```

2. 制作系统镜像
```bash
cd `/path/to/image`
`/path/to/syzkaller`/syzkaller/tools/create-image.sh -s 2048
```

3. 编译内核
```bash
cd `/path/to/kernel`
git clone git://mirrors.ustc.edu.cn/linux.git
cd linux
make defconfig
./scripts/config --enable CONFIG_KCOV && yes "" | make oldconfig
./scripts/config --enable CONFIG_DEBUG_INFO_DWARF5 && yes "" | make oldconfig
./scripts/config --enable CONFIG_KASAN && yes "" | make oldconfig
./scripts/config --enable CONFIG_KASAN_INLINE && yes "" | make oldconfig
./scripts/config --enable CONFIG_CONFIGFS_FS && yes "" | make oldconfig
./scripts/config --enable CONFIG_SECURITYFS && yes "" | make oldconfig
make -j $(nproc)
```

4. 运行syzkaller
```
echo "{
    "target": "linux/amd64",
    "http": "127.0.0.1:8080",
    "workdir": "`/path/to/workdir`",
    "kernel_obj": "`/path/to/kernel`/linux",
    "kernel_src": "`/path/to/kernel`/linux",
    "image": "`/path/to/image`/bullseye.img",
    "sshkey": "`/path/to/image`/bullseye.id_rsa",
    "syzkaller": "`/path/to/syzkaller`/syzkaller",
    "enable_syscalls": [
        "open$test",
        "close$test"
    ],
    "procs": 1,
    "type": "qemu",
    "sandbox": "setuid",
    "vm": {
        "count": 1,
        "kernel": "`/path/to/kernel`/linux/arch/x86/boot/bzImage",
        "cmdline": "net.ifnames=0",
        "cpu": 1,
        "mem": 2048
    }
}" | tee `/path/to/config`/config
`/path/to/syzkaller`/bin/syz-manager -config `/path/to/config`/config
```
