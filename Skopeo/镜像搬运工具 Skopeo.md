# 镜像搬运工具 Skopeo

> 一个好的镜像传输工具能节省大量的人力和 CPU 算力，本文将为大家介绍一个能够完全替代 docker-cli 的工具：Skopeo。
>
> https://zhuanlan.zhihu.com/p/472345879

**K8sMeetup搬砖工具 Skopeo**作为公司内部 PaaS 产品的打包发布人员，容器镜像对我们打工人而言就像是工地上的砖头 ，而我的一部分工作就是将这些砖头在各个仓库之间搬来搬去，最终将这些砖头打包放在产品的安装包中，形成一个完整的 PaaS 产品安装包。而选择一个好的搬砖工具能节省大量的人力和 CPU 算力，在日常开发工作中我们也常常会使用 `docker push` 和 `docker pull` 来推拉镜像，虽然本地 `push & pull` 一个镜像并不是什么难事儿，但对于一些特定的场景（如产品打包发布流水线中），还继续使用 Docker 这个笨重的工具来推拉镜像的话，是十分费时费力的，具体的原理可以参考我之前写的博客《深入浅出容器镜像的一生》。自从 Kubernetes v1.20 之后，K8s 社区也弃用了 Docker 作为容器运行时，docker-shim 相关的代码将在 Kubelet 中不再维护，随后掀起了一波去 `Docker` 的浪潮。那么现在有没有一种能够替代 `docker-cli` 的工具来传输镜像呢？  
**今天给大家介绍一个能够完全替代 docker-cli 来搬运镜像的工具：Skopeo**。  
这玩意儿比 `docker-cli` 高到不知道哪里去了！



## 安装

**K8sMeetup安装方式**官方的安装方式参考安装文档即可[https://github.com/containers/skopeo/blob/main/install.md](https://link.zhihu.com/?target=https%3A//github.com/containers/skopeo/blob/main/install.md) 。  
由于我的 VPS 机器是 Ubuntu 18.04 的 OS ，配置 apt 源并没成功，当场翻车。在官方的 Makefile 里只提供了在 nixos 下构建静态连接的方式，其他 Linux 发行版只能使用动态链接的方式来编译。但动态链接的方式通用性太差，比如在 Ubuntu 18.04 上使用动态链接编译的 Skopeo 只能在 Ubuntu 上使用，无法在 CentOS 上使用。因为动态链接编译的二进制文件在不同的 OS 上所依赖的库文件是不一样的，所以我们需要手动编译一个静态链接的 Skopeo 可执行文件。

### 构建静态连接

- Clone repo

```text
$ SKOPEO_VERSION=v1.3.0
$ git clone --branch ${SKOPEO_VERSION} https://github.com/containers/skopeo
$ cd skopeo
```

- docker build

```text
$ BUILD_IMAGE=nixos/nix:2.3.12
$ docker run --rm -t -v $PWD:/build ${BUILD_IMAGE} \
sh -c "cd /build && nix build -f nix && cp ./result/bin/skopeo skopeo"
```

- 使用 `nixos/nix:2.3.12` 来构建静态链接的 Skopeo 二进制文件需要完整构建 Skopeo 所有的依赖，比如 glibc、systemd、golang 等，所以构建十分耗时。在一台 4c8G 的机器上构建用了将近半个小时，在 GitHub Action 的 runner 机器上构建需要将近一个小时。



![img](https://pic4.zhimg.com/80/v2-aa7cf9081cac8081dd9acd0c1592628b_720w.webp)



- 使用 GitHub Action 构建：

```text
---
name: build static binary
on: push
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      BUILD_IMAGE: "nixos/nix:2.3.12"
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Build static binary
        run: |
          docker run --rm -t -v $PWD:/build --name builder ${BUILD_IMAGE} \
          sh -c "cd /build && nix build -f nix && cp ./result/bin/skopeo skopeo-linux-amd64"

      - name: Release
        uses: softprops/action-gh-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          files: skopeo-linux-amd64
```

不过也可以使用 go build 的方式构建出静态链接的二进制文件，如下`Dockerfile`：

```text
FROM golang:1.14-buster as skopeo-builder
ARG SKOPEO_VERSION=v1.2.0
RUN apt-get update \
 && apt-get install -y -qq libdevmapper-dev libgpgme11-dev 

ENV GOPATH=/
WORKDIR /src/github.com/containers/skopeo
RUN git clone --branch ${SKOPEO_VERSION} https://github.com/containers/skopeo . \
 && CGO_ENABLE=0 GO111MODULE=on go build -mod=vendor "-buildmode=pie" -ldflags '-extldflags "-static"' -gcflags "" \
 -tags "exclude_graphdriver_devicemapper exclude_graphdriver_btrfs containers_image_openpgp" -o /usr/bin/skopeo ./cmd/skopeo

FROM alpine:3.12
COPY --from=skopeo-builder /usr/bin/skopeo /usr/bin/skopeo
# FROM scratch
# COPY --from=skopeo-builder /usr/bin/skopeo /skopeo
# DOCKER_BUILDKIT=1 docker build -o type=local,dest=$PWD -f Dockerfile .
```

- 通过 GitHub Action 来编译



## **K8sMeetup上手体验**

### 指令

- copy：复制一个镜像从 A 到 B，这里的 A 和 B 可以为本地 Docker 镜像或者 Registry 上的镜像；
- inspect：查看一个镜像的 manifest 或者 image config 详细信息；
- delete：删除一个镜像 tag，可以是本地 Docker 镜像或者 Registry 上的镜像；
- list-tags：列出一个 Registry 上某个镜像的所有 tag；
- login：登录到某个 Registry，和 docker login 类似；
- logout：退出已经登录到某个 Registry 的 auth 信息，和 docker logout 类似；
- manifest-digest：几圈一个文件的 sha256sum 值；
- standalone-sign、standalone-verify 这两个是和镜像加密相关的，使用的不是很多；
- sync：同步一个镜像从 A 到 B，感觉和 copy 一样，但 sync 支持的参数更多，功能更强大。

```text
completion             generate the autocompletion script for the specified shell
  copy                   Copy an IMAGE-NAME from one location to another
  delete                 Delete image IMAGE-NAME
  help                   Help about any command
  inspect                Inspect image IMAGE-NAME
  list-tags              List tags in the transport/repository specified by the REPOSITORY-NAME
  login                  Login to a container registry
  logout                 Logout of a container registry
  manifest-digest        Compute a manifest digest of a file
  standalone-sign        Create a signature using local files
  standalone-verify      Verify a signature using local files
  sync                   Synchronize one or more images
```

### **参数**

- command-timeout：命令超时时间；
- debug：开启 debug 模式，输出详细的日志；
- insecure-policy：使用非安全的 policy，如果没有配置 policy 的话，需要加上该参数；
- override-arch：处理镜像时覆盖客户端 CPU 体系架构，如在 AMD64 的机器上用 Skopeo 处理 ARM64 的镜像；
- override-os：处理镜像时覆盖客户端 OS。

```text
Flags:
      --command-timeout duration   timeout for the command execution
      --debug                      enable debug output
  -h, --help                       help for skopeo
      --insecure-policy            run the tool without any policy check
      --override-arch ARCH         use ARCH instead of the architecture of the machine for choosing images
      --override-os OS             use OS instead of the running OS for choosing images
      --override-variant VARIANT   use VARIANT instead of the running architecture variant for choosing images
      --policy string              Path to a trust policy file
      --registries.d DIR           use registry configuration files in DIR (e.g. for container signature storage)
      --tmpdir string              directory used to store temporary files
  -v, --version                    Version for Skopeo
```

以下是我在使用 Skopeo 命令时候的一些参数：

```text
--insecure-policy --src-tls-verify=false --dest-tls-verify=false
```

## **IMAGE NAMES 镜像格式**

在使用 Skopeo 之前，我们首先要知道在命令行中镜像的格式，下面是官方详细的文档格式。**无论是 src 镜像还是 dest 镜像都要满足以下格式才可以**。

| IMAGE NAMES（镜像格式） | example                                                      |
| ----------------------- | ------------------------------------------------------------ |
| containers-storage:     | containers-storage:                                          |
| dir:                    | dir:/PATH                                                    |
| docker://               | docker://[http://k8s.gcr.io/kube-apiserver:v1.17.5](https://link.zhihu.com/?target=http%3A//k8s.gcr.io/kube-apiserver%3Av1.17.5) |
| docker-daemon:          | docker-daemon:alpine:latest                                  |
| docker-archive:         | docker-archive:alpine.tar (docker save)                      |
| oci:                    | oci:alpine:latest                                            |

需要注意的是，这几种镜像的名字，对应着镜像存在的方式，不同存在的方式对镜像的 layer 处理的方式也不一样：

- `docker://`这种方式是存在 `Registry` 上的；
- `docker-daemon`: 是存在本地 docker pull 下来的；
- `docker-archive`是通过 docker save 出来的镜像。

同一个镜像有这几种存在的方式就像水分子有气体、液体、固体一样。可以这样去理解，他们表述的都是同一个镜像，只不过是存在的方式不一样而已。

## **skopeo login** 

在使用 `Skopeo` 前如果 src 或 dest 镜像是在 Registry 中的，如果非 public 的镜像需要相应的 auth 认证，可以使用 docker login 或者 skopeo login 的方式登录到 Registry，生成如下格式的 Registry 登录配置文件。

```text
$ jq "." ~/.docker/config.json                                                             
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "d2sdwdaqWMasss7bSVlJFpmQE43Sw=="
    }
  },
  "HttpHeaders": {
    "User-Agent": "Docker-Client/19.03.5 (linux)"
  },
  "experimental": "enabled"
}
```

## **skopeo copy**

> Copy an IMAGE-NAME from one location to another
> 将一个镜像从 A 复制到 B

注意一下，这里的 location 就是指的上面提到的 IMAGE NAMES，也就是说`skopeo copy src dest`可以有 6*6=36 种组合！比如我可以将一个镜像从一个 Registry 复制到另一个`Registry：skopeo copy docker://IMAGE_NAME docker://IMAGE_NAME`；或者将一个镜像从 Registry 中复制到一个本地目录`skopeo copy docker://k8s.gcr.io/pause:3.3 dir:pause:3.3`。

- 从 Regsitry A 到 Registry B 复制镜像：

```text
$ skopeo copy docker://k8s.gcr.io/kube-apiserver:v1.17.5 docker://hub.k8s.li/kube-apiserver:v1.17.5 --dest-authfile /root/.docker/config.json
Getting image source signatures
Copying blob 597de8ba0c30 done
Copying blob e13a88fa950c done
Copying config f640481f6d done
Writing manifest to image destination
Storing signatures
```

> Skopeo 输出的日志显示是 Copying blob 597de8ba0c30 done.可以看到 Skopeo 是直接从 Registry 中 copy 镜像 layer 的 blob 文件，传输是镜像在 Registry 中存储的原始格式。

- 将镜像从 Registry 复制到本地目录：

```text
$ skopeo copy docker://k8s.gcr.io/pause:3.3 dir:pause:3.3
Getting image source signatures
Copying blob aeab776c4837 done
Copying config 0184c1613d done
Writing manifest to image destination
Storing signatures

$ tree pause:3.3
pause:3.3
├── 0184c1613d92931126feb4c548e5da11015513b9e4c104e7305ee8b53b50a9da
├── aeab776c48375e1a61810a0a5f59e982e34425ff505a01c2b57dcedc6799c17b
├── manifest.json
└── version

# 查看镜像的 manifest 文件
$ jq '.' pause:3.3/manifest.json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
  "config": {
    "mediaType": "application/vnd.docker.container.image.v1+json",
    "size": 743,
    "digest": "sha256:0184c1613d92931126feb4c548e5da11015513b9e4c104e7305ee8b53b50a9da"
  },
  "layers": [
    {
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
      "size": 296517,
      "digest": "sha256:aeab776c48375e1a61810a0a5f59e982e34425ff505a01c2b57dcedc6799c17b"
    }
  ]
}

# 根据 manifest 文件查看镜像的 image config 文件
$ jq '.' pause:3.3/0184c1613d92931126feb4c548e5da11015513b9e4c104e7305ee8b53b50a9da
{
  "architecture": "amd64",
  "config": {
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "Entrypoint": [
      "/pause"
    ],
    "WorkingDir": "/",
    "OnBuild": null
  },
  "created": "2020-05-02T09:46:29.068489061Z",
  "history": [
    {
      "created": "2020-05-02T09:46:29.068489061Z",
      "created_by": "ARG ARCH",
      "comment": "buildkit.dockerfile.v0",
      "empty_layer": true
    },
    {
      "created": "2020-05-02T09:46:29.068489061Z",
      "created_by": "ADD bin/pause-amd64 /pause # buildkit",
      "comment": "buildkit.dockerfile.v0"
    },
    {
      "created": "2020-05-02T09:46:29.068489061Z",
      "created_by": "ENTRYPOINT [\"/pause\"]",
      "comment": "buildkit.dockerfile.v0",
      "empty_layer": true
    }
  ],
  "os": "linux",
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:48a5e87615149095fad57d5db80f2cd9728b5562900eccb32842a45e8e8a61ae"
    ]
  }
}
```

- 将镜像从 Registry 复制到本地目录，以 OCI 格式保存：

```text
$ skopeo copy docker://k8s.gcr.io/pause:3.3 oci:images
Getting image source signatures
Copying blob aeab776c4837 done
Copying config fa5df7713f done
Writing manifest to image destination
Storing signatures
$ tree images
images
├── blobs
│   └── sha256
│       ├── 3450ba84b8fbfd12cbf58710c0b5678f4311a888d4d5c42b053faefa1af4f8be
│       ├── aeab776c48375e1a61810a0a5f59e982e34425ff505a01c2b57dcedc6799c17b
│       └── fa5df7713fc78f96e377d236b353d33815073105bbacd381e50705e576ce4da5
├── index.json
└── oci-layout
```

- 替代 docker push 功能，将镜像从 Docker 本地存储 push 到 Registry 中：

```text
 skopeo copy docker-daemon:alpine:3.12 docker://hub.k8s.li/library/alpine:3.12
Getting image source signatures
Copying blob 32f366d666a5 done
Copying config 13621d1b12 done
Writing manifest to image destination
Storing signatures
```

## **skopeo sync**

skopeo sync 的功能基本上等同于 image-syncer 工具，不过个人觉着 `Skopeo` 要比 `image-syncer` 更强大，灵活性更强一些，如果你还在使用 `image-syncer` 的话，建议使用 `skopeo sync` 替代它 。

- `Skopeo sync` 镜像同步文件：

```text
registry.example.com:
    images:
        busybox: []
        redis:
            - "1.0"
            - "2.0"
            - "sha256:111111"
    images-by-tag-regex:
        nginx: ^1\.13\.[12]-alpine-perl$
    credentials:
        username: john
        password: this is a secret
    tls-verify: true
    cert-dir: /home/john/certs
quay.io:
    tls-verify: false
    images:
        coreos/etcd:
            - latest
```

- Image-syncer 镜像同步配置文件：

```text
# registry 登录配置
{
    "quay.io": {        // This "registry" or "registry/namespace" string should be the same as registry or registry/namespace used below in "images" field.  
                            // The format of "registry/namespace" will be more prior matched than "registry"
        "username": "xxx",             
        "password": "xxxxxxxxx",
        "insecure": true         // "insecure" field needs to be true if this registry is a http service, default value is false, version of image-syncer need to be later than v1.0.1 to support this field
    },
    "registry.cn-beijing.aliyuncs.com": {
        "username": "xxx",
        "password": "xxxxxxxxx"
    },
    "registry.hub.docker.com": {
        "username": "xxx",
        "password": "xxxxxxxxxx"
    },
    "quay.io/coreos": {     // "registry/namespace" format is supported after v1.0.3 of image-syncer     
        "username": "abc",              
        "password": "xxxxxxxxx",
        "insecure": true  
    }
}

# 镜像配置
{
    "quay.io/coreos/kube-rbac-proxy": "quay.io/ruohe/kube-rbac-proxy",
    "xxxx":"xxxxx",
    "xxx/xxx/xx:tag1,tag2,tag3":"xxx/xxx/xx"
}
```

- 将镜从 Registry A 同步到 Registry B：

```text
$ skopeo sync --src docker --dest docker k8s.gcr.io/pause:3.3 hub.k8s.li
INFO[0000] Tag presence check                            imagename="k8s.gcr.io/pause:3.3" tagged=true
INFO[0000] Copying image tag 1/1                         from="docker://k8s.gcr.io/pause:3.3" to="docker://hub.k8s.li/pause:3.3"
Getting image source signatures
Copying blob aeab776c4837 done
Copying config 0184c1613d done
Writing manifest to image destination
Storing signatures
INFO[0000] Synced 1 images from 1 sources
```

- 将一个镜像从 Registry 中同步到本地目录：

```text
$ skopeo sync --src docker --dest dir k8s.gcr.io/pause:3.3 images
images
└── pause:3.3
    ├── 0184c1613d92931126feb4c548e5da11015513b9e4c104e7305ee8b53b50a9da
    ├── aeab776c48375e1a61810a0a5f59e982e34425ff505a01c2b57dcedc6799c17b
    ├── manifest.json
    └── version
```

- 将镜像从本地目录同步到 Registry 中：

```text
$ skopeo sync --src dir --dest docker images hub.k8s.li
INFO[0000] Copying image ref 1/1                         from="dir:images/pause:3.3" to="docker://hub.k8s.li/pause:3.3"
Getting image source signatures
Copying blob aeab776c4837 [--------------------------------------] 0.0b / 0.0b
Copying config 0184c1613d done
Writing manifest to image destination
Storing signatures
INFO[0002] Synced 1 images from 1 sources
```

## **skopeo inspect**

这个命令可以查看一个镜像的 image config 或者 manifests 文件，和 docker inspect 命令差不多。不加 --raw 参数默认是查看镜像的 image config 文件，加上 --raw 参数就是查看镜像的 manifest 文件。

- 查看 Docker 本地存储中的一个镜像的 image config 文件：

```text
$ skopeo inspect docker-daemon:alpine:latest
{
    "Name": "docker.io/library/alpine",
    "Digest": "sha256:ab84514e85b179ff569fd0042969b04f68812f23e187a927cb84664b417e0d3e",
    "RepoTags": [],
    "Created": "2021-04-14T19:19:49.594730611Z",
    "DockerVersion": "19.03.12",
    "Labels": null,
    "Architecture": "amd64",
    "Os": "linux",
    "Layers": [
        "sha256:32f366d666a541852cad754ee1cdb53a736110b550f0c2d5a46bc5ba519896b6"
    ],
    "Env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ]
}
```

- 查看 Registry 中一个镜像的 manifests 文件，可以通过这种方式来判断镜像是否存在：

```text
skopeo inspect docker://alpine:latest --raw | jq '.'
{
  "manifests": [
    {
      "digest": "sha256:1775bebec23e1f3ce486989bfc9ff3c4e951690df84aa9f926497d82f2ffca9d",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      },
      "size": 528
    },
    {
      "digest": "sha256:1f66b8f3041ef8575260056dedd437ed94e7bfeea142ee39ff0d795f94ff2287",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "arm",
        "os": "linux",
        "variant": "v6"
      },
      "size": 528
    },
    {
      "digest": "sha256:8d99168167baa6a6a0d7851b9684625df9c1455116a9601835c2127df2aaa2f5",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "arm",
        "os": "linux",
        "variant": "v7"
      },
      "size": 528
    },
    {
      "digest": "sha256:53b74ddfc6225e3c8cc84d7985d0f34666e4e8b0b6892a9b2ad1f7516bc21b54",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "arm64",
        "os": "linux",
        "variant": "v8"
      },
      "size": 528
    },
    {
      "digest": "sha256:52a197664c8ed0b4be6d3b8372f1d21f3204822ba432583644c9ce07f7d6448f",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "386",
        "os": "linux"
      },
      "size": 528
    },
    {
      "digest": "sha256:b421672fe4e74a3c7eff2775736e854d69e8d38b2c337063f8699de9c408ddd3",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "ppc64le",
        "os": "linux"
      },
      "size": 528
    },
    {
      "digest": "sha256:8a22269106a31264874cc3a719c1e280e76d42dff1fa57bd9c7fe68dab574023",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "s390x",
        "os": "linux"
      },
      "size": 528
    }
  ],
  "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
  "schemaVersion": 2
}
```

## **skopeo delete**

使用这个命令可以删除镜像 tag。需要注意的是，通过 Registry API 来删除镜像的 tag 仅仅是删除了 tag 对 manifests 文件的引用，并非真正将镜像删除掉。如果想要删除镜像的 layer 还是需要通过 registry GC 的方式，可参考之前写过的一篇博客《[Docker Registry GC 原理分析](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzg3ODAzMTMyNQ%3D%3D%26mid%3D2247486616%26idx%3D1%26sn%3D1a83dc17b6ba5dbc033b9da573ea8089%26chksm%3Dcf18b42af86f3d3c8e2e3a1005e4f9e176fdfcaf2d707bb5ead3745355a1f0c4ffee3457adde%26scene%3D21%23wechat_redirect)》。

```text
$ skopeo delete docker://hub.k8s.li/library/pause:3.2 --debug
DEBU[0000] Returning credentials from /home/release/.docker/config.json
DEBU[0000] Using registries.d directory /etc/containers/registries.d for sigstore configuration
DEBU[0000]  No signature storage configuration found for hub.k8s.li/library/pause:3.2
DEBU[0000] Looking for TLS certificates and private keys in /etc/docker/certs.d/hub.k8s.li/library
DEBU[0000] Loading registries configuration "/etc/containers/registries.conf"
DEBU[0000] GET https://hub.k8s.li/library/v2/
DEBU[0000] Ping https://hub.k8s.li/library/v2/ status 401
DEBU[0000] GET https://hub.k8s.li/library/v2/library/pause/manifests/3.2
DEBU[0000] DELETE https://hub.k8s.li/library/v2/library/pause/manifests/sha256:4a1c4b21597c1b4415bdbecb28a3296c6b5e23ca4f9feeb599860a1dac6a0108
```

## **skopeo list-tags**

这个命令可用于列出 Registry 上的某个镜像的所有 tag ，使用标准的 Registry API 来获取镜像 tag：

```text
$ skopeo list-tags docker://k8s.gcr.io/pause
{
    "Repository": "k8s.gcr.io/pause",
    "Tags": [
        "0.8.0",
        "1.0",
        "2.0",
        "3.0",
        "3.1",
        "3.2",
        "3.3",
        "3.4.1",
        "3.5",
        "go",
        "latest",
        "test",
        "test2"
    ]
}
```

## **K8sMeetup最佳实践**

一下给出几个 Skopeo 工具的最佳实践 。**镜像同步**假如给你一个镜像列表，如[https://github.com/kubesphere/ks-installer/blob/master/scripts/images-list.txt](https://link.zhihu.com/?target=https%3A//github.com/kubesphere/ks-installer/blob/master/scripts/images-list.txt)。  
如何将它快速地从一个 Registry 同步到另一个 Registry 中呢？还是 `skopeo copy` 走起！

- images-list.txt

```text
##k8s-images
kubesphere/kube-apiserver:v1.20.6
kubesphere/kube-scheduler:v1.20.6
kubesphere/kube-proxy:v1.20.6
kubesphere/kube-controller-manager:v1.20.6
kubesphere/kube-apiserver:v1.19.8
kubesphere/kube-scheduler:v1.19.8
kubesphere/kube-proxy:v1.19.8
kubesphere/kube-controller-manager:v1.19.8
kubesphere/kube-apiserver:v1.19.9
kubesphere/kube-scheduler:v1.19.9
kubesphere/kube-proxy:v1.19.9
kubesphere/kube-controller-manager:v1.19.9
kubesphere/kube-apiserver:v1.18.8
kubesphere/kube-scheduler:v1.18.8
kubesphere/kube-proxy:v1.18.8
kubesphere/kube-controller-manager:v1.18.8
kubesphere/kube-apiserver:v1.17.9
kubesphere/kube-scheduler:v1.17.9
kubesphere/kube-proxy:v1.17.9
kubesphere/kube-controller-manager:v1.17.9
kubesphere/pause:3.1
kubesphere/pause:3.2
kubesphere/etcd:v3.4.13
calico/cni:v3.16.3
calico/kube-controllers:v3.16.3
calico/node:v3.16.3
calico/pod2daemon-flexvol:v3.16.3
calico/typha:v3.16.3
kubesphere/flannel:v0.12.0
coredns/coredns:1.6.9
kubesphere/k8s-dns-node-cache:1.15.12
openebs/provisioner-localpv:2.10.1
openebs/linux-utils:2.10.0
kubesphere/nfs-client-provisioner:v3.1.0-k8s1.11
##csi-images
csiplugin/csi-neonsan:v1.2.0
csiplugin/csi-neonsan-ubuntu:v1.2.0
csiplugin/csi-neonsan-centos:v1.2.0
csiplugin/csi-provisioner:v1.5.0
csiplugin/csi-attacher:v2.1.1
csiplugin/csi-resizer:v0.4.0
csiplugin/csi-snapshotter:v2.0.1
csiplugin/csi-node-driver-registrar:v1.2.0
csiplugin/csi-qingcloud:v1.2.0
##kubesphere-images
kubesphere/ks-apiserver:v3.1.1
kubesphere/ks-console:v3.1.1
kubesphere/ks-controller-manager:v3.1.1
kubesphere/ks-installer:v3.1.1
kubesphere/kubectl:v1.20.0
kubesphere/kubectl:v1.19.0
redis:5.0.12-alpine
alpine:3.14
haproxy:2.0.22-alpine
nginx:1.14-alpine
minio/minio:RELEASE.2019-08-07T01-59-21Z
minio/mc:RELEASE.2019-08-07T23-14-43Z
mirrorgooglecontainers/defaultbackend-amd64:1.4
kubesphere/nginx-ingress-controller:v0.35.0
osixia/openldap:1.3.0
csiplugin/snapshot-controller:v3.0.3
kubesphere/kubefed:v0.7.0
kubesphere/tower:v0.2.0
kubesphere/prometheus-config-reloader:v0.42.1
kubesphere/prometheus-operator:v0.42.1
prom/alertmanager:v0.21.0
prom/prometheus:v2.26.0
prom/node-exporter:v0.18.1
kubesphere/ks-alerting-migration:v3.1.0
jimmidyson/configmap-reload:v0.3.0
kubesphere/notification-manager-operator:v1.0.0
kubesphere/notification-manager:v1.0.0
kubesphere/metrics-server:v0.4.2
kubesphere/kube-rbac-proxy:v0.8.0
kubesphere/kube-state-metrics:v1.9.7
openebs/provisioner-localpv:2.3.0
thanosio/thanos:v0.18.0
grafana/grafana:7.4.3
##kubesphere-logging-images
kubesphere/elasticsearch-oss:6.7.0-1
kubesphere/elasticsearch-curator:v5.7.6
kubesphere/fluentbit-operator:v0.5.0
kubesphere/fluentbit-operator:migrator
kubesphere/fluent-bit:v1.6.9
elastic/filebeat:6.7.0
kubesphere/kube-auditing-operator:v0.1.2
kubesphere/kube-auditing-webhook:v0.1.2
kubesphere/kube-events-exporter:v0.1.0
kubesphere/kube-events-operator:v0.1.0
kubesphere/kube-events-ruler:v0.2.0
kubesphere/log-sidecar-injector:1.1
docker:19.03
##istio-images
istio/pilot:1.6.10
istio/proxyv2:1.6.10
jaegertracing/jaeger-agent:1.17
jaegertracing/jaeger-collector:1.17
jaegertracing/jaeger-es-index-cleaner:1.17
jaegertracing/jaeger-operator:1.17.1
jaegertracing/jaeger-query:1.17
kubesphere/kiali:v1.26.1
kubesphere/kiali-operator:v1.26.1
##kubesphere-devops-images
kubesphere/ks-jenkins:2.249.1
jenkins/jnlp-slave:3.27-1
kubesphere/s2ioperator:v3.1.0
kubesphere/s2irun:v2.1.1
kubesphere/builder-base:v3.1.0
kubesphere/builder-nodejs:v3.1.0
kubesphere/builder-maven:v3.1.0
kubesphere/builder-go:v3.1.0
kubesphere/s2i-binary:v2.1.0
kubesphere/tomcat85-java11-centos7:v2.1.0
kubesphere/tomcat85-java11-runtime:v2.1.0
kubesphere/tomcat85-java8-centos7:v2.1.0
kubesphere/tomcat85-java8-runtime:v2.1.0
kubesphere/java-11-centos7:v2.1.0
kubesphere/java-8-centos7:v2.1.0
kubesphere/java-8-runtime:v2.1.0
kubesphere/java-11-runtime:v2.1.0
kubesphere/nodejs-8-centos7:v2.1.0
kubesphere/nodejs-6-centos7:v2.1.0
kubesphere/nodejs-4-centos7:v2.1.0
kubesphere/python-36-centos7:v2.1.0
kubesphere/python-35-centos7:v2.1.0
kubesphere/python-34-centos7:v2.1.0
kubesphere/python-27-centos7:v2.1.0
##openpitrix-images
kubespheredev/openpitrix-jobs:v3.1.1
##weave-scope-images
weaveworks/scope:1.13.0
##kubeedge-images
kubeedge/cloudcore:v1.6.2
kubesphere/edge-watcher:v0.1.0
kubesphere/kube-rbac-proxy:v0.5.0
kubesphere/edge-watcher-agent:v0.1.0
##example-images-images
kubesphere/examples-bookinfo-productpage-v1:1.16.2
kubesphere/examples-bookinfo-reviews-v1:1.16.2
kubesphere/examples-bookinfo-reviews-v2:1.16.2
kubesphere/examples-bookinfo-reviews-v3:1.16.2
kubesphere/examples-bookinfo-details-v1:1.16.2
kubesphere/examples-bookinfo-ratings-v1:1.16.3
busybox:1.31.1
joosthofman/wget:1.0
kubesphere/netshoot:v1.0
nginxdemos/hello:plain-text
wordpress:4.8-apache
mirrorgooglecontainers/hpa-example:latest
java:openjdk-8-jre-alpine
fluent/fluentd:v1.4.2-2.0
perl:latest
```

- sync.sh

```text
#!/bin/bash
GREEN_COL="\\033[32;1m"
RED_COL="\\033[1;31m"
NORMAL_COL="\\033[0;39m"

SOURCE_REGISTRY=$1
TARGET_REGISTRY=$2

: ${IMAGES_LIST_FILE:="images-list.txt"}
: ${TARGET_REGISTRY:="hub.k8s.li"}
: ${SOURCE_REGISTRY:="docker.io"}

BLOBS_PATH="docker/registry/v2/blobs/sha256"
REPO_PATH="docker/registry/v2/repositories"

set -eo pipefail

CURRENT_NUM=0
ALL_IMAGES="$(sed -n '/#/d;s/:/:/p' ${IMAGES_LIST_FILE} | sort -u)"
TOTAL_NUMS=$(echo "${ALL_IMAGES}" | wc -l)

skopeo_copy() {
 if skopeo copy --insecure-policy --src-tls-verify=false --dest-tls-verify=false \
 --override-arch amd64 --override-os linux -q docker://$1 docker://$2; then
 echo -e "$GREEN_COL Progress: ${CURRENT_NUM}/${TOTAL_NUMS} sync $1 to $2 successful $NORMAL_COL"
 else
 echo -e "$RED_COL Progress: ${CURRENT_NUM}/${TOTAL_NUMS} sync $1 to $2 failed $NORMAL_COL"
 exit 2
 fi
}

for image in ${ALL_IMAGES}; do
 let CURRENT_NUM=${CURRENT_NUM}+1
 skopeo_copy ${SOURCE_REGISTRY}/${image} ${TARGET_REGISTRY}/${image}
done
```

- `bash sync.sh docker.io localhost:5000`

```text
$ bash sync.sh docker.io localhost:5000
Progress: 1/143 sync docker.io/alpine:3.14 to localhost:5000/alpine:3.14 successful
Progress: 2/143 sync docker.io/busybox:1.31.1 to localhost:5000/busybox:1.31.1 successful
Progress: 3/143 sync docker.io/calico/cni:v3.16.3 to localhost:5000/calico/cni:v3.16.3 successful
Progress: 4/143 sync docker.io/calico/kube-controllers:v3.16.3 to localhost:5000/calico/kube-controllers:v3.16.3 successful
Progress: 141/143 sync docker.io/thanosio/thanos:v0.18.0 to localhost:5000/thanosio/thanos:v0.18.0 successful
 Progress: 142/143 sync docker.io/weaveworks/scope:1.13.0 to localhost:5000/weaveworks/scope:1.13.0 successful
 Progress: 143/143 sync docker.io/wordpress:4.8-apache to localhost:5000/wordpress:4.8-apache successful
```

## **使用 Registry 存储特性**

将镜像从 Registry 中同步到本地目录，使用 Registry 存储的特性，将本地目录中的镜像转换成 Registry 存储的格式。这样子的好处就是可以去除一些 `skopeo dir` 中重复的 `layers`，减少镜像的总大小。具体的原理可以参考我之前写过的 《如何使用 registry 存储的特性》。

- sync.sh

```text
#!/bin/bash
GREEN_COL="\\033[32;1m"
RED_COL="\\033[1;31m"
NORMAL_COL="\\033[0;39m"

SOURCE_REGISTRY=$1
TARGET_REGISTRY=$2
IMAGES_DIR=$2

: ${IMAGES_DIR:="images"}
: ${IMAGES_LIST_FILE:="images-list.txt"}
: ${TARGET_REGISTRY:="hub.k8s.li"}
: ${SOURCE_REGISTRY:="docker.io"}

BLOBS_PATH="docker/registry/v2/blobs/sha256"
REPO_PATH="docker/registry/v2/repositories"

set -eo pipefail

CURRENT_NUM=0
ALL_IMAGES="$(sed -n '/#/d;s/:/:/p' ${IMAGES_LIST_FILE} | sort -u)"
TOTAL_NUMS=$(echo "${ALL_IMAGES}" | wc -l)

skopeo_sync() {
 if skopeo sync --insecure-policy --src-tls-verify=false --dest-tls-verify=false \
 --override-arch amd64 --override-os linux --src docker --dest dir $1 $2 > /dev/null; then
 echo -e "$GREEN_COL Progress: ${CURRENT_NUM}/${TOTAL_NUMS} sync $1 to $2 successful $NORMAL_COL"
 else
 echo -e "$RED_COL Progress: ${CURRENT_NUM}/${TOTAL_NUMS} sync $1 to $2 failed $NORMAL_COL"
 exit 2
 fi
}

convert_images() {
 rm -rf ${IMAGES_DIR}; mkdir -p ${IMAGES_DIR}
 for image in ${ALL_IMAGES}; do
 let CURRENT_NUM=${CURRENT_NUM}+1
 image_name=${image%%:*}
 image_tag=${image##*:}
 image_repo=${image%%/*}
 skopeo_sync ${SOURCE_REGISTRY}/${image} ${IMAGES_DIR}/${image_repo}
 manifest="${IMAGES_DIR}/${image}/manifest.json"
 manifest_sha256=$(sha256sum ${manifest} | awk '{print $1}')
 mkdir -p ${BLOBS_PATH}/${manifest_sha256:0:2}/${manifest_sha256}
 ln -f ${manifest} ${BLOBS_PATH}/${manifest_sha256:0:2}/${manifest_sha256}/data

 # make image repositories dir
 mkdir -p ${REPO_PATH}/${image_name}/{_uploads,_layers,_manifests}
 mkdir -p ${REPO_PATH}/${image_name}/_manifests/revisions/sha256/${manifest_sha256}
 mkdir -p ${REPO_PATH}/${image_name}/_manifests/tags/${image_tag}/{current,index/sha256}
 mkdir -p ${REPO_PATH}/${image_name}/_manifests/tags/${image_tag}/index/sha256/${manifest_sha256}

 # create image tag manifest link file
 echo -n "sha256:${manifest_sha256}" > ${REPO_PATH}/${image_name}/_manifests/tags/${image_tag}/current/link
 echo -n "sha256:${manifest_sha256}" > ${REPO_PATH}/${image_name}/_manifests/revisions/sha256/${manifest_sha256}/link
 echo -n "sha256:${manifest_sha256}" > ${REPO_PATH}/${image_name}/_manifests/tags/${image_tag}/index/sha256/${manifest_sha256}/link

 # link image layers file to registry blobs dir
 for layer in $(sed '/v1Compatibility/d' ${manifest} | grep -Eo "\b[a-f0-9]{64}\b"); do
 mkdir -p ${BLOBS_PATH}/${layer:0:2}/${layer}
 mkdir -p ${REPO_PATH}/${image_name}/_layers/sha256/${layer}
 echo -n "sha256:${layer}" > ${REPO_PATH}/${image_name}/_layers/sha256/${layer}/link
 ln -f ${IMAGES_DIR}/${image}/${layer} ${BLOBS_PATH}/${layer:0:2}/${layer}/data
 done
 done
}

convert_images
```

- install.sh

使用这个脚本将 Registry 存储中的镜像转换成 skopeo dir 的方式，然后再将镜像同步到 Registry 中。

```text
#!/bin/bash
REGISTRY_DOMAIN="harbor.k8s.li"
REGISTRY_PATH="/var/lib/registry"

# 切换到 registry 存储主目录下
cd ${REGISTRY_PATH}

gen_skopeo_dir() {
   # 定义 registry 存储的 blob 目录 和 repositories 目录，方便后面使用
    BLOB_DIR="docker/registry/v2/blobs/sha256"
    REPO_DIR="docker/registry/v2/repositories"
    # 定义生成 skopeo 目录
    SKOPEO_DIR="docker/skopeo"
    # 通过 find 出 current 文件夹可以得到所有带 tag 的镜像，因为一个 tag 对应一个 current 目录
    for image in $(find ${REPO_DIR} -type d -name "current"); do
        # 根据镜像的 tag 提取镜像的名字
        name=$(echo ${image} | awk -F '/' '{print $5"/"$6":"$9}')
        link=$(cat ${image}/link | sed 's/sha256://')
        mfs="${BLOB_DIR}/${link:0:2}/${link}/data"
        # 创建镜像的硬链接需要的目录
        mkdir -p "${SKOPEO_DIR}/${name}"
        # 硬链接镜像的 manifests 文件到目录的 manifest 文件
        ln ${mfs} ${SKOPEO_DIR}/${name}/manifest.json
        # 使用正则匹配出所有的 sha256 值，然后排序去重
        layers=$(grep -Eo "\b[a-f0-9]{64}\b" ${mfs} | sort -n | uniq)
        for layer in ${layers}; do
          # 硬链接 registry 存储目录里的镜像 layer 和 images config 到镜像的 dir 目录
            ln ${BLOB_DIR}/${layer:0:2}/${layer}/data ${SKOPEO_DIR}/${name}/${layer}
        done
    done
}

sync_image() {
    # 使用 skopeo sync 将 dir 格式的镜像同步到 harbor
    for project in $(ls ${SKOPEO_DIR}); do
        skopeo sync --insecure-policy --src-tls-verify=false --dest-tls-verify=false \
        --src dir --dest docker ${SKOPEO_DIR}/${project} ${REGISTRY_DOMAIN}/${project}
    done
}

gen_skopeo_dir
sync_image
```

## **从 Registry 存储中 select 出镜像**

先将镜像同步到一个 Registry 中，再将镜像从 Registry 存储中捞出来。这个 Registry 可以当作一个镜像存储的池子，我们使用 Linux 中硬链接的特性将镜像"复制"一份出来，然后再打一个 tar 包。  
这样做的好处就是每次打包镜像的时候都能复用历史的镜像数据，而且性能极快。具体的原理可以参考我之前的博客《[什么？发布流水线中镜像“同步”速度又提升了 15 倍 ！](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzg3ODAzMTMyNQ%3D%3D%26mid%3D2247492554%26idx%3D1%26sn%3D78ab4066efdb6ee031eae3c349021806%26chksm%3Dcf1b5b78f86cd26efcf2a4d26efddfd1d27ed2fc58b4ba5876857c9c7408f725b06b70bdbe20%26scene%3D21%23wechat_redirect)》。

- 先将镜像同步到一个固定的 Registry 中：

```text
$ bash sync.sh docker.io localhost:5000
```

- 再使用该脚本将镜像从 Registry 存储中捞出来：

```text
#!/bin/bash
set -eo pipefail

IMAGES_LIST="$1"
REGISTRY_PATH="$2"
OUTPUT_DIR="$3"
BLOB_DIR="docker/registry/v2/blobs/sha256"
REPO_DIR="docker/registry/v2/repositories"

rm -rf ${OUTPUT_DIR}; mkdir -p ${OUTPUT_DIR}
for image in $(find ${IMAGES_LIST} -type f -name "*.list" | xargs grep -Ev '^#|^/' | grep ':'); do
    image_tag=${image##*:}
    image_name=${image%%:*}
    tag_link=${REGISTRY_PATH}/${REPO_DIR}/${image_name}/_manifests/tags/${image_tag}/current/link
    manifest_sha256=$(sed 's/sha256://' ${tag_link})
    manifest=${REGISTRY_PATH}/${BLOB_DIR}/${manifest_sha256:0:2}/${manifest_sha256}/data
    mkdir -p ${OUTPUT_DIR}/${BLOB_DIR}/${manifest_sha256:0:2}/${manifest_sha256}
    ln -f ${manifest} ${OUTPUT_DIR}/${BLOB_DIR}/${manifest_sha256:0:2}/${manifest_sha256}/data

    # make image repositories dir
    mkdir -p ${OUTPUT_DIR}/${REPO_DIR}/${image_name}/{_uploads,_layers,_manifests}
    mkdir -p ${OUTPUT_DIR}/${REPO_DIR}/${image_name}/_manifests/revisions/sha256/${manifest_sha256}
    mkdir -p ${OUTPUT_DIR}/${REPO_DIR}/${image_name}/_manifests/tags/${image_tag}/{current,index/sha256}
    mkdir -p ${OUTPUT_DIR}/${REPO_DIR}/${image_name}/_manifests/tags/${image_tag}/index/sha256/${manifest_sha256}

    # create image tag manifest link file
    echo -n "sha256:${manifest_sha256}" > ${OUTPUT_DIR}/${REPO_DIR}/${image_name}/_manifests/tags/${image_tag}/current/link
    echo -n "sha256:${manifest_sha256}" > ${OUTPUT_DIR}/${REPO_DIR}/${image_name}/_manifests/revisions/sha256/${manifest_sha256}/link
    echo -n "sha256:${manifest_sha256}" > ${OUTPUT_DIR}/${REPO_DIR}/${image_name}/_manifests/tags/${image_tag}/index/sha256/${manifest_sha256}/link
    for layer in $(sed '/v1Compatibility/d' ${manifest} | grep -Eo '\b[a-f0-9]{64}\b' | sort -u); do
        mkdir -p ${OUTPUT_DIR}/${BLOB_DIR}/${layer:0:2}/${layer}
        mkdir -p ${OUTPUT_DIR}/${REPO_DIR}/${image_name}/_layers/sha256/${layer}
        ln -f ${BLOB_DIR}/${layer:0:2}/${layer}/data ${OUTPUT_DIR}/${BLOB_DIR}/${layer:0:2}/${layer}/data
        echo -n "sha256:${layer}" > ${OUTPUT_DIR}/${REPO_DIR}/${image_name}/_layers/sha256/${layer}/link
    done
done
```


