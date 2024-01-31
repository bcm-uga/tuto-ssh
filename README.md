## How to work on krakenator/luxor from home?

1. Create an ssh key on your home computer:
    run `ssh-keygen`
    
    If you use rstudio you can also do this in the rstudio GUI: Tools -> Global Options -> Git/SVN

    In any case **use a strong passphrase**! This ssh key will provide an entry point into the TIMC network, the passphrase is what prevents any compromise of your home computer from spreading into the TIMC network.

    This will create a pair of files in `~/.ssh/` , the private key (`id_rsa` with the current defaults) and the associated public key (`id_rsa.pub`). Copy the public key on a flash drive or email it to youself so you have access to it from the lab (**NEVER copy the private key anywhere**).

2. Authorize your new ssh key on the TIMC servers. From within the lab you can connect to dolto.imag.fr using your TIMC password, while from outside you must connect to lacan.imag.fr which only allows key authentication, but both servers share your homedir. So, you must connect from your lab computer to dolto and add your new public key to `~/.ssh/authorized_keys`. Two ways to do this:
    1. From your lab computer, do `ssh-copy-id -f -i [myNewPublicKey.pub] dolto.imag.fr`
    2. Same thing by hand: from your lab computer do `ssh dolto.imag.fr` and in file `~/.ssh/authorized_keys`, copy the complete public key you created (one key per line). In this case make sure the permissions are restrictive on `~/.ssh/` (must be 700) and on  `~/.ssh/authorized_keys` (must be 600).

Check: now, you should be able to `ssh lacan.imag.fr` from your home computer (or if your home computer username is different from your TIMC username, you will need to `ssh TIMCusername@lacan.imag.fr` or `ssh -l TIMCusername lacan.imag.fr`). You can then `ssh krakenator.imag.fr` from within that ssh session to connect from lacan to krakenator (using your krakenator password).

Note: to disconnect from an ssh connection, you can use `Ctrl/Cmd + D`


## How to make the bounce through lacan.imag.fr transparent?

On your home computer, create file `~/.ssh/config` if it doesn't exist (restrictive permissions 600 again), and add the following lines (replacing `nthierry` (twice) by *your* username on the TIMC systems)
```
Host krakenator
User nthierry
ProxyCommand ssh -W krakenator.imag.fr:%p nthierry@lacan.imag.fr 
```

On very old systems you may have to replace the `ProxyCommand` line with
```
ProxyCommand ssh nthierry@lacan.imag.fr "nc krakenator.imag.fr %p"
```
       
You can now run `ssh krakenator` to connect to krakenator, transparently going through an ssh connection via lacan.imag.fr .
If it doesn't work check the permissions on ~/.ssh and it's content, they must be restrictive as stated above.
    

## How to prevent ssh disconnects?

You may see your ssh sessions closing for no apparent reason after some period of inactivity. It's not ssh's fault, ssh is rock-solid in my experience: this is due to something (ISP?) timing out your connection between your home computer and luxor/krakenator.
To solve this, edit `~/.ssh/config` on your home computer and add the following lines close to the top:
```
# have ssh send no-op codes to avoid disconnections
Host *
ServerAliveInterval 600
```
Depending on your ISP you may have to lower the interval value to 300 or even 60.

    
## How to easily access/edit your files on luxor?

All this happens on your home computer.
1. Install sshfs, for example on RHEL / ALMA linux / Fedora systems run `sudo dnf install fuse-sshfs`.
1. Create a mount point for luxor. We recommend creating a subdir where all your sshfs mountpoints will be created, so for example: `mkdir -p ~/sshMounts/luxor`.
1. Add a line in `/etc/fstab` (replacing `nthierry` (twice) by *your* username):
    ```
    nthierry@luxor:/    /home/nthierry/sshMounts/luxor    fuse.sshfs    noauto,users,idmap=user    0 0
    ```

You can now run `mount ~/sshMounts/luxor` on your home computer: this mounts your luxor homedir locally in `~/sshMounts/luxor/` via an ssh tunnel (and transparently via lacan thanks to the previous steps). You can then view or edit your files on luxor with whatever fancy GUI text editor you prefer, running on your home computer (please don't run GUI apps on luxor). When you are done simply `umount ~/sshMounts/luxor`.

Of course the same process can be used to access files on any other ssh-accessible system, e.g. krakenator or your lab desktop computer; just create a new mountpoint and add the relevant line to `/etc/fstab`.

NOTE: the above fstab line mounts the luxor rootdir (/), therefore you have access to your homedir but also to /data/ on luxor (where most of your stuff should be).
