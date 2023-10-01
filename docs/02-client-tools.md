# Installing the Client Tools

In this lab you will install the command line utilities required to complete this tutorial: [cfssl](https://github.com/cloudflare/cfssl), [cfssljson](https://github.com/cloudflare/cfssl), and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl).

## Install CFSSL

The `cfssl` and `cfssljson` command line utilities will be used to provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) and generate TLS certificates.

Download and install `cfssl` and `cfssljson` from the [cfssl repository](https://github.com/cloudflare/cfssl/releases):

### OS X

OS X users may experience problems using the pre-built binaries in which case [Homebrew](https://brew.sh) might be a better option:

```
brew install cfssl
```

### Linux

```shell
wget -q --show-progress --https-only --timestamping \
  https://github.com/cloudflare/cfssl/releases/download/v1.6.3/cfssl_1.6.3_linux_amd64 \
  https://github.com/cloudflare/cfssl/releases/download/v1.6.3/cfssljson_1.6.3_linux_amd64
```

```shell
chmod +x cfssl_1.6.3_linux_amd64 cfssljson_1.6.3_linux_amd64
```

```shell
sudo mv cfssl_1.6.3_linux_amd64 /usr/local/bin/cfssl
```

```shell
sudo mv cfssljson_1.6.3_linux_amd64 /usr/local/bin/cfssljson
```

### Windows

For windows on 64 bit use powershell, using administrative rights

```shell
PS C:\Windows\system32>Invoke-WebRequest -Uri https://github.com/cloudflare/cfssl/releases/download/v1.6.3/cfssl_1.6.3_windows_amd64.exe -OutFile cfssl.exe
PS C:\Windows\system32>Invoke-WebRequest -Uri https://github.com/cloudflare/cfssl/releases/download/v1.4.1/cfssljson_1.6.3_windows_amd64.exe -OutFile cfssljson.exe
```

### Verification

Verify `cfssl` version 1.6.3 or higher is installed:

```shell
cfssl version
```

If this step fails with a runtime error, try installing cfssl following instructions on [CloudFlare's repository](https://github.com/cloudflare/cfssl#installation)

> output

```shell
Version: 1.6.3
Runtime: go1.19.2
```

```shell
cfssljson -version
```

> output

```shell
Version: 1.6.3
Runtime: go1.19.2
```

## Install kubectl

The `kubectl` command line utility is used to interact with the Kubernetes API Server. Download and install `kubectl` from the official release binaries:

### OS X

```
brew install kubectl
```

### Linux

```shell
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

```shell
chmod +x ./kubectl
```

```shell
sudo mv ./kubectl /usr/local/bin/kubectl
```

### Windows

Note you need to have chocolately package manager installed first (<https://chocolatey.org/>)

```shell
PS C:\Windows\system32>choco install kubernetes-cli
```

### Verification

Verify `kubectl` version 1.26.3 or higher is installed:

```shell
kubectl version --client
```

> output

```shell
clientVersion:
  buildDate: "2023-03-15T13:33:11Z"
  compiler: gc
  gitCommit: 9e644106593f3f4aa98f8a84b23db5fa378900bd
  gitTreeState: clean
  gitVersion: v1.26.3
  goVersion: go1.19.7
  major: "1"
  minor: "26"
  platform: darwin/arm64
kustomizeVersion: v4.5.7
```

To quick check kubectl version, you can also use the following command :

```shell
 kubectl version -oyaml --client | awk '/gitVersion/{print $2;}'
```

> output

```shell
v1.26.3
```

Next: [Provisioning Compute Resources](03-compute-resources.md)
