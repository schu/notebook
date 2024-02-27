# Git post-receive hook for static site deployments

A Git [post-receive](https://git-scm.com/docs/githooks#post-receive) hook to
deploy a static site to a directory:

```
#!/bin/bash
set -euo pipefail

while read oldrev newrev ref; do
    if [[ $ref =~ .*/master$ ]]; then
        echo "Master ref received. Deploying master branch to production..."
        git --work-tree=/var/www/www.example.com --git-dir=/root/repos/example.com/ checkout -f
        git --work-tree=/var/www/www.example.com --git-dir=/root/repos/example.com/ clean -fd
    else
        echo "Ref $ref successfully received. Doing nothing: only the master branch may be deployed on this server."
    fi
done
```
