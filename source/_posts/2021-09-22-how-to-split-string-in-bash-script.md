---
title: How to split string in bash script
cover: /images/fsqs-tips-tricks-notes.png
date: 2021-09-22T10:08:22.921Z
categories:
  - Bash
tags:
  - split-sting-bash-script
---

How to split string in bash script?

<!-- more -->

## Split string in bash script

For example, we want to extract aws region from docker repository url `225658775244.dkr.ecr.us-east-1.amazonaws.com`.

```bash
  docker_url=123232323.dkr.ecr.us-east-1.amazonaws.com
  IFS='.'
  #Read the split words into an array based on comma delimiter
  read -a strarr <<<"$docker_url"
  aws ecr get-login-password --region "${strarr[3]}" | docker login --username AWS --password-stdin "$docker_url"
```
