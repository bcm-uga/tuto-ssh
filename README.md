## How to connect to krakenator from home?

1. Create an ssh key on your personal computer 
    - just run `ssh-keygen`
    - if you prefer you can do this in Tools -> Global Options -> Git/SVN of RStudio
In any case <b>use a strong passphrase</b>! This ssh key will provide an entry point into the TIMC network, the passphrase is what prevents any compromise of your home computer to spread into the TIMC network.
This will create a pair of files in ~/.ssh/ , the private key (id_rsa with the current defaults) and the associated public key (same with .pub appended).

1. From your lab computer, do `ssh dolto`

    1. In file `~/.ssh/authorized_keys`, copy the complete public key you created (one key per line)
    1. Check: now, you should be able to do `ssh lacan.imag.fr` from your personal computer
    1. Note: to disconnect from an ssh connection, you can use `Ctrl/Cmd + D`

1. On your personal computer, in file `~/ssh/config`, add the following lines (replacing `nthierry` (twice) by *your* name)
    ```
    Host krakenator
    User nthierry
    ProxyCommand ssh nthierry@lacan.imag.fr "nc krakenator %p"
    ForwardX11 yes
    ```
       
    **You should be able to do `ssh krakenator`** (if you have a permissions problem, use `chmod 600 ~/ssh/config`)
    
1. If you want to connect to the RStudio server on krakenator

    1. Use `ssh -L  8787:localhost:8787 krakenator` (and keep it running)
    1. In a browser, go to `http://localhost:8787/`
