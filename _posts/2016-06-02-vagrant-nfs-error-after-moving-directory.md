---
layout: post
title: "Vagrant/VM Error After Moving/Renaming Parent Directory"
subtitle: "Because it's always something..."
bg-image: "post-bg.jpg"

date: 2016-06-02
---

After moving a parent directory that contains your Vagrant config you might get the following error:

```
NFS is reporting that your exports file is invalid. Vagrant does
this check before making any changes to the file. Please correct
the issues below and execute "vagrant reload":

exports:8: path contains non-directory or non-existent components: /Users/username/path/to/shared/dir
exports:8: no usable directories in export entry
exports:8: using fallback (marked offline): /
```

To fix this, you can edit your /etc/export file with sudo privileges and do a vagrant reload

somewhat related source: http://stackoverflow.com/questions/20726248
