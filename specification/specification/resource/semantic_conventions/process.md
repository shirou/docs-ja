<!--
# Process and process runtime resources
-->
# プロセスとプロセスのランタイムリソース

**Status**: [Experimental](../../document-status.md)

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

<!-- - [Process](#process)
- [Process runtimes](#process-runtimes)
  * [Erlang Runtimes](#erlang-runtimes)
  * [Go Runtimes](#go-runtimes)
  * [Java runtimes](#java-runtimes)
  * [JavaScript runtimes](#javascript-runtimes)
  * [.NET Runtimes](#net-runtimes)
  * [Python Runtimes](#python-runtimes)
  * [Ruby Runtimes](#ruby-runtimes)
-->

- [プロセス](#プロセス)
- [プロセスランタイム](#プロセスランタイム)
  * [Erlang Runtimes](#erlang-runtimes)
  * [Go Runtimes](#go-runtimes)
  * [Java runtimes](#java-runtimes)
  * [JavaScript runtimes](#javascript-runtimes)
  * [.NET Runtimes](#net-runtimes)
  * [Python Runtimes](#python-runtimes)
  * [Ruby Runtimes](#ruby-runtimes)

<!-- tocstop -->

<!--
## Process
-->
## プロセス

**type:** `process`

<!--
**Description:** An operating system process.
-->

**Description:** OSのプロセス。

<!-- semconv process -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `process.pid` | int | プロセスの識別子(PID) | `1234` | No |
| `process.executable.name` | string | 実行可能なプロセスの名前。Linuxベースのシステムでは、`proc/[pid]/status` の `Name` を設定できます。Windowsでは、`GetProcessImageFileNameW`のBaseNameを設定できます。 | `otelcol` | See below |
| `process.executable.path` | string | 実行可能なプロセスへのフルパス。Linuxベースのシステムでは、`proc/[pid]/exe` を設定できます。Windowsでは、`GetProcessImageFileNameW`の結果を設定できます。 | `/usr/bin/cmd/otelcol` | See below |
| `process.command` | string | プロセスを起動するために使用されたコマンド (すなわち、コマンド名)。Linuxベースのシステムでは、`proc/[pid]/cmdline`の0番目の文字列を設定できます。Windowsでは、`GetCommandLineW`から抽出した最初のパラメータを設定できます。 | `cmd/otelcol` | See below |
| `process.command_line` | string | プロセスを起動するために使用された完全なコマンドを1つの文字列で表します。Windowsでは、`GetCommandLineW`の結果を設定できます。モニタリングのためだけに構築する必要がある場合は設定しないでください。代わりに `process.command_args` を使ってください。 | `C:\cmd\otecol --config="my directory\config.yaml"` | See below |
| `process.command_args` | string[] | プロセスが受け取ったすべてのコマンド引数(コマンド/実行可能なもの自体を含む)。Linuxベースのシステム(およびprocfsをサポートする他のいくつかのUnix系システム)では、`proc/[pid]/cmdline`から抽出されたNULLで区切られた文字列のリストを設定できます。libcベースの実行ファイルの場合、これは `main` に渡される完全な argv ベクトルになります。 | `[cmd/otecol, --config=config.yaml]` | See below |
| `process.owner` | string | プロセスを所有するユーザーのユーザー名。 | `root` | No |
<!-- endsemconv -->

<!--
Between `process.command_args` and `process.command_line`, usually `process.command_args` should be preferred.
On Windows and other systems where the native format of process commands is a single string,
`process.command_line` can additionally (or instead) be used.
-->

`process.command_args` と `process.command_line` では、通常は `process.command_args` が優先されます。Windowsや、プロセスコマンドのネイティブフォーマットが単一の文字列である他のシステムでは、`process.command_line` を追加で (または代わりに) 使用することができます。

<!--
For backwards compatibility with older versions of this semantic convention,
it is possible but deprecated to use an array as type for `process.command_line`.
In that case it MUST be interpreted as if it was `process.command_args`.
-->

このセマンティック規約の古いバージョンとの後方互換性のために、`process.command_line` の型として配列を使用することは可能ですが、非推奨です。その場合、それは `process.command_args` であるかのように解釈されなければなりません (MUST)。

<!--
At least one of `process.executable.name`, `process.executable.path`, `process.command`, `process.command_line` or `process.command_args` is required to allow back ends to identify the executable.
-->

バックエンドが実行ファイルを識別するためには、`process.executable.name`、`process.executable.path`、`process.command`、`process.command_line`、`process.command_args` のうち少なくともひとつが必要です。

<!--
## Process runtimes
-->
## プロセスランタイム

**type:** `process.runtime`

<!--
**Description:** The single (language) runtime instance which is monitored.
-->

**Description:** モニター対象となる単一の(言語)ランタイムインスタンスです。

<!-- semconv process.runtime -->
| Attribute  | Type | Description  | Examples  | Required |
|---|---|---|---|---|
| `process.runtime.name` | string | このプロセスのランタイムの名前。コンパイルされたネイティブバイナリの場合、これはコンパイラの名前であるべきです[SHOULD]。 | `OpenJDK Runtime Environment` | No |
| `process.runtime.version` | string | ランタイムが修正せずに返した、このプロセスのランタイムのバージョン。 | `14.0.2` | No |
| `process.runtime.description` | string | プロセスのランタイムに関する追加の記述、例えば、ランタイム環境の特定のベンダのカスタマイズなど。 | `Eclipse OpenJ9 Eclipse OpenJ9 VM openj9-0.21.0` | No |
<!-- endsemconv -->

<!--
How to set these attributes for particular runtime kinds is described in the following subsections.
-->

これらの属性を特定のランタイムタイプに設定する方法については、以下の項目で説明します。

<!--
In addition to these attributes, [`telemetry.sdk.language`](README.md#telemetry-sdk) can be used to determine the general kind of runtime used.
-->

これらの属性に加えて、[`telemetry.sdk.language`](README.md#telemetry-sdk)を使用して、使用するランタイムの一般的な種類を決定することができます。

<!--
### Erlang Runtimes
-->

### Erlangランタイム

<!--
- `process.runtime.name` - The name of the Erlang VM being used, i.e., `erlang:system_info(machine)`.
- `process.runtime.version` -  The version of the runtime (ERTS - Erlang Runtime System), i.e., `erlang:system_info(version)`.
- `process.runtime.description` - string | An additional description about the runtime made by combining the OTP version, i.e., `erlang:system_info(otp_release)`, and ERTS version.
-->

- `process.runtime.name` - 使用されているErlang VMの名前、つまり `erlang:system_info(machine)` です。
- `process.runtime.version` - ランタイムのバージョン (ERTS - Erlang Runtime System)、つまり `erlang:system_info(version)` です。
- `process.runtime.description` - string | OTPバージョン、つまり `erlang:system_info(otp_release)` と ERTSバージョンを組み合わせて作られるランタイムに関する追加の説明です。


例:

| `process.runtime.name` | `process.runtime.version` | `process.runtime.description` |
| --- | --- | --- |
| BEAM | 11.1 |  Erlang/OTP 23 erts-11.1 |

<!--
### Go Runtimes
-->

### Goランタイム

<!--
TODO(<https://github.com/open-telemetry/opentelemetry-go/issues/1181>): Confirm the contents here
-->

TODO(<https://github.com/open-telemetry/opentelemetry-go/issues/1181>)。ここの内容を確認


| Value | Description |
| --- | --- |
| `gc` | Go compiler |
| `gccgo` | GCC Go frontend |

### Javaランタイム

<!--
Java instrumentation should fill in the values by copying from system properties.
-->

Javaの計装は、システムのプロパティからコピーして値を埋める必要があります。

<!--
- `process.runtime.name` - Fill in the value of `java.runtime.name` as is
- `process.runtime.version` - Fill in the value of `java.runtime.version` as is
- `process.runtime.description` - Fill in the values of `java.vm.vendor`, `java.vm.name`, `java.vm.version`
  in that order, separated by spaces.
-->

- `process.runtime.name` - `java.runtime.name` の値をそのまま入力してください。
- `process.runtime.version` - `java.runtime.version` の値をそのまま入力してください。
- `process.runtime.description` - `java.vm.vender`, `java.vm.name`, `java.vm.version` の順に、スペースで区切って入力してください。

<!--
Examples for some Java runtimes
-->

いくつかのJavaランタイムの例

| Name | `process.runtime.name` | `process.runtime.version` | `process.runtime.description` |
| --- | --- | --- | --- |
| OpenJDK | OpenJDK Runtime Environment | 11.0.8+10 | Oracle Corporation OpenJDK 64-Bit Server VM 11.0.8+10 |
| AdoptOpenJDK Eclipse J9 | OpenJDK Runtime Environment | 11.0.8+10 | Eclipse OpenJ9 Eclipse OpenJ9 VM openj9-0.21.0 |
| AdoptOpenJDK Hotspot | OpenJDK Runtime Environment | 11.0.8+10 | AdoptOpenJDK OpenJDK 64-Bit Server VM 11.0.8+10 |
| SapMachine | OpenJDK Runtime Environment | 11.0.8+10-LTS-sapmachine | SAP SE OpenJDK 64-Bit Server VM 11.0.8+10-LTS-sapmachine |
| Zulu OpenJDK | OpenJDK Runtime Environment | 11.0.8+10-LTS | Azul Systems, Inc OpenJDK 64-Bit Server VM Zulu11.41+23-CA |
| Oracle Hotspot 8 (32 bit) | Java(TM) SE Runtime Environment | 1.8.0_221-b11 | Oracle Corporation Java HotSpot(TM) Client VM 25.221-b11 |
| IBM J9 8 | Java(TM) SE Runtime Environment | 8.0.5.25 - pwa6480sr5fp25-20181030_01(SR5 FP25) | IBM Corporation IBM J9 VM 2.9 |
| Android 11 | Android Runtime | 0.9 | The Android Project Dalvik 2.1.0 |

### JavaScriptランタイム

<!--
TODO(<https://github.com/open-telemetry/opentelemetry-js/issues/1544>): Confirm the contents here
-->

TODO(<https://github.com/open-telemetry/opentelemetry-js/issues/1544>): この内容を確認


| Value | Description |
| --- | --- |
| `nodejs` | NodeJS |
| `browser` | Web Browser |
| `iojs` | io.js |
| `graalvm` | GraalVM |

<!--
When the value is `browser`, `process.runtime.version` SHOULD be set to the User-Agent header.
-->

値が `browser` の場合、`process.runtime.version` は User-Agent ヘッダに設定されるべきです(SHOULD)。

### .NETランタイム

<!--
TODO(<https://github.com/open-telemetry/opentelemetry-dotnet/issues/1281>): Confirm the contents here
-->

TODO(<https://github.com/open-telemetry/opentelemetry-dotnet/issues/1281>): この内容を確認


| Value | Description |
| --- | --- |
| `dotnet-core` | .NET Core, .NET 5+ |
| `dotnet-framework` | .NET Framework |
| `mono` | Mono |

### Pythonランタイム

<!--
Python instrumentation should fill in the values as follows:
-->

Pythonの計装では、以下のように値を埋める必要があります。

<!--
- `process.runtime.name` -
  Fill in the value of [`sys.implementation.name`][py_impl]
- `process.runtime.version` -
  Fill in the [`sys.implementation.version`][py_impl] values separated by dots.
  Leave out the release level and serial if the release level
  equals `final` and the serial equals zero
  (leave out either both or none).

  This can be implemented with the following Python snippet:

  ```python
  vinfo = sys.implementation.version
  result =  ".".join(map(
      str,
      vinfo[:3]
      if vinfo.releaselevel == "final" and not vinfo.serial
      else vinfo
  ))
  ```

- `process.runtime.description` - Fill in the value of [`sys.version`](https://docs.python.org/3/library/sys.html#sys.version) as-is.
-->

- `process.runtime.name` - [`sys.implementation.name`][py_impl]の値を記入します。
- `process.runtime.version` - ドットで区切られた[`sys.implementation.version`][py_impl]の値を入力します。リリースレベルが `final` で、シリアルが 0 の場合は、リリースレベルとシリアルを省きます (両方を省くか、何も省かない)。

  これは、以下のPythonスニペットで実装できます。

  ```python
  vinfo = sys.implementation.version
  result =  ".".join(map(
      str,
      vinfo[:3]
      if vinfo.releaselevel == "final" and not vinfo.serial
      else vinfo
  ))
  ```

- `process.runtime.description` - [`sys.version`](https://docs.python.org/3/library/sys.html#sys.version)の値をそのまま記入します。

[py_impl]: https://docs.python.org/3/library/sys.html#sys.implementation

<!--
Examples for some Python runtimes:
-->

いくつかのPythonランタイムの例:

| Name | `process.runtime.name` | `process.runtime.version` | `process.runtime.description` |
| --- | --- | --- | --- |
| CPython 3.7.3 on Windows | cpython | 3.7.3 | 3.7.3 (v3.7.3:ef4ec6ed12, Mar 25 2019, 22:22:05) [MSC v.1916 64 bit (AMD64)] |
| CPython 3.8.6 on Linux | cpython | 3.8.6 | 3.8.6 (default, Sep 30 2020, 04:00:38) <br>[GCC 10.2.0] |
| PyPy 3 7.3.2 on Linux | pypy | 3.7.4 | 3.7.4 (?, Sep 27 2020, 15:12:26)<br>[PyPy 7.3.2-alpha0 with GCC 10.2.0] |

<!--
Note that on Linux, there is an actual newline in the `sys.version` string,
and the CPython string had a trailing space in the first line.
-->

Linuxでは、`sys.version`の文字列に実際の改行があり、CPythonの文字列では1行目に末尾のスペースがあったことに注意してください。

<!--
Pypy provided a CPython-compatible version in `sys.implementation.version` instead of the actual implementation version which is available in `sys.version`.
-->

Pypyでは、`sys.version`にある実際の実装バージョンではなく、`sys.implementation.version`にCPythonとの互換性のあるバージョンを提供していました。

### Rubyランタイム

<!--
Ruby instrumentation should fill in the values by copying from built-in runtime constants.
-->

Rubyの計装では、組み込みのランタイム定数からコピーして値を埋める必要があります。

<!--
- `process.runtime.name` - Fill in the value of `RUBY_ENGINE` as is
- `process.runtime.version` - Fill in the value of `RUBY_VERSION` as is
- `process.runtime.description` - Fill in the value of `RUBY_DESCRIPTION` as is
-->

- `process.runtime.name` - `RUBY_ENGINE` の値をそのまま入力してください。
- `process.runtime.version` - `RUBY_VERSION` の値をそのまま入力してください。
- `process.runtime.description` - `RUBY_DESCRIPTION` の値を入力してください。

<!--
Examples for some Ruby runtimes
-->

いくつかのRubyランタイムの例

| Name | `process.runtime.name` | `process.runtime.version` | `process.runtime.description` |
| --- | --- | --- | --- |
| MRI | ruby | 2.7.1 | ruby 2.7.1p83 (2020-03-31 revision a0c7c23c9c) [x86_64-darwin19] |
| TruffleRuby | truffleruby | 2.6.2 | truffleruby (Shopify) 20.0.0-dev-92ed3059, like ruby 2.6.2, GraalVM CE Native [x86_64-darwin] |
