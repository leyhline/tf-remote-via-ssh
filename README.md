# University of Freiburg, Faculty of Engineering, SSH Access
If you are a student at this institution you gain access to a pool of student PCs.
The majority of people just uses them in person (or never because they bring their own notebooks) but
that's such a waste since you can also gain remote access via `ssh`. With this you gain your very
own compute cluster of up to 63 machines! If you think that's worthwhile read further.

## General
To gain access you need to tunnel your connection through a gateway server running NetBSD.
(login.informatik.uni-freiburg.de)
Next, you can connect to one of the 63 PCs residing at the Faculty of Engineering named:
* tfpool01
* tfpool02
* …
* tfpool63

### Constraints
* The computer has to be turned on. If someone on site shuts the PC down you obviously get disconnected, too.
* The computer needs to be booted with Linux for the ssh server to run.
* As a student, you only get 1 GB disk space in your personal home directory which is shared across all computers.
    * You can request more if you can justify its necessity for a lecture or your thesis.
    * You can also use the local /tmp/ folder which is, as implied by the name, temporary.
* There is a nightly routine reboot at 3:10 AM.

### Specs
* 8-core Intel Core i7-4770 CPU @ 3.40GHz
* 8 GB RAM
* For tfpool01 to tfpool19: GeForce GT 630 2 GB
    * CUDA Capability Major/Minor version number:    3.0
    * ( 1) Multiprocessors, (192) CUDA Cores/MP:     192 CUDA Cores
* For tfpool20 to tfpool63: GeForce GTX 1060 3 GB
    * CUDA Capability Major/Minor version number:    6.1
    * (10) Multiprocessors, (128) CUDA Cores/MP:     1280 CUDA Cores
    * **Use these for GPU programming/ Deep Learning!**

## Remote connection using ssh
For a basic connection you just need to first connect to the gateway, next to one of the student computers. Replace <username> with your actual username. You will be queried for your password. (Oh, and of course you need to have ssh installed.)

```
ssh <username>@login.informatik.uni-freiburg.de
ssh tfpool42
```

### Check for available hosts
Since you won't know which PCs are turned on beforehand, you can use this handy script after you have connected with the gateway:
```bash
for i in `seq -w 1 63`; do
    (ssh -q -o ConnectTimeout=1 -o StrictHostKeyChecking=no tfpool${i} exit
    if [ $? -eq 0 ]; then
        echo tfpool${i}
    fi) &
done | sort
```

### Automatic key authentification
At the moment you need to re-type your password every time you make a connection. But fear not, you can generate a key for automatic authentification.
```bash
ssh-keygen # This will generate a new key pair.
ssh-copy-id <username>@login.informatik.uni-freiburg.de # Copy public key to gateway
ssh <username>@login.informatik.uni-freiburg.de # No password query

ssh-keygen           # This will do the same but for
ssh-copy-id tfpool42 # ALL student computers at once
ssh tfpool42         # since they share this key
```

### Running GUI applications with X11 forwarding
Fortunately, X11 is supported by the server. Now you can even stream graphical applications.

TODO

## Configuration
You need to edit `.ssh/config` (or create it if it doesn't exist yet).

TODO

## Applications

### Running a temporary webserver
This may be a bit risky securitywise but since the student computers can be reached via a public IP it's possible.
To try it out you first need to find out the ip address with `ip addr`. There will be multiple matches but I'm trusting your intuition.
```
ip addr | grep inet
```

Next, you can run a simple webserver e.g. [with Python](https://docs.python.org/3.7/library/http.server.html#http.server.SimpleHTTPRequestHandler):
```
python3 -m http.server 8000
```

Now, open your local browser, type in the IP address you found followed by :8000 for the port and enjoy!
There are lots of cool things you can do with this. The possibilities are endless!
For example, why not run a GPU accelerated version of [PaintsTransfer](://github.com/lllyasviel/style2paints)?