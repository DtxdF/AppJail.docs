An ajspec file provides information about the image, such as the checksum, the site where the image will be downloaded, etc. The syntax of the ajspec file is the same as used in templates.

The following table shows detailed information about ajspec parameters. The `Optional` column specifies whether a parameter is mandatory or not for some commands such as `appjail image update` and `appjail image import`. The `Multiple` column specifies whether a parameter can be specified multiple times or not.

| Parameter | Optional | Multiple | Description  |
| --- | --- | --- | --- |
| `<tag>.name` | Yes | No | Image name. Although it is optional it is highly recommended to have it set as it is not, the user will have to specify one when importing. |
| `<tag>.timestamp.<arch>` | No | No | Timestamp specifiying when the image was last updated. |
| `<tag>.maintainer` | Yes | Yes | In charge of maintaining the image and everything related to it. |
| `<tag>.comment` | Yes | No | One-line description |
| `<tag>.url` | Yes | No | Home page project or a website that provides more information about the image for the specific tag |
| `<tag>.description` | Yes | Yes | Long description. |
| `<tag>.sum.<arch>` | No | No | Checksum. |
| `<tag>.source.<arch>` | No | Yes | Sites where the image will be downloaded. If the first one fails, AppJail will try the second one and if it fails, AppJail will try the third one, and so on. |
| `<tag>.size.<arch>` | Yes | No | Compressed image size. |
| `entrypoint` | No | No | It is used by `appjail image update` to retrieve the ajspec when `appjail image import` is called and set by the latter. |
| `ajspec` | Yes | No | Ajspec filename used by git-like methods. |
| `<tag>.arch` | No | Yes | Architectures allowed by this image. |
| `tags` | No | Yes | Image tags. |

As you can see there are some things to talk about, such as `<tag>` and `<arch>`. `<tag>` specifies the image tag and `<arch>` the image architecture. It is important to remember to add the image tag to `tags` and the image architecture to `<tag>.arch` for AppJail recognize them. See `appjail image` for more details.

You probably won't need to manipulate an ajspec file many times, especially if you only want to use the image, but for when you need to edit it, `appjail image metadata` if your best friend.

Before editing, it is worth looking at what we need to edit.

```console
# appjail image metadata info nginx
Name            :    nginx (132)
Maintainer(s)   :
  - Francis Ford Coppola <fcoppola@example.org>
  - Quentin Tarantino <qtarantino@example.org>
Build on        :
  - amd64
Image           :    amd64
  - SHA256 = 481f09014b94433430efb2f0da52c7bdd2cc4e9b478ac9336b06710dab9d2ea5
  - SIZE = 27360
  - TIMESTAMP = Tue Jun 13 20:05:19 2023
Source          :    amd64
  - http://192.168.1.105:8080/AppJail-images/nginx/132-amd64-image.appjail
Entrypoint      :    gh+DtxdF/nginx-image
AJSPEC          :    .ajspec
```

This image is descriptive, but a description and perhaps other data are very important, such as the home page and a one-line description.

```console
# appjail image metadata set -t 132 nginx comment="Robust and small WWW server"
# appjail image metadata set -t 132 nginx url="https://nginx.org/"
# cat << EOF | while IFS= read -r line; do appjail image metadata set -t 132 nginx description+="${line}"; done
NGINX is a high performance edge web server with the lowest memory footprint
and the key features to build modern and efficient web infrastructure.

NGINX functionality includes HTTP server, HTTP and mail reverse proxy, caching,
load balancing, compression, request throttling, connection multiplexing and
reuse, SSL offload and HTTP media streaming.
EOF
# appjail image metadata info nginx
Name            :    nginx (132)
Maintainer(s)   :
  - Francis Ford Coppola <fcoppola@example.org>
  - Quentin Tarantino <qtarantino@example.org>
Comment         :    Robust and small WWW server
Build on        :
  - amd64
Image           :    amd64
  - SHA256 = 481f09014b94433430efb2f0da52c7bdd2cc4e9b478ac9336b06710dab9d2ea5
  - SIZE = 27976978
  - TIMESTAMP = Tue Jun 13 20:05:19 2023
Source          :    amd64
  - http://localhost:8080/132-amd64-image.appjail
WWW             :    https://nginx.org/
Description     :
NGINX is a high performance edge web server with the lowest memory footprint
and the key features to build modern and efficient web infrastructure.

NGINX functionality includes HTTP server, HTTP and mail reverse proxy, caching,
load balancing, compression, request throttling, connection multiplexing and
reuse, SSL offload and HTTP media streaming.
Entrypoint      :    gh+DtxdF/nginx-image
AJSPEC          :    .ajspec
```

---

**See also**:

* [Images](intro.md)
