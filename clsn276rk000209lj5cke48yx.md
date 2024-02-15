---
title: "Fixing Catastrophic Failures, "Failed to translate", and 0x80071772 errors in WSL"
datePublished: Thu Feb 15 2024 10:09:28 GMT+0000 (Coordinated Universal Time)
cuid: clsn276rk000209lj5cke48yx
slug: fixing-catastrophic-failures-failed-to-translate-and-0x80071772-errors-in-wsl
tags: error, docker, windows, wsl

---

One day I turned on my computer, logged in, and was met with a Command Prompt window asking me to update WSL. I pressed the key to start the update, and then a few seconds later saw the dreaded "Catastrophic failure" message.

The update was being driven by Docker Desktop, which I had configured to use WSL rather than Hyper-V, as is now recommended by Docker.

I tried multiple things to get WSL to work, but I wasn't able to perform any other commands with WSL as anything I tried, e.g. `wsl --unregister ubuntu` would be met with the "You need to upgrade" message.

I tried various methods to uninstall WSL completely so I could do a fresh install but nothing would fix it.

Here I'll explain how I fixed the issues on my machine.

# Tl;dr Version

1. Follow the steps in [this guide](https://www.minitool.com/news/uninstall-wsl.html) to uninstall WSL
    
2. Installed WSL from the Microsoft Store [here](https://www.microsoft.com/store/productId/9P9TQF7MRM4R?ocid=pdpshare)
    
3. Uninstall Ubuntu
    
4. Ensure your `New apps will save to:` setting in Windows is set to your main drive
    
5. Run `wsl --install -d Ubuntu`
    
6. Run `wsl --setdefault Ubuntu`
    

# Long Version

Firstly I did the following

1. I followed the steps in [this guide](https://www.minitool.com/news/uninstall-wsl.html) to uninstall WSL
    
2. I installed WSL from the Microsoft Store [here](https://www.microsoft.com/store/productId/9P9TQF7MRM4R?ocid=pdpshare)
    

I was then able to at least run `wsl` without it asking to update, but I was then faced with another error: `<3>WSL (206) ERROR: CreateProcessParseCommon:731: Failed to translate`.

After a lot of reading it turned out this error was because:

1. Docker Desktop had set itself as the default distro for WSL, which was what was returning those errors when I tried to run `wsl` directly
    
2. Ubuntu had not been successfully installed as a distro when installing WSL through the Microsoft Store, which is the distro I wanted to use when I ran `wsl` via the command line
    

I found now that my Docker containers were working fine, but I couldn't use WSL to run Ubuntu.

Running `wsl --list` showed the following:

```bash
Windows Subsystem for Linux Distributions:
docker-desktop-data
docker-desktop (Default)
```

Here we can see that Ubuntu isn't listed, and Docker is set as the default.

I noticed that Windows had attempted to install Ubuntu (it was listed in my Windows applications):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707990392930/a68a9ac8-c762-40dd-8c34-6af8e8696cc6.png align="center")

I uninstalled this and then ran `wsl --install -d Ubuntu`.

I was then met with another error:

```bash
WslRegisterDistribution failed with error: 0x80071772
Error: 0x80071772 The specified file is encrypted and the user does not have the ability to decrypt it.
```

Now this one happened because in the past I had set my Windows apps to be installed on a different drive from C:. WSL isn't able to force Ubuntu to be installed on C, and thus gives this cryptic error.

To fix this, I went to `Settings > System > Advanced storage settings > Where new content is saved`. I changed the `New apps will save to:` option to be your main drive:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707990696241/4d0b4cd5-dfbb-42a3-a1ba-80c9d3615ff1.png align="center")

Once this has been done, I uninstalled `Ubuntu` again via the Windows start menu, and then ran `wsl --install -d Ubuntu` which ran successfully.

Lastly, I set Ubuntu as my default distro by running `wsl --setdefault Ubuntu` .

Finally, I was able to use Ubuntu via WSL by running `wsl` on the command line.

I hope this helps anyone else who is struggling with the same issue!