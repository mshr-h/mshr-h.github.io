---
title: "2021 03 13 Config Gpg for Github on M1 Mac"
slug: "config-gpg-for-github-on-m1-mac"
subtitle:    ""
description: ""
date:        "2021-03-13"
author:      "Masahiro Hiramori"
image:       ""
tags:        []
categories:  ["Tech"]
draft:       true
---

```
brew install gpg
LANG=C gpg --gen-key
gpg --full-generate-key
```

```
$ gpg -k
/Users/username/.gnupg/pubring.kbx
--------------------------------
pub   rsa4096 2021-01-01 [SC]
      ABCDEFGHIJKL12345
uid           [ultimate] AAAA BBBB <username@gmail.com>
sub   rsa4096 2021-01-01 [E]
```

```
gpg -a --export ABCDEFGHIJKL12345 > pubkey.gpg
git config --global gpg.program /opt/homebrew/bin/gpg
git config --global user.signingkey "ABCDEFGHIJKL12345"
git config --global commit.gpgsign true


brew install pinentry-mac
echo "pinentry-program /opt/homebrew/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf
killall gpg-agent

echo "test" | gpg --clearsign
```
