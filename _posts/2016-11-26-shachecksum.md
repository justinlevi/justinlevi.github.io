---
layout: post
title: "shasum Checksum function"
subtitle: "Verify the checksum of your download before you install a hacked or corrupt app"
bg-image: "post-bg.jpg"

date: 2016-11-26
---

##New PHPStorm Version released today.##

Jetbrains provides a checksum, and it's really a good idea to use it to verify that the file hasn't been corrupted during transmission.

Here's a function I have in my `~/.bash_profile`.

```
function sha256check () {
    if [[ $(pbpaste) == $(shasum -b -a 256 "$@" | awk '{print $1}') ]]; then echo 'match'; fi ;
}
```

source: http://stackoverflow.com/questions/36382599/fastest-way-to-compare-sha1-checksum-in-clipboard-with-sha1-of-local-file-using

After you save, make sure to run $`source ~/.bash_profile` on your terminal, otherwise the function won't be available.

After downloading the upgrade (dmg),

1. Open your terminal and type `sha256check `
2. Copy the hash portion of the checksum that JetBrains provides and then head back to your terminal
3. Find the download and drag the file onto the terminal window. This essentially just adds the path to the terminal instead of you having to type it. 

So in my case, the command looks like this:

$ `sha256check /Users/justinwinter/Downloads/PhpStorm-2016.3.dmg`
(Note: at this point I have already copied the hash from the JetBrains website)


And the response  is:

$ `sha256check /Users/justinwinter/Downloads/PhpStorm-2016.3.dmg`
`match`

This lets me know that the file is likely legit.






