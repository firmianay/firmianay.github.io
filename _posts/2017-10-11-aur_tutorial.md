---
layout: post
title:  "AUR 入门"
image: ''
date:   2017-10-11 00:06:31
tags:
- linux
description: ''
categories:
- linux
---

- [什么是 AUR](#什么是-aur)
- [安装 AUR 中的软件包](#安装-aur-中的软件包)
- [创建一个软件包](#创建一个软件包)
- [提交软件包到 AUR](#提交软件包到-aur)


用 Arch Linux 已经一年左右了，有事没事滚一滚，虽然不时会有些小毛病，自己动手修一修甚至忍一忍也就过了。毕竟在它强大的社区和包管理面前，这都不是事。前几天 pwntools 的 AUR 包维护者由于一些原因不再及时更新包，我就将维护的工作接了过来。pwntools 是我最喜爱的工具之一，也经常使用 yaourt 直接从 AUR 进行安装，继续维护它既能方便自己，也是为开源做一点贡献。

由于我之前从来没有包维护的经验，对它的原理和操作都不熟悉，好在 archwiki 很完善，很快就能上手了。这篇文章会详细介绍 AUR，包括安装、创建和提交软件包的全过程。

## 什么是 AUR
在介绍 AUR 之前，你需要先知道 Arch Linux 的一些基础知识。Arch Linux 使用 Pacman 作为包管理器，它在提供了一个简单的包管理器的同时，页提供了一个易用的包构建系统，使用户能够轻松地管理和定制官方提供的、用户自己制作的、甚至是来自第三方的各种软件包。Arch Linux 的基本安装包由“core"提供，此外”extra“、”community“和”testing“软件库则提供了大量的高品质软件以满足你的需求。

AUR(Arch User Repository)也是一个软件仓库，但它是由用户主导的非官方的仓库，AUR 中的软件包以软件包生成脚本（PKGBUILD）的形式提供，用户自己通过 makepkg 生成包，再由 pacman 安装。创建 AUR 的初衷是方便用户维护和分享新软件包，并由官方定期从中挑选优质的软件包进入 community 仓库。通过 AUR，大家可以互相分享软件包生成脚本、为软件包投票、评论等等。


## 安装 AUR 中的软件包
#### 准备工作
要想进行下面的工作，你首先要会使用终端和熟悉 bash 脚本。

然后，安装 base-devel 软件包组，里面包含了编译软件包所需的工具。
```
$ sudo pacman -S --needed base-devel
```
当然，如果你还没有安装 yaourt 的话，它是一个 AUR helper，使用与 pacman 完全相同的语法，能够自动化软件包的编译和安装：
```
$ sudo pacman -S yaourt
```

最后需要提醒的是，AUR 中的软件包是由用户上次的，使用时需要自行承担风险。

#### 获取软件包构建所需文件
首先到 [AUR 网站](https://aur.archlinux.org)上找到你需要的软件包，点击它即可查看详细信息，如 pwntools。（建议注册一个账户）。

![](/post_pic/2017-10-11-aur_tutorial/aur_1.png)

![](/post_pic/2017-10-11-aur_tutorial/aur_2.png)

点击右侧的”View PKGBUILD“可查看软件包生成脚本，点击”Download snapshot“可下载软件包的快照，是一个包含了 `PKGBUILD` 和其他安装文件的 tar 包。

解压：
```
$ tar -zxvf python2-pwntools.tar.gz
```
查看一下软件包目录下的文件：
```
$ cd python2-pwntools/
$ ls -al
total 16
drwxr-xr-x 2 firmy firmy 4096 10月  8 20:52 .
drwxr-xr-x 9 firmy firmy 4096 10月  9 16:33 ..
-rw-r--r-- 1 firmy firmy 1348 10月  8 20:52 PKGBUILD
-rw-r--r-- 1 firmy firmy 1021 10月  8 20:52 .SRCINFO
```
`PKGBUILD` 是软件包生成脚本，而 `.SRCINFO` 中记录了软件包的元数据，通过它，AUR 网站后台或 AUR 工具可以不用解析 `PKGBUILD` 就获取到需要的信息。当我们编写好 `PKGBUILD` 后，可以使用 makepkg 自动生成 `.SRCINFO` 文件：
```
$ makepkg --printsrcinfo > .SRCINFO
```

当然还有多种方法可以下载所需文件：

- 从终端下载：
```
curl -L -O https://aur.archlinux.org/cgit/aur.git/snapshot/python2-pwntools.tar.gz
```
- 下载 Git 仓库：
```
git clone https://aur.archlinux.org/python2-pwntools.git
```

再次强调 AUR 可能是危险的，请务必仔细检查里面的所有文件！现在我们回到安装过程中来，在检查完成后，就可以运行 makepkg 来创建并安装软件包：
```
$ makepkg -rsi
```
> `-r`：在安装完成后删除只在编译时需要的软件包
>
> `-s`：使用 pacman 安装编译时需要的所有依赖
>
> `-i`：编译完成后安装软件包

然后软件包就安装好了：
```
$ pacman -Qi python2-pwntools
Name            : python2-pwntools
Version         : 3.9.0-2
Description     : A CTF framework and exploit development library.
Architecture    : any
URL             : https://github.com/Gallopsled/pwntools
Licenses        : MIT  GPL2  BSD
Groups          : None
Provides        : None
Depends On      : python2>=2.7  python2-mako  python2-paramiko  python2-pyelftools
                  python2-capstone  python2-pyserial  python2-requests  python2-psutil  python2-tox
                  python2-pysocks  python2-dateutil  python2-pygments  python2-pypandoc
                  python2-packaging  python2-unicorn  python2-intervaltree  python2-pip  ropgadget
Optional Deps   : None
Required By     : None
Optional For    : None
Conflicts With  : python2-pwntools  python2-pwntools-git
Replaces        : None
Installed Size  : 32.02 MiB
Packager        : Unknown Packager
Build Date      : 2017年10月09日 星期一 17时03分27秒
Install Date    : 2017年10月09日 星期一 17时03分46秒
Install Reason  : Explicitly installed
Install Script  : No
Validated By    : None
```

当然了，使用 yaourt 就可以免去这些手动操作的麻烦，你只需要运行下面的命令：
```
$ yaourt -S python2-pwntools
```


## 创建一个软件包
Arch Linux 中的软件包是通过 `makepkg` 工具以及存储在 `PKGBUILD` 文件中的信息编译的。下面我们会详细介绍它。

在我们刚刚运行过 `makepkg -rsi` 的目录下，有下面的文件：
```
$ ls -al
total 2540
drwxr-xr-x 5 firmy firmy    4096 10月  9 17:03 .
drwxr-xr-x 3 firmy firmy    4096 10月  9 16:58 ..
-rw-r--r-- 1 firmy firmy 1124598 10月  9 17:01 3.9.0.tar.gz
drwxr-xr-x 3 firmy firmy    4096 10月  9 17:01 pkg
-rw-r--r-- 1 firmy firmy    1348 10月  9 16:59 PKGBUILD
-rw-r--r-- 1 firmy firmy 1443856 10月  9 17:03 python2-pwntools-3.9.0-2-any.pkg.tar.xz
drwxr-xr-x 3 firmy firmy    4096 10月  9 17:01 src
-rw-r--r-- 1 firmy firmy    1021 10月  9 16:59 .SRCINFO
$ ls -al pkg/python2-pwntools/
total 540
drwxr-xr-x 3 firmy firmy   4096 10月  9 17:03 .
drwxr-xr-x 3 firmy firmy   4096 10月  9 17:01 ..
-rw-r--r-- 1 firmy firmy  44459 10月  9 17:03 .BUILDINFO
-rw-r--r-- 1 firmy firmy 490121 10月  9 17:03 .MTREE
-rw-r--r-- 1 firmy firmy    922 10月  9 17:03 .PKGINFO
drwxr-xr-x 5 firmy firmy   4096 10月  9 17:02 usr
```
一个 Arch 软件包就是一个使用 xz 压缩的 tar 包，它包含了由 `makepkg` 生成的文件：
- `.PKGINFO`：包含了所有 pacman 处理软件包的元数据、依赖等待
- `.MTREE`：包含了文件的哈希值和时间戳，pacman 根据这些信息校验软件包的完整性
- `.BUILDINFO`

通过这些文件我们不难知道 `makepkg` 做了哪些事情。
1. 检查相关依赖是否安装
2. 从指定的服务器下载源文件
3. 解压源文件
4. 编译源文件并将它安装于伪root环境下
5. 删除二进制文件和库文件的符号链接
6. 生成包的meta文件
7. 将伪root环境压缩成一个包文件
8. 将生成的包文件保存到配置的文件夹中（默认为当前工作目录）

#### 普通方式安装
在编写 PKGBUILD 之前，要先尝试以软件作者的方式安装软件包，记录下过程中需要的所有步骤和命令，后面会使用到它们。

一个典型的例子是：
```
./configure
make
make install
```

#### 编写 PKGBUILD
如果软件包能够正常安装，接下来就可以编写 PKGBUILD 文件了。PKGBUILD 是一个 shell 脚本，包含了构建软件包时需要的信息，当运行 makepkg 时，它会在当前目录寻找 `PKGBUILD` 文件，并按照其中的指令去获取依赖文件，编译出 `pkgname.pkg.tar.xz` 文件，该文件可以使用 pacman 进行安装(`pacman -U`)。`pkgname`、`pkgver`、`pkgrel` 和 `arch` 是必须包含的变量，`license` 在分享时也是需要的。

我们通过 pwntools 的 PKGBUILD 文件来学习吧，它非常简单，但包含了所有必须的元素：
```
# Maintainer: Firmy <firmianay@gmail.com>
pkgname=python2-pwntools
pkgver=3.9.0
pkgrel=2
pkgdesc='A CTF framework and exploit development library.'
arch=('any')
url='https://github.com/Gallopsled/pwntools'
license=('MIT' 'GPL2' 'BSD')
makedepends=('lib32-glibc'
             'python2-setuptools')
depends=('python2>=2.7'
         'python2-mako'
         'python2-paramiko'
         'python2-pyelftools'
         'python2-capstone'
         'python2-pyserial'
         'python2-requests'
         'python2-psutil'
         'python2-tox'
         'python2-pysocks'
         'python2-dateutil'
         'python2-pygments'
         'python2-pypandoc'
         'python2-packaging'
         'python2-unicorn'
         'python2-intervaltree'
         'python2-pip'
         'ropgadget')
conflicts=('python2-pwntools' 'python2-pwntools-git')
options=('strip')
source=("https://github.com/Gallopsled/pwntools/archive/${pkgver}.tar.gz")
sha256sums=('bc62c6ae0ac0e9ea14e660b4a603355a53ca3134c1ab90b654e618eb2be6df5b')

_repodir="pwntools-${pkgver}"

prepare() {
  cd ${srcdir}/${_repodir}
}

package() {
  cd ${srcdir}/${_repodir}
  python2 setup.py install --root=${pkgdir}/ --optimize=1 --only-use-pwn-command
  install -D -m 644 LICENSE-pwntools.txt ${pkgdir}/usr/share/licenses/${pkgname}/LICENSE
  rm ${pkgdir}/usr/lib/python*/site-packages/*.{txt,md}
}
```
文件中包含的变量：
- `pkgname`：软件包的名称
- `pkgver`：软件包的版本号，应该与软件原作者发布的版本号一致
- `pkgrel`：发布号，用来区分同一版本软件的多次构建，当调整和优化 PKGBUILD 时，发布号加 1，而当版本号变化时，发布号重置为 1
- `pkgdesc`：软件包描述
- `arch`：表示了 PKGBUILD 可以编译和使用的架构
- `url`：软件官网
- `license`：软件发布许可证
- `makedepends`：仅在编译时需要的软件包列表
- `depends`：软件的运行时依赖列表
- `conflicts`：与当前软件包发生冲突的包列表
- `options`：该变量允许你重置 `makepkg` 的部分默认行为
- `source`：构建软件包时需要的文件列表
- `sha256sums`：校验和

另外还可以看到 makepkg 定义的变量：
- `srcdir`：makepkg 会将源文件解压到此文件夹或在此文件夹中生成指向 PKGBUILD 里 source 数组中文件的软连接
- `pkgdir`：makepkg 会将该文件夹当成系统根目录，并将软件安装在此文件夹下

PKGBUILD 一共有五个函数，`pkgver()`、`prepare()`、`build()`、`check()` 和 `package()`，其中 `package()` 是必须的，该文件中使用了其中的两个：
- `prepage()`：在该函数中，那些用于处理源文件以进行构建的命令会被执行
- `package()`：最后一步是把编译好的文件放到 `pkg` 文件夹（一个伪root环境）中。


## 提交软件包到 AUR
现在我们已经有创建了一个软件包，那么就可以将它上传到 AUR 中分享。

首先是注册一个 AUR 账户，由于 AUR 使用 Git 来管理，所以我们需要为 AUR 生成一个 SSH 密钥。[注册地址](https://aur.archlinux.org/register)
```
$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/firmy/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/firmy/.ssh/id_rsa.
Your public key has been saved in /home/firmy/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:g1uzMJEnSzhftdF47LawMO8e+YvdfaWG7JZHY9QrAl4 firmy@firmy-pc
The key's randomart image is:
+---[RSA 2048]----+
|          o+     |
|     . . ..o+    |
|    o = o .o   . |
|     + Bo..Eo . .|
|      * S+o+ o  .|
|       = =oo..+..|
|      . ..o..=.o.|
|          .==.+..|
|         .oo=+. o|
+----[SHA256]-----+

$ ls .ssh/
id_rsa  id_rsa.pub
[firmy@firmy-pc ~]$ cat .ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDFmUEMYewe8Wwv4EHcNQoLuqptuxwQGJglSb1mFLnAxG3Hb/rsjaix6A5EvzIMAg7h0HyEo2T5b1D/Dt2EZUeVuqsm62bXhCr2CCYlfqzxPlHOBk7btlQNYKNsO1gwroG3sLM3jK8E/gJfcRbtM40Sce1jma8z0jW7pJGq3KgeBe936WXTVcEi6tLfzKkHawN7N3AZPRORUe0vAa82mBMTX7jCixMLw9+48Jh+NjpdH3gHypv1RxQ2PXBWl69A8KUKGakYomr3feARdeYTMAjdzexl2ElTkJ8dgdEuUu9F0yEP5M7Ea61Tvzasl7nR04G2hQMnsyFNWPZVTvjboYrt firmy@firmy-pc
```
复制公钥到 AUR Web 界面”My Account“中的”SSH Public Key“中。

接下来编辑文件 `~/.ssh/config`，告诉 SSH 使用我们创建的密钥进行连接。
```
Host aur.archlinux.org
    IdentityFile ~/.ssh/id_rsa  # 你的私钥路径
    User Firmy  # 你的账户名
```

然后就可以创建新仓库了，如pwntools，但记得要使用 ssh 而不是 https：
```
$ git clone ssh://aur@aur.archlinux.org/python2-pwntools.git
```
修改 PKGBUILD 后一定要记得生成 `.SRCINFO`，验证无误后即可提交到 AUR：
```
$ git add .
$ git commit -m "update xxx"
$ git push
```

到这里，你已经成功创建了一个软件包并上传分享。但事情还没有结束，你还需要对软件包进行维护，当上游更新了，就要将及时的更新上传，当有人评论或提出建议，就要试着改进。当然了，如果你不想再维护该软件包，可以 disown 它。


## 参考资料
- [Arch User Repository](https://wiki.archlinux.org/index.php/Arch_User_Repository)
- [Creating packages](https://wiki.archlinux.org/index.php/Creating_packages)
- [PKGBUILD](https://wiki.archlinux.org/index.php/PKGBUILD)
- [makepkg](https://wiki.archlinux.org/index.php/Makepkg)
- [Arch packaging standards](https://wiki.archlinux.org/index.php/Arch_packaging_standards)
