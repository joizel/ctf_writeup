=============================================================================================================
[2015_HITCON] [Forensic] Piranha Gun
=============================================================================================================


일단 nc로 접속해보니 root 쉘이 하나 떨어집니다.

.. code-block:: console

    $ nc 54.178.235.243 10004    

    bash: cannot set terminal process group (-1): Inappropriate ioctl for device
    bash: no job control in this shell
    bash: /root/.bashrc: Permission denied

    

폴더 안을 보니 README라는 파일이 하나 있네요.

.. code-block:: console

    $ ls
    ls
    README

파일을 보니 Piranha Gum은 jungle.chest에서 찾을 수 있다고 합니다.

.. code-block:: console

    $ cat README

    cat README
    The Piranha Gun can be found in "jungle.chest".

프로세스를 확인하려고 보니 다음과 같이 마운트를 하라고 합니다.

.. code-block:: bash

    $ ps aux

    ps aux
    Error, do this: mount -t proc proc /proc

마운트를 한 후 다시 프로세스를 확인해보니 정상적으로 프로세스가 나오는데 특별한 점은 없어 보입니다.

.. code-block:: console

    $ mount -t proc proc /proc

    mount -t proc proc /proc
    
    $ ps aux

    ps aux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         1  0.0  0.0   2116    56 ?        S    00:17   0:00 wrapper root 0 600 262144 /home/PiranhaGun /bin/bash -i
    root         2  0.0  0.0  18196  3300 ?        S    00:17   0:00 /bin/bash -i
    root        16  0.0  0.0  15572  2092 ?        R    00:18   0:00 ps aux

시스템 mount 정보에 대해 확인을 위해 proc/mounts를 보니 뭔가 chest라는 폴더에 mount가 걸려있는게 보입니다.

.. code-block:: console

    $ cat /proc/mounts

    cat /proc/mounts
    /dev/disk/by-uuid/2ed4c374-2ddb-4a75-af24-98df753dbf6d / ext4 rw,relatime,discard,data=ordered 0 0
    udev /dev devtmpfs rw,relatime,size=15702768k,nr_inodes=3925692,mode=755 0 0
    devpts /dev/pts devpts rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000 0 0
    tmpfs /run tmpfs rw,nosuid,noexec,relatime,size=3141528k,mode=755 0 0
    none /run/lock tmpfs rw,nosuid,nodev,noexec,relatime,size=5120k 0 0
    none /run/shm tmpfs rw,nosuid,nodev,relatime 0 0
    none /run/user tmpfs rw,nosuid,nodev,noexec,relatime,size=102400k,mode=755 0 0
    sysfs /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0
    none /sys/fs/cgroup tmpfs rw,relatime,size=4k,mode=755 0 0
    cgroup /sys/fs/cgroup/cpuset cgroup rw,relatime,cpuset 0 0
    cgroup /sys/fs/cgroup/cpu cgroup rw,relatime,cpu 0 0
    cgroup /sys/fs/cgroup/cpuacct cgroup rw,relatime,cpuacct 0 0
    cgroup /sys/fs/cgroup/memory cgroup rw,relatime,memory 0 0
    cgroup /sys/fs/cgroup/devices cgroup rw,relatime,devices 0 0
    cgroup /sys/fs/cgroup/freezer cgroup rw,relatime,freezer 0 0
    cgroup /sys/fs/cgroup/net_cls cgroup rw,relatime,net_cls 0 0
    cgroup /sys/fs/cgroup/blkio cgroup rw,relatime,blkio 0 0
    cgroup /sys/fs/cgroup/perf_event cgroup rw,relatime,perf_event 0 0
    cgroup /sys/fs/cgroup/net_prio cgroup rw,relatime,net_prio 0 0
    cgroup /sys/fs/cgroup/hugetlb cgroup rw,relatime,hugetlb 0 0
    systemd /sys/fs/cgroup/systemd cgroup rw,nosuid,nodev,noexec,relatime,name=systemd 0 0
    none /sys/fs/fuse/connections fusectl rw,relatime 0 0
    none /sys/kernel/debug debugfs rw,relatime 0 0
    none /sys/kernel/security securityfs rw,relatime 0 0
    none /sys/fs/pstore pstore rw,relatime 0 0
    proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
    /dev/disk/by-uuid/2ed4c374-2ddb-4a75-af24-98df753dbf6d /chest ext4 rw,relatime,discard,data=ordered 0 0
    proc /proc proc rw,nodev,relatime 0 0

chest 폴더에 디렉토리가 아무것도 없는 데 umount로 마운트를 해제해보니 jungle.chest 파일이 있네요. -.-;;

.. code-block:: console

    $ ls /chest
    
    ls /chest

    $ umount /chest

    umount /chest
    
    $ ls -al /chest

    ls -al /chest
    total 12
    drwxr-xr-x  2 nobody nogroup 4096 Oct 16 13:31 .
    drwxr-xr-x 23 nobody nogroup 4096 Oct 16 13:29 ..
    -rw-r--r--  1 nobody nogroup   42 Oct 16 13:08 jungle.chest

파일을 보면 정답이 나오네요. 

.. code-block:: bash

    $ cat /chest/jungle.chest

    cat /chest/jungle.chest
    
    
