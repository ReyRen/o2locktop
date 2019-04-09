
# o2locktop - a top-like OCFS2 DLM lock monitor

## Introduction

o2locktop is a top-like tool to monitor OCFS2 DLM lock usage in the cluster,
and can be used to detect hot files/directories, which intensively acquire DLM
locks.

The average/maximal wait time for DLM lock acquisitions likely gives hints to
the administrator when concern about OCFS2 performance, for example,
- if the workload is unbalanced among nodes.
- if a file is too hot, then maybe need check the related applications above.
- if a directory is too hot, then maybe split it to smaller with less number
  of files underneath.

For slightly more implementation details, as a shared disk cluster file system,
OCFS2 files and directies can be accessed from the different nodes
simultaneously. To protect the data consistency, the file access is coordinated
through Distributed Lock Manager(DLM). For example, "Meta DLM lock" is used to
protect file(per inode) meta data change. "Write DLM lock" is used to protect
file data write. "Open DLM lock" could be used for one node keeps accessing a
opened file while other nodes might delete it, and eventually get deleted once
all associated file descriptors are close. FIXME: ??? For more information
about how OCFS2 works with DLM, please check [OCFS2 Project web page][OCFS2_wiki].


o2locktop reads OCFS2 kernel debugfs statistics under /sys/kernel/debug/.
That says, for all cluster nodes, OCFS2_FS_STATS kernel config option must be
set(enabled). To check it out:

```shell
grep OCFS2_FS_STATS < /boot/config-`uname -r`
```

## Installation

Note: o2locktop is Python 2 and Python 3 compatible.

- RPM:

  https://download.opensuse.org/repositories/network:/ha-clustering:/Factory/

  Download openSUSE_Tumbleweed/noarch/o2locktop-1.0.0...noarch.rpm

```shell
  sudo zypper install <http_rpm_uri>
  or
  sudo rpm -ivh <o2locktop-1.0.0...noarch.rpm>
```

- Python pip:

```shell
  sudo pip install o2locktop
```

- non-root users can use o2locktop directly from the source code tree:

```shell
  git clone https://github.com/ganghe/o2locktop.git
  cd o2locktop 
  ~/o2locktop> o2locktop -h
```

  FIXME:
  sudo python setup.py install  ?????
  Comments: ?????
  - can we avoid 'sudo', ROOT is not flexible enough
  - Should be directly useable

## Usage

- Check o2locktop help text in details, also availble in the below [REFERENCE](#reference)
```shell
o2locktop -h
```
- Or, check the asciidemo [here][o2locktop_demo]

- Known limitations

  1. Since OCFS2 file system statistics in kernel calculation starts when
     applying for DLM lock and ends when it returns. If it never returns due to
the deadlock because of a bug just in case, o2locktop does not reflect this
situation currently.

  2. o2locktop can't display the file names of the inode. The additional step
     is needed to translate inode to the file name.
```shell
     find /mountpoint -inum ino
```

## Unit Test

  FIXME: ...

## Development

```shell
git clone https://github.com/ganghe/o2locktop
```

### TODO

- Replay o2locktop log file.  
- Inside of the cluster, o2lockto can run withoug any argument.
  It can automatically discover all those arguments.

## Community

* Report bugs at the [o2locktop issues @ Github.com](https://github.com/ganghe/o2locktop/issues) page.
* Contact [OCFS2 developer mailing list](https://oss.oracle.com/mailman/listinfo/ocfs2-devel)




[OCFS2_wiki]: https://ocfs2.wiki.kernel.org
[o2locktop_demo]: https://asciinema.org/a/fktChiXJpLGL8Z3WaoWDaXLE2  


REFERENCE
---------

usage: o2locktop [-h] [-n NODE_IP] [-o LOG_FILE] [-l DISPLAY_LENGTH] [-V] [-d]
                 [MOUNT_POINT]

It is a top-like tool to monitor OCFS2 DLM lock usage in the cluster, and can
be used to detect hot files/directories, which intensively acquire DLM locks.

positional arguments:
  MOUNT_POINT        ocfs2 mount point, eg. /mnt

optional arguments:
  -h, --help         show this help message and exit
  -n NODE_IP         OCFS2 node IP address for ssh
  -o LOG_FILE        log path
  -l DISPLAY_LENGTH  number of lock records to display, the default is 15
  -V, --version      the current version of o2locktop
  -d, --debug        show all the inode including the system inode number

The average/maximal wait time for DLM lock acquisitions likely gives hints to
the administrator when concern about OCFS2 performance, for example,
- if the workload is unbalanced among nodes.
- if a file is too hot, then maybe need check the related applications above.
- if a directory is too hot, then maybe split it to smaller with less number
  of files underneath.

PREREQUISITES:
  o2locktop reads OCFS2 kernel debugfs statistics under /sys/kernel/debug/.
  That says, for all cluster nodes, OCFS2_FS_STATS kernel config option must be
  set(enabled). To check it out:
      grep OCFS2_FS_STATS < /boot/config-\`uname -r\`

  o2lockto uses the passwordless SSH to access OCFS2 nodes. Set it up if not:
      ssh-keygen; ssh-copy-id root@node1

OUTPUT ANNOTATION:
  - The output is refreshed every 5 seconds, and sorted by the sum of 
    DLM EX(exclusive) and PR(protected read) lock average wait time
  - One row, one inode (including the system meta files if with '-d' argument)
  - Columns:
    "TYPE" is DLM lock types,
      'M' -> Meta data lock for the inode
      'W' -> Write lock for the inode
      'O' -> Open lock for the inode

    "INO" is the inode number of the file

    "EX NUM" is the number of EX lock acquisitions
    "EX TIME" is the maximal wait time to get EX lock
    "EX AVG" is the average wait time to get EX lock

    "PR NUM" is the number of PR(read) lock acquisitions
    "PR TIME" is the maximal wait time to get PR lock
    "PR AVG" is the average wait time to get PR lock

SHORTCUTS:
  - Type "d" to display DLM lock statistics for each node
  - Type "Ctrl+C" or "q" to exit o2locktop process

EXAMPLES:
  - At any machine within or outside of the cluster:

    o2locktop -n node1 -n node2 -n node3 /mnt/shared

    To find the absolute path of the inode file:
    find <MOUNT_POINT> -inum <INO>
 
