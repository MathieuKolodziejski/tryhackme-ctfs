# Room: Cyberheroes

## General Info
- **Room link:** [Cyberheroes](https://tryhackme.com/room/cyberheroes)

## Description

Want to be a part of the elite club of CyberHeroes? Prove your merit by finding a way to log in!

## Provided Files / Links
- [x] Target Machine IP (10.10.52.204)

## Website analysis

Given the room description, the vulnerability is about authentification bypass.

I went to the login page, put some credentials in the form and headed to the network tab. I noticed that no request was made, so it meant that the authentification happens in a static file (html of javascript).

I had a look at the HTML and found the following javascript code :

```
    function authenticate() {
      a = document.getElementById('uname')
      b = document.getElementById('pass')
      const RevereString = str => [...str].reverse().join('');
      if (a.value=="h3ck3rBoi" & b.value==RevereString("54321@terceSrepuS")) { 
        var xhttp = new XMLHttpRequest();
        xhttp.onreadystatechange = function() {
          if (this.readyState == 4 && this.status == 200) {
            document.getElementById("flag").innerHTML = this.responseText ;
            document.getElementById("todel").innerHTML = "";
            document.getElementById("rm").remove() ;
          }
        };
        xhttp.open("GET", "RandomLo0o0o0o0o0o0o0o0o0o0gpath12345_Flag_"+a.value+"_"+b.value+".txt", true);
        xhttp.send();
      }
      else {
        alert("Incorrect Password, try again.. you got this hacker !")
      }
    }
```

The username is "h3ck3rBoi" and the password is "SuperSecret@12345".

I then got the flag **flag{edb0be532c540b1a150c3a7e85d2466e}**.