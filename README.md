## How to work on luxor/mordor from home?

1. Create an ssh key on your "home computer" (eg your TIMC laptop):
    run `ssh-keygen`
   
    **Use a strong passphrase**! This ssh key will provide an entry point into the TIMC network, the passphrase is what prevents any compromise of your home computer from spreading into the TIMC network.
   
    This will create a pair of files in `~/.ssh/`: the private key (eg `id_ed25519`) and the associated public key (`id_ed25519.pub`). Copy the public key on a flash drive or email it to youself so you have access to it from the lab (**NEVER copy the private key anywhere**).

3. Authorize your new ssh key on the TIMC servers. From within the lab you can connect to interne-timc.u-ga.fr using your TIMC password, while from outside you must connect to ssh-timc.u-ga.fr which only allows key authentication, but both servers share your homedir. So, you must connect from your lab computer to interne-timc and add your new public key to `~/.ssh/authorized_keys`. Two ways to do this:
    1. From your lab computer, do `ssh-copy-id -f -i [myNewHomePublicKey.pub] interne-timc.u-ga.fr`
    2. Same thing by hand: from your lab computer do `ssh interne-timc.u-ga.fr` and in file `~/.ssh/authorized_keys`, copy the complete public key you created (one key per line). In this case make sure the permissions are restrictive on `~/.ssh/` (must be 700) and on  `~/.ssh/authorized_keys` (must be 600).

Check: now, you should be able to `ssh ssh-timc.u-ga.fr` from your home computer (or if your home computer username is different from your TIMC username, you will need to `ssh TIMCusername@ssh-timc.u-ga.fr` or `ssh -l TIMCusername ssh-timc.u-ga.fr`). You can then `ssh mordor.u-ga.fr` from within that ssh session to connect from ssh-timc to mordor (using your mordor password).

Note: to disconnect from an ssh connection: `Ctrl + D`


## How to make the bounce through ssh-timc.u-ga.fr transparent?

On your home computer, create file `~/.ssh/config` if it doesn't exist (restrictive permissions 600 again), and add the following lines
```
# if your username on your laptop/home computer is different from your TIMC/UGA username:
Host *.u-ga.fr *.imag.fr
User yourTIMCusername

# internal TIMC hosts
Host mordor mordor.u-ga.fr luxor luxor.u-ga.fr
ProxyCommand ssh -W %h:%p ssh-timc.u-ga.fr
```
       
You can now run `ssh mordor` to connect to mordor, transparently going through an ssh connection via ssh-timc.u-ga.fr .
If it doesn't work check the permissions on ~/.ssh and it's content, they must be restrictive as stated above.
    

## How to prevent ssh disconnects?

You may see your ssh sessions closing for no apparent reason after some period of inactivity. It's not ssh's fault, ssh is rock-solid in my experience: this is due to something (ISP?) timing out your connection between your home computer and luxor/mordor.
To solve this, edit `~/.ssh/config` on your home computer and add the following lines close to the top:
```
# have ssh send no-op codes to avoid disconnections
Host *
ServerAliveInterval 600
```
Depending on your ISP you may have to lower the interval value to 300 or even 60.

    
## How to easily access/edit your files on mordor & luxor?

All this happens on your home computer.
1. Install sshfs, for example on RHEL / ALMA linux / Fedora systems run `sudo dnf install fuse-sshfs`.
1. Create mount points for mordor and luxor. We recommend creating a subdir where all your sshfs mountpoints will be created, so for example: `mkdir -p ~/sshMounts/mordor ~/sshMounts/luxor`.
1. Add these lines in `/etc/fstab` as root (replacing `thierryn` by *your* username on your home computer):
    ```
    mordor:    /home/thierryn/sshMounts/mordor    fuse.sshfs    noauto,users,idmap=user    0 0
    luxor:/    /home/thierryn/sshMounts/luxor     fuse.sshfs    noauto,users,idmap=user    0 0
    ```

You can now run `mount ~/sshMounts/mordor` on your home computer: this mounts your mordor homedir locally in `~/sshMounts/mordor/` via an ssh tunnel (and transparently via ssh-timc.u-ga.Fr thanks to the previous steps). You can then view or edit your files on mordor with whatever fancy GUI text editor you prefer, running on your home computer (please don't run GUI apps on luxor). When you are done simply `umount ~/sshMounts/mordor`.

Of course the same process can be used to access files on any other ssh-accessible system, e.g. your lab desktop computer; just create a new mountpoint and add the relevant line to `/etc/fstab`.

NOTE: Often you just need to access your homedir (/home/yourTIMCusername/), but sometimes it can be useful to mount the rootdir (/) instead, eg on luxor where you will want to access /data/ (where most of your stuff should be). The above fstab lines illustrate this: on mordor they mount your homedir, but on luxor they mount the rootdir (notice the trailing slash in 'luxor:/'). 
