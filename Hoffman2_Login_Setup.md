
# SET up easy SSH

locally run:

```
ssh-keygen –t rsa
pbcopy < ~/.ssh/id_rsa.pub 
```

in hoffman:

```
vi ~/.ssh/authorized_keys

```
& paste in the rsa key that you copied using pbcopy


test locally:

```
ssh briscoel@hoffman2.idre.ucla.edu
```

should require no password to login


## SET UP alias for ssh

locally add this inside .ssh/config

```
Host hoffy
  User briscoel
  HostName hoffman2.idre.ucla.edu
  IdentityFile ~/.ssh/id_rsa

Host hoffy3
  User briscoel
  HostName login3.hoffman2.idre.ucla.edu
  IdentityFile ~/.ssh/id_rsa
```

Two aliases for Hoffman2

test, now try:

```
ssh hoffy
```

should get you into hoffman without password and long address
