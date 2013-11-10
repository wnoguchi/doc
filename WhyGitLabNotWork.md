# GitLabが動かない原因

```
Bash completion has been installed to:
  /usr/local/etc/bash_completion.d

The OS X keychain credential helper has been installed to:
  /usr/local/bin/git-credential-osxkeychain

The 'contrib' directory has been installed to:
  /usr/local/share/git-core/contrib
Error: The linking step did not complete successfully
The formula built, but is not symlinked into /usr/local
You can try again using `brew link git'
==> Summary
/usr/local/Cellar/git/1.7.11.3: 1204 files, 23M, built in 34 seconds
```

```
wnoguchi_pg1x_com_1354987714

config core.sharedRepository = 0660
```

↑原因はこいつだった。なんでこんなのくっついてくるの？？

```
770
```

add shared repository support by SaitoWu · Pull Request #1719 · gitlabhq/gitlabhq
https://github.com/gitlabhq/gitlabhq/pull/1719

これですべてが解決した。

Trouble Shooting Guide · gitlabhq/gitlab-public-wiki Wiki
https://github.com/gitlabhq/gitlab-public-wiki/wiki/Trouble-Shooting-Guide
