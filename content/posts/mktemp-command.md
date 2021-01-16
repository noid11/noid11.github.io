---
title: "mktemp コマンド"
date: 2021-01-16T08:56:33+09:00
toc: true
draft: true
---

- 一時的なファイルやディレクトリを作成する mktemp コマンドについて調べたメモ
- 一時的なディレクトリを作成して cd で移動しつつ作業する・・・といった事がサクッとできて便利

```bash
cd "(mktemp -d)"
```

<!--more-->

## 一時的なファイルを作成する

```bash
mktemp
```

実行例

- `/var/folders/` 配下に一時的な空のファイルが作成される

```bash
$ mktemp
/var/folders/2s/xv1nds5j2d3b795qy43yzcthb7qc6n/T/tmp.9HMytC5L 
% ls -lh /var/folders/2s/xv1nds5j2d3b795qy43yzcthb7qc6n/T/tmp.9HMytC5L 
-rw-------  1 myusername  1896053708     0B Jan 16 09:03 /var/folders/2s/xv1nds5j2d3b795qy43yzcthb7qc6n/T/tmp.9HMytC5L
% cat /var/folders/2s/xv1nds5j2d3b795qy43yzcthb7qc6n/T/tmp.9HMytC5L
```


## 一時的なディレクトリを作成する

```bash
mktemp -d
```

実行例

```bash
% mktemp -d
/var/folders/2s/xv1nds5j2d3b795qy43yzcthb7qc6n/T/tmp.t3PtNiwY
% ls -lh /var/folders/2s/xv1nds5j2d3b795qy43yzcthb7qc6n/T/ | grep t3PtNiwY
drwx------      2 myusername  1896053708    64B Jan 16 09:22 tmp.t3PtNiwY
```


## 生成されるファイルやディレクトリの名前を指定する

- `mktemp [ファイル名].XXXXX` といった形で、任意の X を追加することでランダムな文字列を設定できる

```bash
mktemp mytmp.XXXXX
mktemp -d mytmp.XXXXX
```

実行例

```bash
% mktemp mytmp.XXXXX
mytmp.M2wuV
% mktemp -d mytmp.XXXXX
mytmp.moGrd
```


## シェルスクリプト例

安全な一時ファイルを取得できない場合にスクリプトを終了する例

```sh
tempfoo=`basename $0`
TMPFILE=`mktemp /tmp/${tempfoo}.XXXXXX` || exit 1
echo "program output" >> $TMPFILE
```

実行例

```bash
% sh example.sh
% cat example.sh.kZXaAa
program output
```

---

$TMPDIR を使う

```sh
tempfoo=`basename $0`
TMPFILE=`mktemp -t ${tempfoo}` || exit 1
echo	"program output" >> $TMPFILE
```

実行例

```bash
% sh example.sh
% echo $TMPDIR
/var/folders/2s/xv1nds5j2d3b795qy43yzcthb7qc6n/T/
% ls -l /var/folders/2s/xv1nds5j2d3b795qy43yzcthb7qc6n/T/ | grep example
-rw-------      1 myusername  1896053708        15  1 16 09:43 example.sh.gihepulJ
```

---

エラーハンドリングを考慮する

```sh
tempfoo=`basename $0`
TMPFILE=`mktemp -q /tmp/${tempfoo}.XXXXXX`
if [ $? -ne 0 ]; then
    echo "$0: Can't create temp file, exiting..."
    exit 1
fi
```


## オプション

- `-d` オプション: 一時的なファイルではなく、一時的なディレクトリを作成する
- `-q` オプション: エラーメッセージを表示しない
- `-t` オプション: 指定した prefix と、 TMPDIR が設定されている場合にはそれを使って、テンプレートを作成し、ファイル名テンプレートを作成する
- `-u` オプション: unsafe モードを使用する。一時的なファイルは mktemp コマンドが終了する前に、リンクが解除される。これは mktemp（3）よりもわずかに優れていますが、それでも競合状態が発生するため、このオプションの使用は推奨されていない


## manual

```
% man mktemp
MKTEMP(1)                 BSD General Commands Manual                MKTEMP(1)

NAME
     mktemp -- make temporary file name (unique)

SYNOPSIS
     mktemp [-d] [-q] [-t prefix] [-u] template ...
     mktemp [-d] [-q] [-u] -t prefix

DESCRIPTION
     The mktemp utility takes each of the given file name templates and overwrites a portion of it to create a file name. 
     This file name is unique and suitable for use by the application.  
     The template may be any file name with some number of `Xs' appended to it, for example /tmp/temp.XXXX. 
     The trailing `Xs' are replaced with the current process number and/or a unique letter combination. 
     The number of unique file names mktemp can return depends on the number of `Xs' provided; six `Xs' will result in mktemp selecting 1 of 56800235584 (62 ** 6) possible file names. 
     On case-insensitive file systems, the effective number of unique names is significantly less; given six `Xs', mktemp will instead select 1 of 2176782336 (36 ** 6) possible unique file names.

     If mktemp can successfully generate a unique file name, the file is created with mode 0600 (unless the -u flag is given) and the filename is printed to standard output.

     If the -t prefix option is given, mktemp will generate a template string based on the prefix and the _CS_DARWIN_USER_TEMP_DIR configuration variable if available. 
     Fallback locations if _CS_DARWIN_USER_TEMP_DIR is not available are TMPDIR and /tmp.  
     Care should be taken to ensure that it is appropriate to use an environment variable potentially supplied by the user.

     If no arguments are passed or if only the -d flag is passed mktemp behaves as if -t tmp was supplied.

     Any number of temporary files may be created in a single invocation, including one based on the internal template resulting from the -t flag.

     The mktemp utility is provided to allow shell scripts to safely use temporary files.  
     Traditionally, many shell scripts take the name of the program with the pid as a suffix and use that as a temporary file name.  
     This kind of naming scheme is predictable and the race condition it creates is easy for an attacker to win. 
     A safer, though still inferior, approach is to make a temporary directory using the same naming scheme.  
     While this does allow one to guarantee that a temporary file will not be subverted, it still allows a simple denial of service attack.  
     For these reasons it is suggested that mktemp be used instead.

OPTIONS
     The available options are as follows:

     -d      Make a directory instead of a file.

     -q      Fail silently if an error occurs.  This is useful if a script does not want error output to go to standard error.

     -t prefix
             Generate a template (using the supplied prefix and TMPDIR if set) to create a filename template.

     -u      Operate in ``unsafe'' mode.  The temp file will be unlinked before mktemp exits.  This is slightly better than mktemp(3) but still introduces
             a race condition.  Use of this option is not encouraged.

EXIT STATUS
     The mktemp utility exits 0 on success, and >0 if an error occurs.

EXAMPLES
     The following sh(1) fragment illustrates a simple use of mktemp where the script should quit if it cannot get a safe temporary file.

           tempfoo=`basename $0`
           TMPFILE=`mktemp /tmp/${tempfoo}.XXXXXX` || exit 1
           echo "program output" >> $TMPFILE

     To allow the use of $TMPDIR:

           tempfoo=`basename $0`
           TMPFILE=`mktemp -t ${tempfoo}` || exit 1
           echo "program output" >> $TMPFILE

     In this case, we want the script to catch the error itself.

           tempfoo=`basename $0`
           TMPFILE=`mktemp -q /tmp/${tempfoo}.XXXXXX`
           if [ $? -ne 0 ]; then
                   echo "$0: Can't create temp file, exiting..."
                   exit 1
           fi

SEE ALSO
     mkdtemp(3), mkstemp(3), mktemp(3), confstr(3), environ(7)

HISTORY
     A mktemp utility appeared in OpenBSD 2.1.  This implementation was written independently based on the OpenBSD man page, and first appeared in
     FreeBSD 2.2.7.  This man page is taken from OpenBSD.

BSD                            December 30, 2005                           BSD
```


## この記事を試した環境

```bash
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
```
