==============================================================
[2017_Inc0gnito] [REV] leon
==============================================================

문제내용
==============================================================

Please.. come..
Service available at: Here
Source code available at: Here


문제 풀이
==============================================================


strace는 애플리케이션들이 사용하는 system call과 signal 등을 추적해서 성능 저하를 일으키는 부분은 없는지, 에러가 나는 부분은 없는지를 확인하는데 사용하는 디버깅 툴이다.


strace를 통해 해당 바이너리 파일을 확인해보자.

.. code-block:: bash

    $ strace ./leon
    execve("./leon", ["./leon"], [/* 36 vars */]) = 0
    brk(NULL)                               = 0xc72000
    access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
    mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f07e912e000
    access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
    open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
    fstat(3, {st_mode=S_IFREG|0644, st_size=54768, ...}) = 0
    mmap(NULL, 54768, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f07e9120000
    close(3)                                = 0
    access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
    open("/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
    read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\t\2\0\0\0\0\0"..., 832) = 832
    fstat(3, {st_mode=S_IFREG|0755, st_size=1864888, ...}) = 0
    mmap(NULL, 3967392, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f07e8b42000
    mprotect(0x7f07e8d01000, 2097152, PROT_NONE) = 0
    mmap(0x7f07e8f01000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1bf000) = 0x7f07e8f01000
    mmap(0x7f07e8f07000, 14752, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f07e8f07000
    close(3)                                = 0
    mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f07e911f000
    mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f07e911e000
    mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f07e911d000
    arch_prctl(ARCH_SET_FS, 0x7f07e911e700) = 0
    mprotect(0x7f07e8f01000, 16384, PROT_READ) = 0
    mprotect(0x600000, 4096, PROT_READ)     = 0
    mprotect(0x7f07e9130000, 4096, PROT_READ) = 0
    munmap(0x7f07e9120000, 54768)           = 0
    ptrace(PTRACE_TRACEME, 0, 0x1, NULL)    = -1 EPERM (Operation not permitted)
    exit_group(1)                           = ?
    +++ exited with 1 +++

뭔가 파일이 존재하지 않는다고 하면서 ENOENT 에러를 뱉고, ptrace로 인해 EPERM 에러를 뱉는 것을 확인할 수 있습니다.

objdump는 GNU 바이너리 유틸리티의 일부로서, 라이브러리, 컴파일된 오브젝트 모듈, 공유 오브젝트 파일, 독립 실행파일등의 바이너리 파일들의 정보를 보여주는 프로그램이다. objdump는 ELF 파일을 어셈블리어로 보여주는 역어셈블러로 사용될 수 있다.

.. code-block:: bash

    $ objdump -d ./leon | grep \<ptrace@plt\>
    0000000000400660 <ptrace@plt>:
      400818:       e8 43 fe ff ff          callq  400660 <ptrace@plt>

ptrace 주소를 확인하였으니, 해당 주소 이전에 브레이크를 걸어보자.

.. code-block:: bash

    gdb-peda$ b *0x40081d
    Breakpoint 1 at 0x40081d
    gdb-peda$ r
    Starting program: /home/joizel/leon
    [----------------------------------registers-----------------------------------]
    RAX: 0xffffffffffffffff
    RBX: 0x0
    RCX: 0x7ffff7b0adee (<ptrace+78>:       cmp    rax,0xfffffffffffff000)
    RDX: 0xffffffffffffff98
    RSI: 0x0
    RDI: 0x0
    RBP: 0x7fffffffe210 --> 0x4008a0 (<__libc_csu_init>:    push   r15)
    RSP: 0x7fffffffe1b0 --> 0x2a ('*')
    RIP: 0x40081d (<main+54>:       cmp    rax,0xffffffffffffffff)
    R8 : 0xffffffff
    R9 : 0x7ffff7de7ab0 (<_dl_fini>:        push   rbp)
    R10: 0x0
    R11: 0x282
    R12: 0x400680 (<_start>:        xor    ebp,ebp)
    R13: 0x7fffffffe2f0 --> 0x1
    R14: 0x0
    R15: 0x0
    EFLAGS: 0x213 (CARRY parity ADJUST zero sign trap INTERRUPT direction overflow)
    [-------------------------------------code-------------------------------------]
       0x40080e <main+39>:  mov    edi,0x0
       0x400813 <main+44>:  mov    eax,0x0
       0x400818 <main+49>:  call   0x400660 <ptrace@plt>
    => 0x40081d <main+54>:  cmp    rax,0xffffffffffffffff
       0x400821 <main+58>:  jne    0x40082d <main+70>
       0x400823 <main+60>:  mov    edi,0x1
       0x400828 <main+65>:  call   0x400670 <exit@plt>
       0x40082d <main+70>:  mov    rdx,QWORD PTR [rip+0x20088c]        # 0x6010c0 <stdin@@GLIBC_2.2.5>
    [------------------------------------stack-------------------------------------]
    0000| 0x7fffffffe1b0 --> 0x2a ('*')
    0008| 0x7fffffffe1b8 --> 0x601080 ("INC0{doesn't_seem_to_be_write_something..}")
    0016| 0x7fffffffe1c0 --> 0x7fffffffe1d0 --> 0x2
    0024| 0x7fffffffe1c8 --> 0x4007df (<_init_+24>: pop    rbp)
    0032| 0x7fffffffe1d0 --> 0x2
    0040| 0x7fffffffe1d8 --> 0x4008ed (<__libc_csu_init+77>:        add    rbx,0x1)
    0048| 0x7fffffffe1e0 --> 0xff000000
    0056| 0x7fffffffe1e8 --> 0x0
    [------------------------------------------------------------------------------]
    Legend: code, data, rodata, value

    Breakpoint 1, 0x000000000040081d in main ()
    gdb-peda$


스택상에서 플래그 확인 
