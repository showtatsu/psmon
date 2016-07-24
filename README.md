# psmon

### What is it ?

psmonは「プロセス監視デーモン」です。
net-snmpのprocでは面倒を見切れない環境において、プロセス監視機能を強化するために使用します。
指定した条件でプロセス数を常時カウントし、閾値を超えるとsyslog経由で通報を行います。

### What is required ?

- ps ( GNU/Linux Compatible )
- perl
  - 5.8, or later in 5.x
  - It's corelist packages.
- logger

### How to install ?

#### install
```bash
sudo su -
INSTALL_DIR=/usr/local/psmon
git clone https://github.com/showtatsu/psmon $INSTALL_DIR

ln -s $INSTALL_DIR/psmon /usr/local/bin/psmon
ln -s $INSTALL_DIR/conf.d /etc/psmon.d
ln -s $INSTALL_DIR/init.sysv /etc/init.d/psmon
```

#### test run (enable trace out)
```bash
/usr/local/bin/psmon -t
```

#### start
```bash
sudo su -
chkconfig psmon on
service psmon start
```

### Configurations

設定ファイルは ``` /etc/psmon.d ``` 配下に置きます。
設定ファイルは "=" 区切りテキストです。サンプルは ./sample.conf.d を見てください。
"." 始まりのファイル名(隠しファイル)は読み込まれません。

```
name  = vim
max   = 3
min   = 1
warn  = 2
match = vim
memo  = monitor vim
```

#### 通報に関する設定項目
- name
  - 必須項目です。この設定にユニークな名前をつけます。この名前は通報に使用されます。
- max
  - 検知されたプロセス数がこの値を超えた時に「Major Alerm」を発生させます。「0」は非監視。
- min
  - 検知されたプロセス数がこの値を下回った時に「Major Alerm」を発生させます。「0」は非監視。
- warning
  - 検知されたプロセス数がこの値を超えた時に「Minor Alerm」を発生させます。「0」は非監視。

#### プロセス検索に関する設定項目
プロセス検索に関する定義です。以下の項目のうち、少なくとも1つを設定する必要があります。
1つの監視ファイルに対して複数の検索条件を指定した場合は論理積として扱われます。

- proc
  - net-snmpのprocと同じく、 ``` ps -e ``` の出力と完全一致するプロセスを数えます。
- match
  - ``` ps -eo cmdline ``` の出力に対して、指定した正規表現がマッチするプロセスを数えます。
- pidfile
  - 指定したファイルに書かれたPIDと一致するプロセスを数えます。
  - この項目を設定すると検知されるプロセス数は1を超えなくなります。
- ppidfile
  - 指定したファイルに書かれたPIDをPPID(親プロセスID)として持つプロセスを数えます。
  - 特定のデーモンが起動した子プロセス数を数えることができます。
  - 孫プロセスは検索対象になりません。
- user
  - 指定したユーザ名(完全一致)で起動したプロセスを数えます。
- group
  - 指定したグループ名(完全一致)で起動したプロセスを数えます。


### psmon自体の起動オプション

設定変更はinitファイルを直接書き換える必要があります。
デフォルトで initスクリプトに設定されているのは下記です。

``` -d -s -i 10 -c /etc/psmon.d ```

- -d
  - psmon を無限ループで起動します。
  - このオプションを指定しないとpsmonは一度のチェックを行った後すぐに終了します。

- -c /path/to/confdir
  - 設定ファイルの格納ディレクトリを指定します。
  - 標準では、/etc/psmon.d が使用されます。

- -i 10
  - チェックインターバルを秒数で指定します。デフォルトでは 10秒毎です。

- -s
  - 状態変化時にloggerを用いてsyslogへの通報を有効にします。

- -t
  - Trace出力を有効にします。
  - このオプションを指定しない場合、psmonは標準出力への情報をほとんど出しません。

- -D 10
  - 実行後指定回数ループした後にpsmonを終了させます
  - 実行中にSIGTERMを受け取った際と動作は変わりません
  - NYTProfなどでパフォーマンスチェックを行う場合などに便利です
    - 確実に指定回数ループさせた後、正常終了させることができます

### Support and Copyright

- Tatsuya SHORIKI <show.tatsu.devel@gmail.com>, 2016

