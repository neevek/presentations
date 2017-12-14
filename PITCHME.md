@title[介绍]

# Dive into Git

#### 彻底理解 Git

<br/>
###### neevek / 谢佳敏
###### 2017.12.14

---
@title[理解Git对象]

# 理解 Git 对象

@fa[arrow-down]

+++

@title[Key-value store]

- Git 是一个键值存储（Key-value store），每一个对象由一个 40 个字符的 SHA-1 Hash 确定。
- 往 Git 仓库添加文件、提交或者打 tag 的背后是创建一系列不同类型的对象。

+++

@title[伪代码]

就像这样...

```python
type = 'blob'
data = readfile("file.txt")
sha1_hash = sha1(data)                  # the key
compressed_data = zlib_compress(data)   # the value
# put the data in the git repo
git_repo.put(type, sha1_hash, compressed_data)
```

+++

### Blob

一个文件，其内容是文件的内容

+++

### Tree

一个目录，其内容可包含 Blob 对象或者嵌套其他 Tree 对象的描述信息

+++

### Commit

一个指向 Tree 的指针，其内容包含 Tree 的 hash、author 和 timestamp 等信息。

+++

### Tag（Annotated）

一个标记，用于给以上三种对象“命名”（打版本号），多数情况下用在 Commit 上。其内容包含任意其他三种对象的 hash 和 description 等信息。

+++

@title[对象图]

![git-objects](assets/git-objects1.png)

---

## 手动创建一次提交

不使用任何 git 命令创建一个 git 仓库并做一次提交

@fa[arrow-down]

+++

@title[一个简单的 git 仓库]

一个简单的 git 仓库

```shell
.git
├── COMMIT_EDITMSG
├── HEAD
├── config
├── description
├── hooks
├── index
├── info
│   └── exclude
├── logs
│   ├── HEAD
│   └── refs
│       └── heads
│           └── master
├── objects
│   ├── aa
│   │   └── a96ced2d9a1c8e72c56b253a0e2fe78393feb7
│   ├── ce
│   │   └── 013625030ba8dba906f756967f9e9ca394464a
│   ├── d4
│   │   └── 406279bd35b4a9dac692c257e3314c81aa8796
│   ├── info
│   └── pack
└── refs
    ├── heads
    │   └── master
    └── tags
```

+++

@title[一个最简单的 git 仓库]

一个最简单的 git 仓库

```shell
$ tree .git
.git
├── HEAD
├── objects
└── refs
    └── heads

3 directories, 1 file

$ cat .git/HEAD
ref: refs/heads/master
```

+++

@title[手动创建 git 对象]

手动创建 git 对象

```perl
#!/usr/bin/perl -w 
use warnings;
use strict;
use v5.14;

use Compress::Zlib;
use Digest::SHA1 qw(sha1_hex);
use File::Path qw(make_path);

sub usage {
  say("usage: handcraft_git_object.pl <type> [options...]");
  say(" e.g.:");
  say("       handcraft_git_object.pl blob 'content'");
  say("       handcraft_git_object.pl tree BLOB_HASH /path/to/file");
  say("       handcraft_git_object.pl commit TREE_HASH 'commit message'");
}

sub check_args_count {
  my $count = shift;

  if (($#ARGV + 1) < $count) {
    usage();
    exit(1);
  }
}

check_args_count(1);

my $data = '';
my $type = $ARGV[0];
my $switch = {
  'blob' => sub {
    check_args_count(2);

    my $content = $ARGV[1];
    $data = "blob " . length($content) . "\0" . $content;
  },

  'tree' => sub {
    check_args_count(3);

    my $hash = $ARGV[1];
    my $filename = $ARGV[2];
    my $content = "100644 " . $filename . "\0" . pack("H*", $hash);
    $data = "tree " . length($content) . "\0" . $content;
  },

  'commit' => sub {
    check_args_count(3);

    my $hash = $ARGV[1];
    my $message = $ARGV[2];
    my $content = 'tree ' . $hash . "\n" .
      'author neevek <jiamin.xjm@alibaba-inc.com> ' . time() . " +0800\n" .
      'committer neevek <jiamin.xjm@alibaba-inc.com> ' . time() . " +0800\n\n" .
      $message;
    $data = "commit " . length($content) . "\0" . $content;
  },

  'default' => sub {
    say "incorrect type: ". $type;
    usage();
    exit(1);
  },
};

($switch->{$type} or $switch->{'default'})->();

my $sha1 = sha1_hex($data);
my $dir = ".git/objects/". substr($sha1, 0, 2);
make_path($dir);

my $blobfile= "$dir/" . substr($sha1, 2);
open my $fh, ">:raw", $blobfile or
  die "Failed to write to '$blobfile' - $!";
print $fh compress($data);
close $fh;

print "$sha1\n";
```

@[69-71](使用 blob 对象的 hash 的前两个字符创建一个目录)
@[73-77](把 blob 对象的内容使用 zlib 压缩并且保存至以其 hash 的后 38 个字符命名的文件)
@[32-37](拼接 blob 对象的内容)
@[39-46](拼接 tree 对象的内容)
@[48-58](拼接 commit 对象的内容)

---

## 理解引用

---

## 常用与不常用但很有用的命令

@fa[arrow-down]

+++

@title[Useful Commands]

### log, reset, cherry-pick, cherry, bisect, fetch, merge, rebase, diff, difftool, reflog, worktree


---

## Git 工作流演示

---

## Thanks.