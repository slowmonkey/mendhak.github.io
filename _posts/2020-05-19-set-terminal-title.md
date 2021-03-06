---
title: "How to set the title of a tab in terminal"
description: "From bash, setting the title of a tab in Ubuntu gnome-terminal or in Windows Terminal"
categories: 
  - wsl
  - ubuntu
  - bash
  - tweak

---

Both gnome-terminal in Ubuntu as well as Windows Terminal with bash allow you to set the title of the current tab you're working in.  This can be useful if you're in multiple shell sessions and need a visual cue to switch between them.  

Open up your `~/.bashrc` file, 

```
nano ~/.bashrc
```

And then add this function at the end:


```bash
function set-title() {
  if [[ -z "$ORIG" ]]; then
    ORIG=$PS1
  fi
  TITLE="\[\e]2;$*\a\]"
  PS1=${ORIG}${TITLE}
}
```

Then save and exit (`Ctrl X`), and reload the file with `source ~/.bashrc`. 

Now try setting the title. 

```
set-title Hello World!
```

![results]({{ site.baseurl }}/assets/images/set-terminal-title/001.png)

![results]({{ site.baseurl }}/assets/images/set-terminal-title/002.png)
