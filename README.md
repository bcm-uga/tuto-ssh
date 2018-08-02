## How to connect to krakenator from home?

1. Create a public ssh key on your personal computer (you can do this easily in Tools -> Global Options -> Git/SVM) of RStudio)

1. From your lab computer, do `ssh dolto`

    1. In *.ssh/authorized_keys*, copy the complete public key you created (one key per line)
    1. Check: now, you should be able to do `ssh lacan.imag.fr` from your personal computer.
    
