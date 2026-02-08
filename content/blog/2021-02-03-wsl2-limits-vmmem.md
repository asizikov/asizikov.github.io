---
date: "2021-02-03T00:00:00Z"
title: Configure WSL2 limits on Windows 10
tags: 
 - devops
slug: wsl2-limits-vmmem 
---

WSL2 on Windows 10 has some issues, but I still prefer it to run my docker containers locally.

One of the problems which used to bug me a lot was the memory consumption by `Vmmem` process.

![](/images/2021-02-vmmem/vmmem.png)

The memory consumption goes though the roof. Even on 32GB machine you may run out of memory.

I can't fix the problem, but at least I know how to patch it up. Here is what I do:

Stop the WSL first.

```bash
 wsl --shutdown
```

then create a `.wslconfig` file into the root directory of your users folder: `C:\Users\<yourUserName>\.wslconfig`

My `.wslconfig` looks this way:

{{< gist asizikov a6ff67d8ce4b18acf77e14ca32771a32 >}}

More details on what options to configure can be found [in documentation](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#configure-global-options-with-wslconfig).

After WSL and Docker Engine restart things look much better now.

![](/images/2021-02-vmmem/fixed.png)