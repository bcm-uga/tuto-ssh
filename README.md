## How to work on krakenator/luxor from home?

1. Create an ssh key on your home computer:
    run `ssh-keygen`
    
    If you use rstudio you can also do this in the rstudio GUI: Tools -> Global Options -> Git/SVN

    In any case **use a strong passphrase**! This ssh key will provide an entry point into the TIMC network, the passphrase is what prevents any compromise of your home computer from spreading into the TIMC network.

    This will create a pair of files in `~/.ssh/` , the private key (`id_rsa` with the current defaults) and the associated public key (`id_rsa.pub`). Copy the public key on a flash drive or email it to youself so you have access to it from the lab (**NEVER copy the private key anywhere**).

2. Authorize your new ssh key on the TIMC servers. From within the lab you can connect to dolto.imag.fr using your TIMC password, while from outside you must connect to lacan.imag.fr which only allows key authentication, but both servers share your homedir. So, you must connect from your lab computer to dolto and add your new public key to `~/.ssh/authorized_keys`. Two ways to do this:
    1. From your lab computer, do `ssh-copy-id -i [myNewPublicKey.pub] dolto`
    2. Same thing by hand: from your lab computer do `ssh dolto` and in file `~/.ssh/authorized_keys`, copy the complete public key you created (one key per line). In this case make sure the permissions are restrictive on `~/.ssh/` (must be 700) and on  `~/.ssh/authorized_keys` (must be 600).

Check: now, you should be able to `ssh lacan.imag.fr` from your home computer. You can then `ssh krakenator` from within that ssh session to connect from lacan to krakenator (using your krakenator password).

Note: to disconnect from an ssh connection, you can use `Ctrl/Cmd + D`


## How to make the bounce through lacan.imag.fr transparent?

On your home computer, create file `~/.ssh/config` if it doesn't exist (restrictive permissions 600 again), and add the following lines (replacing `nthierry` (twice) by *your* username on the TIMC systems)
```
Host krakenator
User nthierry
ProxyCommand ssh -W krakenator:%p nthierry@lacan.imag.fr 
```

On very old systems you may have to replace the `ProxyCommand` line with
```
ProxyCommand ssh nthierry@lacan.imag.fr "nc krakenator %p"
```
       
You can now run `ssh krakenator` to connect to krakenator, transparently going through an ssh connection via lacan.imag.fr .
If it doesn't work check the permissions on ~/.ssh and it's content, they must be restrictive as stated above.
    
    
## How to easily access/edit your files on krakenator?

All this happens on your home computer.
1. Install sshfs, for example on Centos 6 and 7 systems run `sudo yum install fuse-sshfs`.
1. Create a mount point for your krakenator homedir. We recommend creating a subdir where all your sshfs mountpoints will be created, so for example: `mkdir -p ~/sshMounts/krakenator`.
1. Add a line in `/etc/fstab` (replacing `nthierry` (twice) by *your* username on the TIMC systems):
    ```
    nthierry@krakenator:    /home/nthierry/sshMounts/krakenator    fuse.sshfs    noauto,users    0 0
    ```

You can now run `mount ~/sshMounts/krakenator` on your home computer: this mounts your krakenator homedir locally in `~/sshMounts/krakenator/` via an ssh tunnel (and transparently via lacan thanks to the previous steps). You can then view or edit your files on krakenator with whatever fancy GUI text editor you prefer, running on your home computer (please don't run GUI apps on krakenator). When you are done simply use `umount ~/sshMounts/krakenator`.

Of course the same process can be used to access files on any other ssh-accessible system, e.g. luxor or your lab desktop computer; just create a new mountpoint and add the relevant line to `/etc/fstab`.

NOTE: the above fstab line mounts your homedir, so you cannot access anything outside of /home/YOURLOGIN/. You can mount the root filesystem (and therefore have access to /data/) by adding a slash to the end of the first field:
```
nthierry@luxor:/    /home/nthierry/sshMounts/luxor    fuse.sshfs    noauto,users    0 0
```


## How to connect to the RStudio servers on krakenator and luxor

Both krakenator and luxor offer an rstudio-like web interface thanks to rstudio-server. Each server listens on tcp port 8787, but this port is not exposed: rstudio-server only accepts connections locally. You therefore need to create an ssh tunnel from your computer to krakenator (or luxor), redirecting every request made on your local computer port 8787 to krakenator:8787 via this ssh tunnel:
1. Run `ssh -L  8787:localhost:8787 krakenator` (and keep it running)
1. In a browser, go to `http://localhost:8787/`

You can do this from your lab computer, but also from your home computer if you configured your ssh access as instructed in the previous steps.

To install R packages that require a recent version of *gcc* on krakenator, connect to krakenator via a terminal, run `scl enable devtoolset-7 bash`, then install the packages you need from this terminal. 
