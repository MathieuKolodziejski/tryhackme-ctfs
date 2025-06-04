# Room: Commited

## General Info
- **Room link:** [Commited](https://tryhackme.com/room/committed)

## Description

Oh no, not again! One of our developers accidentally committed some sensitive code to our GitHub repository. Well, at least, that is what they told us... the problem is, we don't remember what or where! Can you track down what we accidentally committed?

## Provided Files / Links
- [x] Target Machine IP (10.10.114.213)

## Commited zip

I first extrated the commited zip and noticed it was a git repository.

I found that there are 2 branches in the repository: master and dbint (`git branch -a`).

In the dbint branch (`git checkout dbint`), I found an `oops` commit (`git log`).

I then accessed the previous commit and found the flag in the password field in the main.py file: **flag{a489a9dbf8eb9d37c6e0cc1a92cda17b}**.

Bonus
---

Since I knew the flag pattern I was looking for, I also wanted to create a bash script to go through all the files of the different commits of the different branches to find the flag:

I created the following flag.sh script:

```
git rev-list --all | while read commit; do
  git ls-tree -r --name-only "$commit" | while read file; do
    git show "$commit:$file" 2>/dev/null | grep -Ei "[a-z0-9]{4}\{" && echo "Found in $file at $commit"
  done
done
```

```
    password="flag{a489a9dbf8eb9d37c6e0cc1a92cda17b}" # Password Goes Here
    password="flag{a489a9dbf8eb9d37c6e0cc1a92cda17b}", #password Goes here
    password="flag{a489a9dbf8eb9d37c6e0cc1a92cda17b}",
Found in main.py at 3a8cc16f919b8ac43651d68dceacbb28ebb9b625
```