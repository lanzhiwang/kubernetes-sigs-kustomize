[ConfigMaps]: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#configmap-v1-core
[ELF]: https://en.wikipedia.org/wiki/Executable_and_Linkable_Format
[Go plugin]: https://golang.org/pkg/plugin
[Secrets]: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#secret-v1-core
[base64]: https://tools.ietf.org/html/rfc4648#section-4
[configuration directory]: https://wiki.archlinux.org/index.php/XDG_Base_Directory#Specification
[grpc]: https://grpc.io
[tag]: /../../releases
[v2.0.3]: /../../releases/tag/v2.0.3
[`exec.Command`]: https://golang.org/pkg/os/exec/#Command

# Generating Secrets

## What's a Secret?

Kubernetes [ConfigMaps] and [Secrets] are both
key:value maps, but the latter is intended to
signal that its values have a sensitive nature -
e.g. pass phrases or ssh keys.
Kubernetes ConfigMaps 和 Secrets 都是键值映射,
但后者旨在表明其值具有敏感性质, 例如密码短语或 SSH 密钥.

Kubernetes developers work in various ways to hide
the information in a Secret more carefully than
the information held by ConfigMaps, Deployments,
etc.
Kubernetes 开发人员采用各种方法来更仔细地隐藏 Secret 中的信息,
而不是 ConfigMap、Deployment 等中保存的信息.

## Make a place to work

<!-- @establishBase @testAgainstLatestRelease -->
```
DEMO_HOME=$(mktemp -d)

DEMO_HOME=~/work/code/go_code/kustomize/learn/generators
```

## Secret values from local files

kustomize has three different (builtin) ways to
generate a secret from local files:
kustomize 提供了三种不同的(内置)方法, 可以从本地文件生成密钥:

 * get them from so-called _env_ files (`NAME=VALUE`, one per line),
   从所谓的环境变量文件( NAME=VALUE , 每行一个)中获取它们,

 * consume the entire contents of a file to make one secret value,
   将文件的全部内容转换为一个秘密值,

 * get literal values from the kustomization file itself.
   从自定义文件本身获取字面值.

Here's an example combining all three methods:
以下是一个结合这三种方法的示例:

Make an env file with some short secrets:
创建一个包含一些简短密钥的环境变量文件:

<!-- @makeEnvFile @testAgainstLatestRelease -->
```
cat <<'EOF' >$DEMO_HOME/foo.env
ROUTER_PASSWORD=admin
DB_PASSWORD=iloveyou
EOF

$ tree -a .
.
├── README.md
└── foo.env

1 directory, 2 files
$ cat foo.env
ROUTER_PASSWORD=admin
DB_PASSWORD=iloveyou
$
```

Make a text file with a long secret:
创建一个包含长密钥的文本文件:

<!-- @makeLongSecretFile @testAgainstLatestRelease -->
```
cat <<'EOF' >$DEMO_HOME/longsecret.txt
Lorem ipsum dolor sit amet,
consectetur adipiscing elit,
sed do eiusmod tempor incididunt
ut labore et dolore magna aliqua.
EOF

$ tree -a .
.
├── README.md
├── foo.env
└── longsecret.txt

1 directory, 3 files
$ cat longsecret.txt
Lorem ipsum dolor sit amet,
consectetur adipiscing elit,
sed do eiusmod tempor incididunt
ut labore et dolore magna aliqua.
$
```

And make a kustomization file referring to the
above and _additionally_ defining some literal KV
pairs in line:
然后创建一个自定义文件, 引用上述内容, 并在其中额外定义一些字面意义上的键值对:

<!-- @makeKustomization1 @testAgainstLatestRelease -->
```
cat <<'EOF' >$DEMO_HOME/kustomization.yaml
secretGenerator:
- name: mysecrets
  envs:
  - foo.env
  files:
  - longsecret.txt
  literals:
  - FRUIT=apple
  - VEGETABLE=carrot
EOF

$ tree -a .
.
├── README.md
├── foo.env
├── kustomization.yaml
└── longsecret.txt

1 directory, 4 files
$ cat kustomization.yaml
secretGenerator:
- name: mysecrets
  envs:
  - foo.env
  files:
  - longsecret.txt
  literals:
  - FRUIT=apple
  - VEGETABLE=carrot
$
```

Now generate the Secret:

<!-- @build1 @testAgainstLatestRelease -->
```
result=$(kustomize build $DEMO_HOME)
echo "$result"
# Spot check the result:
test 1 == $(echo "$result" | grep -c "FRUIT: YXBwbGU=")

$ result=$(kustomize build $DEMO_HOME)
$ echo "$result"
apiVersion: v1
data:
  DB_PASSWORD: aWxvdmV5b3U=
  FRUIT: YXBwbGU=
  ROUTER_PASSWORD: YWRtaW4=
  VEGETABLE: Y2Fycm90
  longsecret.txt: |
    TG9yZW0gaXBzdW0gZG9sb3Igc2l0IGFtZXQsCmNvbnNlY3RldHVyIGFkaXBpc2NpbmcgZW
    xpdCwKc2VkIGRvIGVpdXNtb2QgdGVtcG9yIGluY2lkaWR1bnQKdXQgbGFib3JlIGV0IGRv
    bG9yZSBtYWduYSBhbGlxdWEuCg==
kind: Secret
metadata:
  name: mysecrets-654gkkf95b
type: Opaque
$ echo "$result" | grep -c "FRUIT: YXBwbGU="
1
$
```

This emits something like

> ```
> apiVersion: v1
> kind: Secret
> metadata:
>   name: mysecrets-hfb5df789h
> type: Opaque
> data:
>   FRUIT: YXBwbGU=
>   VEGETABLE: Y2Fycm90
>   ROUTER_PASSWORD: YWRtaW4=
>   DB_PASSWORD: aWxvdmV5b3U=
>   longsecret.txt: TG9yZW0gaXBzdW0gZG9sb3Igc2l0I... (elided)
> ```

The name of the resource is the prefix `mysecrets`
(as specfied in the kustomization file), followed
by a hash of its contents.

Use your favorite base64 decoder to confirm the raw
versions of any of these values.
使用您最喜欢的 base64 解码器来确认这些值的原始版本.

The problem that these three approaches share is
that the purported secrets must live on disk.
这三种方法的共同问题是, 所谓的秘密必须存储在磁盘上.

This adds additional security questions - who can
see the files, who installs them, who deletes
them, etc.
这增加了额外的安全问题——谁可以查看文件, 谁可以安装文件, 谁可以删除文件等等.

## Secret values from anywhere

A general alternative is to enshrine secret
value generation in a [plugin](../docs/plugins).
另一种常见的做法是将秘密值生成功能封装在插件中.

The values can then come in via, say, an
authenticated and authorized RPC to a password
vault service.
然后, 这些值可以通过例如经过身份验证和授权的 RPC 发送到密码库服务.

[sgp]: ../plugin/someteam.example.com/v1/secretsfromdatabase

Here's a [secret generator plugin][sgp]
that pretends to pull the values of a map
from a database.
这是一个秘密生成器插件 它假装提取映射表中的值. 来自数据库.

Download it

<!-- @copyPlugin @testAgainstLatestRelease -->
```
repo=https://raw.githubusercontent.com/kubernetes-sigs/kustomize
pPath=plugin/someteam.example.com/v1/secretsfromdatabase
dir=$DEMO_HOME/kustomize/$pPath

mkdir -p $dir

curl -s -o $dir/SecretsFromDatabase.go \
  ${repo}/master/$pPath/SecretsFromDatabase.go
```

Compile it

<!-- @compilePlugin @xtest -->
```
go build -buildmode plugin \
  -o $dir/SecretsFromDatabase.so \
  $dir/SecretsFromDatabase.go

$ dir=~/work/code/go_code/kustomize/plugin/someteam.example.com/v1/secretsfromdatabase
$ cd ${dir}
$ go build -buildmode plugin -o $dir/SecretsFromDatabase.so $dir/SecretsFromDatabase.go
$ ls $dir/SecretsFromDatabase.so
~/work/code/go_code/kustomize/plugin/someteam.example.com/v1/secretsfromdatabase/SecretsFromDatabase.so
$
```

Create a configuration file for it:

<!-- @makeConfiguration @testAgainstLatestRelease -->
```
cat <<'EOF' >$DEMO_HOME/secretFromDb.yaml
apiVersion: someteam.example.com/v1
kind: SecretsFromDatabase
metadata:
  name: mySecretGenerator
name: forbiddenValues
namespace: production
keys:
- ROCKET
- VEGETABLE
EOF
```

Create a new kustomization file
referencing this plugin:

<!-- @makeKustomization2 @testAgainstLatestRelease -->
```
cat <<'EOF' >$DEMO_HOME/kustomization.yaml
generators:
- secretFromDb.yaml
EOF
```

Finally, generate the secret, setting
`XDG_CONFIG_HOME` so that the plugin
can be found under `$DEMO_HOME`:

<!-- @build2 @xtest -->
```
result=$( \
  XDG_CONFIG_HOME=$DEMO_HOME \
  kustomize build --enable_alpha_plugins $DEMO_HOME )
echo "$result"
# Spot check the result:
test 1 == $(echo "$result" | grep -c "FRUIT: YXBwbGU=")
```

This should emit something like:

> ```
> apiVersion: v1
> kind: Secret
> metadata:
>   name: mysecrets-bdt27dbkd6
> type: Opaque
> data:
>  FRUIT: YXBwbGU=
>  VEGETABLE: Y2Fycm90
> ```

i.e. a subset of the same values as above.