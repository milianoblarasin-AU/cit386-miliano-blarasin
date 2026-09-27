# Shared Server Working-Directory Session

## Result

I connected to the shared Ubuntu server with PuTTY's SSH client as `azureuser`. Before creating anything, I ran `pwd` and confirmed that the landing location was `/home/azureuser`. I then created my lowercase, hyphenated working directory at `/home/azureuser/miliano-blarasin`, entered it, created the required empty `hello.txt`, and used `ls -la` so the permissions, owner, group, size, and modification time were visible.

## Session transcript

The current public address, host-key fingerprint, and unrelated network addresses from the Ubuntu login banner are intentionally omitted from this public-repository transcript.

```text
Using username "azureuser".
Welcome to Ubuntu 24.04.4 LTS
[Ubuntu system banner omitted because it contained current network addresses]

azureuser@VM01:~$ pwd
/home/azureuser

azureuser@VM01:~$ mkdir -p miliano-blarasin
azureuser@VM01:~$ cd miliano-blarasin
azureuser@VM01:~/miliano-blarasin$ touch hello.txt
azureuser@VM01:~/miliano-blarasin$ ls -la
total 8
drwxrwxr-x 2 azureuser azureuser 4096 Sep 27 23:45 .
drwxr-x--- 9 azureuser azureuser 4096 Sep 27 23:45 ..
-rw-rw-r-- 1 azureuser azureuser    0 Sep 27 23:52 hello.txt
azureuser@VM01:~/miliano-blarasin$ exit
logout
```

## Verification

- `pwd` was run before `mkdir`, proving that the directory was created from the intended landing location.
- The second prompt shows that the shell moved into `~/miliano-blarasin` before the file was created.
- The final long listing shows `hello.txt` has a size of zero bytes, so it is empty.
- The owner and group columns both show `azureuser`, confirming that the created directory and file belong to the connected account.

