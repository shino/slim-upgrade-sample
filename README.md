# Sample to Upgrade slim release, which does not work...

In upgrading, an error occurs.

Environment:
- R16B02 used both for generating slim release and for boot it
- `rebar` from `slim-release-support` branch of shino's fork at commit: `eafc1bd`
  https://github.com/shino/rebar/tree/slim-release-support
  (included in this repository as binary)

## Step 1

Generate version 1 and 2 of releases and upgrade package between them.
The script `generate.sh` does this work.

```
$ ./generate.sh
Clean up working directory...
Set up application version 1 and generate release...
==> sample (create-app)
Writing src/sample.app.src
Writing src/sample_app.erl
Writing src/sample_sup.erl
==> sample (compile)
Compiled src/sample_app.erl
Compiled src/sample_sup.erl
==> rel (create-node)
Writing reltool.config
Writing files/erl
Writing files/nodetool
Writing files/sample
Writing files/sys.config
Writing files/vm.args
Writing files/sample.cmd
Writing files/start_erl.cmd
Writing files/install_upgrade.escript
==> rel (generate)
Change version to 2 and generate release again...
--- sample.app.src.v1   2013-10-26 23:57:59.158624896 +0900
+++ src/sample.app.src  2013-10-26 23:57:59.158624896 +0900
@@ -2,5 +2,5 @@
  [
   {description, ""},
-  {vsn, "1"},
+  {vsn, "2"},
   {registered, []},
   {applications, [
==> sample (clean)
==> sample (compile)
Compiled src/sample_app.erl
Compiled src/sample_sup.erl
--- reltool.config.v1   2013-10-26 23:58:01.074549078 +0900
+++ reltool.config      2013-10-26 23:58:01.074549078 +0900
@@ -3,5 +3,5 @@
        {erts, [{mod_cond, derived}, {app_file, strip}]},
        {app_file, strip},
-       {rel, "sample", "1",
+       {rel, "sample", "2",
         [
          kernel,
==> rel (generate)
Generate appup and upgrade...
==> rel (generate-appups)
Generated appup for sample
Appup generation complete
==> rel (generate-upgrade)
sample_2 upgrade package created
```

## Step 2

Before booting node, take a look of `ls` at the Erlang/OTP releases directory to be used.

```
$ ls -l /opt/erlang/R16B02_default/lib/erlang/releases
total 16
drwxrwxr-x 2 shino 4096 Sep 18 23:06 R16B02
-rw-rw-r-- 1 shino  329 Sep 18 23:06 RELEASES
-rw-rw-r-- 1 shino  248 Sep 18 23:06 RELEASES.src
-rw-rw-r-- 1 shino   14 Sep 18 23:06 start_erl.data
```

Then, boot the node with release version 1 and try to unpack the upgrade package.
(Line breaks are inserted properly for `Exec:`)

```
$ ./sample/rel/sample_1/bin/sample console
Exec: /opt/erlang/R16B02_default/bin/../lib/erlang/erts-5.10.3/bin/erlexec
 -boot_var RELTOOL_EXT_LIB /home/shino/work/git/slim-upgrade-sample/sample/rel/sample_1/lib
 -sasl releases_dir "/home/shino/work/git/slim-upgrade-sample/sample/rel/sample_1/releases"
 -boot /home/shino/work/git/slim-upgrade-sample/sample/rel/sample_1/releases/1/sample
 -mode embedded
 -config /home/shino/work/git/slim-upgrade-sample/sample/rel/sample_1/releases/1/sys.config
 -args_file /home/shino/work/git/slim-upgrade-sample/sample/rel/sample_1/releases/1/vm.args
 -- console
Root: /opt/erlang/R16B02_default/bin/../lib/erlang
Erlang R16B02 (erts-5.10.3) [source] [64-bit] [smp:8:8] [async-threads:10] [kernel-poll:false] [systemtap]

Eshell V5.10.3  (abort with ^G)
(sample@127.0.0.1)1> release_handler:unpack_release("sample_2").
{error,{enoent,"/home/shino/work/git/slim-upgrade-sample/sample/rel/sample_1/releases/sample_2.rel"}}
```

Error occured. It says `rel` file does not found.

Again, `ls` at the Erlang/OTP releases directory, so `sample_2.rel` is seen there.

```
(py27_base)shino@shino-xub-vbox$ ls -l /opt/erlang/R16B02_default/lib/erlang/releases
total 20
drwxrwxr-x 2 shino 4096 Sep 18 23:06 R16B02
-rw-rw-r-- 1 shino  329 Sep 18 23:06 RELEASES
-rw-rw-r-- 1 shino  248 Sep 18 23:06 RELEASES.src
-rw-rw-r-- 1 shino  304 Oct 26 23:58 sample_2.rel
-rw-rw-r-- 1 shino   14 Sep 18 23:06 start_erl.data
```

## Some observations

Look into `lib/sasl/src/release_handler.erl`
https://github.com/erlang/otp/blob/OTP_R16B02/lib/sasl/src/release_handler.erl

- `extract_rel_file/2`: Extract rel file in tar ball under root directory.
  https://github.com/erlang/otp/blob/OTP_R16B02/lib/sasl/src/release_handler.erl#L835
- `check_rel/3,4`: Try rel file under releases directory of slim release.
  https://github.com/erlang/otp/blob/OTP_R16B02/lib/sasl/src/release_handler.erl#L836
