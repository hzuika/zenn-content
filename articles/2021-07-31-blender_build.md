---
title: "Blenderã®ãã«ã"
emoji: "ð¡"
type: "tech" # tech: æè¡è¨äº / idea: ã¢ã¤ãã¢
topics: []
published: false
---

## éçºãã¼ã«ã®ç¨æ
Windowsã®å ´åï¼[scoop]([lukesampson/scoop: A command-line installer for Windows.](https://github.com/lukesampson/scoop))ã§ã¤ã³ã¹ãã¼ã«ããã¨æ¥½ã§ãï¼
```
scoop install git cmake sliksvn
```

[Visual Studio](https://visualstudio.microsoft.com/)ã¯Microsoftã®Webãã¼ã¸ããã¤ã³ã¹ãã¼ã«ãã¾ãï¼
C++ã«ãããã¹ã¯ãããéçºãã¤ã³ã¹ãã¼ã«ãã¾ãï¼

## Blenderã®ãã«ã

```ps1
# é©å½ãªãã©ã«ããã¤ããã¾ãï¼
mkdir C:\blender-git
cd C:\blender-git
# ã½ã¼ã¹ã³ã¼ãããã¦ã³ã­ã¼ããã¾ãï¼
git clone git:://git.blender.org/blender.git
# ç§»åãã¾ãï¼
cd C:\blender-git\blender
# ã©ã¤ãã©ãªããã¦ã³ã­ã¼ããã¾ãï¼
make update
# ã¤ã³ã¹ãã¼ã«ãããå°ã­ãããã®ã§ï¼ y ãå¥åãã¾ãï¼
y
# ãã«ããã¾ãï¼
make
# Powershellã®å ´å make ã§ã¯ãªãï¼ .\make.batã¨ãã¾ãï¼
.\make.bat update
.\make.bat
```

