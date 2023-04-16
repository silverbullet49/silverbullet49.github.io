# What is the digest?

The Digest is the sha256 hash value of the images to prevent corruption of the contents.

when we make a container image using `docker build`, docker engine runs sha256 hash function to create the hash value of the layers. it's **container image id, not digest**.

you can see the IMAGE ID field when you run `docker images` below. it's the container image id. `docker inspect` shows id field in json, which is same as IMAGE ID field in `docker images`

```plaintext
$ sudo docker images
REPOSITORY                    TAG                  IMAGE ID       CREATED         SIZE
leekasong/nginx-custom        v1                   59d51481f59f   11 days ago     128MB
nginx                         custom-kslee         59d51481f59f   11 days ago     128MB
docker101tutorial             latest               b39c703f069c   2 weeks ago     26.7MB
leekasong/docker101tutorial   latest               b39c703f069c   2 weeks ago     26.7MB
kindest/node                  <none>               8a36bcc3fcb4   2 weeks ago     1.07GB
nginx                         latest               ab2a5aa39300   3 weeks ago     126MB
alpine/git                    latest               cfd9fa28a348   3 weeks ago     25.2MB
kindest/node                  <none>               522f68bf7ee2   4 weeks ago     1.09GB
kindest/haproxy               v20200708-548e36db   3007f6a20bad   11 months ago   24.3MB
nginx                         1.14.2               7bbc8783b8ec   2 years ago     103MB
```

```plaintext
$ sudo docker inspect nginx:custom-kslee
[
    {
        "Id": "sha256:59d51481f59fe1c0ef6ebaea0e18a630794e63328cbb5981aa78b4cfb9bafde3",
        "RepoTags": [
            "leekasong/nginx-custom:v1",
            "nginx:custom-kslee"
        ],
        "RepoDigests": [
            "leekasong/nginx-custom@sha256:bc3ed28f16aed700769f60101a840f9049eabb03b5212c9261c7b119f322a869"
        ],
        "Parent": "",
(...)
```

# So, what the hell is the digest?

Actually, we have to define some terms first : **layer, manifest, index**. it helps us to understand why the container image has many hash values and what are used for.

## Index, manifest, layer

this is the picture of the container image internal.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668215742970/vmF5UcCm2.png align="left")

### layer

The layer is a basic element of the image. it consists of some configurations for the image, such as root filesystem, namespaces, shell, init process, and any packages.

### manifest

The manifest consists of the configuration and layers. this is what we call 'image': a list of the layer.

```plaintext
$ sudo docker manifest inspect nginx:custom-kslee
{
	"schemaVersion": 2,
	"mediaType": "application/vnd.docker.distribution.manifest.v2+json",
	"config": {
		"mediaType": "application/vnd.docker.container.image.v1+json",
		"size": 6836,
		"digest": "sha256:59d51481f59fe1c0ef6ebaea0e18a630794e63328cbb5981aa78b4cfb9bafde3"
	},
	"layers": [
		{
			"mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
			"size": 25911250,
			"digest": "sha256:fcad0c936ea5c12e1c8c4edb81a97c0cde04ee71e7067ee3b246474cf1854d7a"
		},
		{
			"mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
			"size": 27136816,
			"digest": "sha256:9aedff44070167cabd40e81eca493c2857f8125beed7822200da6f5e1be816af"
		},
		{
			"mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
			"size": 603,
			"digest": "sha256:07b2fdc00c51b9a1dd9acef3b0a790d2867068dae78819d36e574f9ec07d5cde"
		},
		{
			"mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
			"size": 891,
			"digest": "sha256:5aa278fdb263628da6921e94804c5bdc6a1f01fdd0c849890e210039bf8c6f89"
		},
		{
			"mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
			"size": 663,
			"digest": "sha256:9501d14a6216409ad48f38c46958e0615f7c10dfee7018d58ad05093573dcfa5"
		},
		{
			"mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
			"size": 1392,
			"digest": "sha256:7298f75ff08b1761a25d2d2d3e71a31e734048ac8db0881c82fcc32deddd2e0d"
		}
	]
}
```

### index

the index is a list of the manifest.

index is used for supporting mutiple architectures of the container image.

As people used docker image more, docker group had to consider about different environments: kernel(fedora, debian, unix...), cpu architecture.(intel, amd, or 32 bits/64bits...)

so, it is neccessary to make a distributed repository to provide variaous version of the container image even if the purpose of the image is same.(i want to use python3 in my environment, but docker registry can ask 'what's your environment? macos, linux, windows or amd64, m1?' )

so, usually the distributed official registry has variaous 'manifest' for the single image. you can see the various hash value of the single ruby image in the docker hub.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668216429398/dC_k2E-ML.png align="left")

# So, Digest is a hash value of manifest.

what we usuall call 'digest of image' refer to the manifest digest of the image. A image digest is a result of hashing manifest.

so, the digest of the manifest is used for identifying the image. i mean when we pull/push the image from/to the image registry, we need to have a field to indentify the image and that field is the manifest digest. so, the manifest digest must be unique field at every registry, including official, unofficial, private registries whatever.

# Note: `docker pull` shows Index Digest

```plaintext
$ sudo docker pull ruby:latest
latest: Pulling from library/ruby
c54d9402d498: Pull complete 
6a8aff564b22: Pull complete 
67e893e0778b: Pull complete 
72bf2dd6193d: Pull complete 
71f77eac104c: Pull complete 
9a79dd4ab436: Pull complete 
d66e5d9b687b: Pull complete 
de50cd8eaefe: Pull complete 
Digest: sha256:9ed07720a7cfcfdff947d998ff2164381542b9049ecae7c15ae7d9757a7e4dfd
Status: Downloaded newer image for ruby:latest
docker.io/library/ruby:latest
```

```plaintext
$ sudo docker manifest inspect ruby:latest
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
   "manifests": [
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 2002,
         "digest": "sha256:d5c828809239010c8549eeaf1f6da84f67bee7f61353e6a4a52159bf3f397aa6",
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 2002,
         "digest": "sha256:5ea5fa6f13598f9c72f6d82fc24f85e24e71608f1a558df86d7cc22a37e6d074",
         "platform": {
            "architecture": "arm",
            "os": "linux",
            "variant": "v5"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 2002,
         "digest": "sha256:bfb87ed918bf4b4c475e71b15662cf9e15665d2bf11137fb1493dd8319ea7ec7",
         "platform": {
            "architecture": "arm",
            "os": "linux",
            "variant": "v7"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 2002,
         "digest": "sha256:df2a92838643cd05156d3d7773979d618d9240f651c1e66d9f12bb107f05e562",
         "platform": {
            "architecture": "arm64",
            "os": "linux",
            "variant": "v8"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 2003,
         "digest": "sha256:182a1761ba26978aef77b652615b5647b18f4a55ea2a2a7fc0e35abdd1da0109",
         "platform": {
            "architecture": "386",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 2003,
         "digest": "sha256:e71379d475bff7f5db85b5a3817066e943bab5017a4538be82b8ed497b40756a",
         "platform": {
            "architecture": "mips64le",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 2003,
         "digest": "sha256:1c93484d76ba7abeb48cf59e2fe1a0853befd6572033f6ba2acb48466398ed22",
         "platform": {
            "architecture": "ppc64le",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 2002,
         "digest": "sha256:64e84adefcbf51124ea795a10c8bc769eae8218406ae85d33c44307c0f429a75",
         "platform": {
            "architecture": "s390x",
            "os": "linux"
         }
      }
   ]
}
```

you can see that the digest from `docker pull` is different from any other manifest digest from the result of `docker manifest inspect` it's because the digest from `docker pull` is **the index digest**. And you can also see that the manifest digests are same as the hash values of the ruby latest image in the docker hub.

# Conclusion

To verify image, docker use the digest for the various elements. when we pull the image from distributed registry, docker use the index digest. But actual image that you will use is the manifest, so you can verify the image with the manifest digest.