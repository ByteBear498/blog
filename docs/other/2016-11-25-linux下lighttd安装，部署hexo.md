---
title: linux下lighttd安装，部署hexo
date: 2016-11-25 01:14:57
categories: 
 - other
---

### 安装第三方软件包
下载第三方软件包
64位

`wget http://epel.mirror.net.in/epel/6/x86_64/epel-release-6-8.noarch.rpm`

32位

`wget http://epel.mirror.net.in/epel/6/i386/epel-release-6-8.noarch.rpm`

安装第三方软件包

`rpm -ivh epel-release-6-8.noarch.rpm`

### 安装lighttpd
`yum install lighttpd`

### lighttpd部署hexo
新建lighttpd.conf配置文件

`touch lighttpd.conf`

输入如下内容
```
server.document-root = "/root/blog/public"

server.port = 80

mimetype.assign             = (
  ".pdf"          =>      "application/pdf",
  ".sig"          =>      "application/pgp-signature",
  ".spl"          =>      "application/futuresplash",
  ".class"        =>      "application/octet-stream",
  ".ps"           =>      "application/postscript",
  ".torrent"      =>      "application/x-bittorrent",
  ".dvi"          =>      "application/x-dvi",
  ".gz"           =>      "application/x-gzip",
  ".pac"          =>      "application/x-ns-proxy-autoconfig",
  ".swf"          =>      "application/x-shockwave-flash",
  ".tar.gz"       =>      "application/x-tgz",
  ".tgz"          =>      "application/x-tgz",
  ".tar"          =>      "application/x-tar",
  ".zip"          =>      "application/zip",
  ".mp3"          =>      "audio/mpeg",
  ".m3u"          =>      "audio/x-mpegurl",
  ".wma"          =>      "audio/x-ms-wma",
  ".wax"          =>      "audio/x-ms-wax",
  ".ogg"          =>      "application/ogg",
  ".wav"          =>      "audio/x-wav",
  ".gif"          =>      "image/gif",
  ".jpg"          =>      "image/jpeg",
  ".jpeg"         =>      "image/jpeg",
  ".png"          =>      "image/png",
  ".xbm"          =>      "image/x-xbitmap",
  ".xpm"          =>      "image/x-xpixmap",
  ".xwd"          =>      "image/x-xwindowdump",
  ".css"          =>      "text/css",
  ".html"         =>      "text/html",
  ".htm"          =>      "text/html",
  ".js"           =>      "text/javascript",
  ".asc"          =>      "text/plain",
  ".c"            =>      "text/plain",
  ".cpp"          =>      "text/plain",
  ".log"          =>      "text/plain",
  ".conf"         =>      "text/plain",
  ".text"         =>      "text/plain",
  ".txt"          =>      "text/plain",
  ".spec"         =>      "text/plain",
  ".dtd"          =>      "text/xml",
  ".xml"          =>      "text/xml",
  ".mpeg"         =>      "video/mpeg",
  ".mpg"          =>      "video/mpeg",
  ".mov"          =>      "video/quicktime",
  ".qt"           =>      "video/quicktime",
  ".avi"          =>      "video/x-msvideo",
  ".asf"          =>      "video/x-ms-asf",
  ".asx"          =>      "video/x-ms-asf",
  ".wmv"          =>      "video/x-ms-wmv",
  ".bz2"          =>      "application/x-bzip",
  ".tbz"          =>      "application/x-bzip-compressed-tar",
  ".tar.bz2"      =>      "application/x-bzip-compressed-tar",
  ".odt"          =>      "application/vnd.oasis.opendocument.text", 
  ".ods"          =>      "application/vnd.oasis.opendocument.spreadsheet", 
  ".odp"          =>      "application/vnd.oasis.opendocument.presentation", 
  ".odg"          =>      "application/vnd.oasis.opendocument.graphics", 
  ".odc"          =>      "application/vnd.oasis.opendocument.chart", 
  ".odf"          =>      "application/vnd.oasis.opendocument.formula", 
  ".odi"          =>      "application/vnd.oasis.opendocument.image", 
  ".odm"          =>      "application/vnd.oasis.opendocument.text-master", 
  ".ott"          =>      "application/vnd.oasis.opendocument.text-template",
  ".ots"          =>      "application/vnd.oasis.opendocument.spreadsheet-template",
  ".otp"          =>      "application/vnd.oasis.opendocument.presentation-template",
  ".otg"          =>      "application/vnd.oasis.opendocument.graphics-template",
  ".otc"          =>      "application/vnd.oasis.opendocument.chart-template",
  ".otf"          =>      "application/vnd.oasis.opendocument.formula-template",
  ".oti"          =>      "application/vnd.oasis.opendocument.image-template",
  ".oth"          =>      "application/vnd.oasis.opendocument.text-web",

# make the default mime type application/octet-stream.
  ""              =>      "application/octet-stream",
)
```

详细的mimetype.assign可参考官网——http://redmine.lighttpd.net/projects/lighttpd/wiki/Mimetype_assignDetails
### 启动lighttpd
`lighttpd -D -f lighttpd.conf`

这里的-D是指在当前窗口运行，不加-D就是在后台运行的。