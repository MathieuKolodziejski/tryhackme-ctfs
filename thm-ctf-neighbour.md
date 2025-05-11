# Room: Neighbour

## General Info
- **Room link:** [Neighbour](https://tryhackme.com/room/neighbour)

## Description

Check out our new cloud service, Authentication Anywhere -- log in from anywhere you would like! Users can enter their username and password, for a totally secure login process! You definitely wouldn't be able to find any secrets that other people have in their profile, right?

## Provided Files / Links
- [x] Target Machine IP (10.10.228.154)

## Website analysis

I navigated to the website http://10.10.228.154/. It a login page with a suggestion to open the source code (`Don't have an account? Use the guest account! (Ctrl+U)`).

In the source code there is the following comment:

`<!-- use guest:guest credentials until registration is fixed. "admin" user account is off limits!!!!! -->`

When logging in with the guest account, I noticed that the query parameter `user` can be exploited in the URL:
`http://10.10.228.154/profile.php?user=guest`

Let's try to exploit an IDOR vulnerability with the admin user => `http://10.10.228.154/profile.php?user=admin`.

The flag is then revealed in the page:

`Hi, admin. Welcome to your site. The flag is: flag{66be95c478473d91a5358f2440c7af1f}`.
