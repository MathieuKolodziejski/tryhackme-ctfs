# Room: W1seGuy

## General Info
- **Room link:** [W1seGuy](https://tryhackme.com/room/w1seguy)

## Description

Your friend told me you were wise, but I don't believe them. Can you prove me wrong?

## Provided Files / Links
- [x] Target Machine IP (10.10.150.252)
- [x] Python source code (listed below)

## Cryptography analysis
- Source code

We are given the following python script:

```
import random
import socketserver 
import socket, os
import string

flag = open('flag.txt','r').read().strip()

def send_message(server, message):
    enc = message.encode()
    server.send(enc)

def setup(server, key):
    flag = 'THM{thisisafakeflag}' 
    xored = ""

    for i in range(0,len(flag)):
        xored += chr(ord(flag[i]) ^ ord(key[i%len(key)]))

    hex_encoded = xored.encode().hex()
    return hex_encoded

def start(server):
    res = ''.join(random.choices(string.ascii_letters + string.digits, k=5))
    key = str(res)
    hex_encoded = setup(server, key)
    send_message(server, "This XOR encoded text has flag 1: " + hex_encoded + "\n")
    
    send_message(server,"What is the encryption key? ")
    key_answer = server.recv(4096).decode().strip()

    try:
        if key_answer == key:
            send_message(server, "Congrats! That is the correct key! Here is flag 2: " + flag + "\n")
            server.close()
        else:
            send_message(server, 'Close but no cigar' + "\n")
            server.close()
    except:
        send_message(server, "Something went wrong. Please try again. :)\n")
        server.close()

class RequestHandler(socketserver.BaseRequestHandler):
    def handle(self):
        start(self.request)

if __name__ == '__main__':
    socketserver.ThreadingTCPServer.allow_reuse_address = True
    server = socketserver.ThreadingTCPServer(('0.0.0.0', 1337), RequestHandler)
    server.serve_forever()
```

When we access the target machine on port 1337 with netcat (`nc 10.10.150.252 1337`), we are welcomed with the value of the encoded 1st flag:

```
This XOR encoded text has flag 1: 2d10000a324839211f363c203930360d6c2e1a2138363f422315143419170b2c3441370b2002033f
What is the encryption key?
```

Reading into the python source code, this value is generated in the setup function.

There are a few things that we can already take into consideration from the code and from the format of the flag that is expected:

- The key only consists of letters and digits
- The key consists of 5 characters
- The 4 first characters of the flag are "THM{"
- The last character of the flag is "}"

## Python Decryption Script

I created the following decryption script considering the restrictions on the key:

```
import binascii
import string

hex_encoded = "2d10000a324839211f363c203930360d6c2e1a2138363f422315143419170b2c3441370b2002033f"
xored_bytes = binascii.unhexlify(hex_encoded)
# The key only consists of letters and digits (A-Za-z0-9)
valid_key_chars = string.ascii_letters + string.digits  

# The key consists of 5 characters
partial_key = bytearray(5) 
# Recover first 4 bytes of key using "THM{" (The 4 first characters of the flag are "THM{")
for i in range(4):
    partial_key[i] = xored_bytes[i] ^ ord("THM{"[i])

# Find the single valid 5th character (The last character of the flag is "}")
for candidate_char in valid_key_chars:
    key = partial_key.copy()
    key[4] = ord(candidate_char)
    
    # Verify the last character produces a valid flag end
    last_char = xored_bytes[-1] ^ key[len(xored_bytes) % 5 - 1]
    if last_char == ord('}'):
        # Full decryption verification
        decrypted = []
        valid = True
        for i in range(len(xored_bytes)):
            decrypted_char = xored_bytes[i] ^ key[i % 5]
            decrypted.append(chr(decrypted_char))
            # Early exit if we hit an invalid character
            if i < 4 and decrypted_char != ord("THM{"[i]):
                valid = False
                break
        
        if valid and decrypted[-1] == '}':
            print(f"Found key: {bytes(key).decode()}")
            print(f"Decrypted flag: {''.join(decrypted)}")
            break
```

Running the script gives us the key: **yXMqB** and the 1st flag: **THM{p1alntExtAtt4ckcAnr3alLyhUrty0urxOr}**.

Inputing the key in the question `What is the encryption key?` gives us the 2nd flag: **THM{BrUt3_ForC1nG_XOR_cAn_B3_FuN_nO?}**.

## Final considerations

This room is an interesting quick room to test the cryptographic knowledge and it makes you consider all the parameters before starting to bruteforce.
