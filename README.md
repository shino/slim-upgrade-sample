# Sample to upgrade slim release, a little workaround needed

In upgrading, we need unpack relup archive manually
and use `release_handler:set_unpacked/2` directly.

cf: [erlang-bugs] release_handler error in upgrading slim release
http://erlang.org/pipermail/erlang-bugs/2013-November/003900.html
(Thanks to Siri Hansen!)

Environment:
- R16B02 used both for generating slim release and for boot it
- `rebar` from `slim-release-support` branch of shino's fork at commit: `eafc1bd`
  https://github.com/shino/rebar/tree/slim-release-support
  (included in this repository as binary)

## Step 1

Generate version 1 and 2 of releases and upgrade package between them.
The script `generate.sh` does this work with help of rebar.
After the script finishes, the node with version 1 release is located
at `sample/rel/sample_1`. Also the node with the version has started,
whose name is `sample@127.0.0.1`.

## Step 2

If we use `release_handler:unpack_release/1` directly, an error happens (at least
with R16B02):

```
(sample@127.0.0.1)1> release_handler:unpack_release("sample_2").
{error,{enoent,"/tmp/slim-upgrade-sample/sample/rel/sample_1/releases/sample_2.rel"}}
```

A workaround is as follows:
1. Unpack the relup archive, which is generated at Step 1, manually and
   place `releases` and new applications to appropriate directories.
2. Call `release_handler:set_unpacked("releases/sample_2.rel", [{sample, "2", "lib"}]).`
   manually.
3. Call `release_handler:install_release("2").`.

All these steps are executed by the script `upgrade.sh`.
After upgrading, we can confirm that new version is working by calling
`sample_app:new_export()` which is a new function in version 2.

## Some observations on `release_handler:unpack_release/1`

Look into `lib/sasl/src/release_handler.erl`
https://github.com/erlang/otp/blob/OTP_R16B02/lib/sasl/src/release_handler.erl

- `extract_rel_file/2`: Extract rel file in tar ball under root directory.
  https://github.com/erlang/otp/blob/OTP_R16B02/lib/sasl/src/release_handler.erl#L835
- `check_rel/3,4`: Try rel file under releases directory of slim release.
  https://github.com/erlang/otp/blob/OTP_R16B02/lib/sasl/src/release_handler.erl#L836
