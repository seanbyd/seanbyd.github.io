# How to install Windows Subsystem for Linux (WSL)

This guide shows you how to install Windows Subsystem for Linux (WSL).

This is required when using podman with OpenShift Local.

For additional details refer to the [What is the Windows Subsysten for Linux](https://learn.microsoft.com/en-us/windows/wsl/about) article.

To install WSL you can visit the Microsoft store and search for WSL. Select the Linux distribution you want and follow the instructions.

Alternatively, open a command windows and enter

````cmd
wsl --list --online
````

    C:\Users\sean>wsl --list --online
    The following is a list of valid distributions that can be installed.
    The default distribution is denoted by '*'.
    Install using 'wsl --install -d <Distro>'.

    NAME                            FRIENDLY NAME
    * Ubuntu                          Ubuntu
    Debian                          Debian GNU/Linux
    kali-linux                      Kali Linux Rolling
    Ubuntu-18.04                    Ubuntu 18.04 LTS
    Ubuntu-20.04                    Ubuntu 20.04 LTS
    Ubuntu-22.04                    Ubuntu 22.04 LTS
    Ubuntu-24.04                    Ubuntu 24.04 LTS
    OracleLinux_7_9                 Oracle Linux 7.9
    OracleLinux_8_7                 Oracle Linux 8.7
    OracleLinux_9_1                 Oracle Linux 9.1
    openSUSE-Leap-15.6              openSUSE Leap 15.6
    SUSE-Linux-Enterprise-15-SP5    SUSE Linux Enterprise 15 SP5
    SUSE-Linux-Enterprise-15-SP6    SUSE Linux Enterprise 15 SP6
    openSUSE-Tumbleweed             openSUSE Tumbleweed

Refer to [Basic commands for WSL](https://docs.microsoft.com/en-us/windows/wsl/basic-commands) for more details on the WSL options.

## Install a specific Linux distribution

I have chosen to install Ubuntu. This happens to be the default so I have not specified any additional options.

Open a command window as Administrator and enter:

```
wsl --install
```

Follow the prompts.

The command output is as follows.

    C:\Windows\System32>wsl --install
    Installing: Windows Subsystem for Linux
    Windows Subsystem for Linux has been installed.
    Installing: Ubuntu
    Ubuntu has been installed.
    The requested operation is successful. Changes will not be effective until the system is rebooted.

Reboot your PC.

Check the version installed distribution including the WSL version. In my case I have installed Ubuntu using WSL 2.

````
wsl --list --verbose
````

    C:\Users\sean>wsl --list --verbose
    NAME      STATE           VERSION
    * Ubuntu    Stopped         2

Enter the following command to start WSL

```cmd
wsl
```

Open a second command window and enter the following to see Ubuntu is runnning.

````cmd
wsl -l -v
````

You will see Ubuntu running.

    C:\Users\sean>wsl -l -v
    NAME      STATE           VERSION
    * Ubuntu    Running         2

Exit from WSL.

## Re-install Ubuntu to to reset my user

As I missed entering a non-root user and password, I unregistered Ubuntu.

I'm sure there are other more efficient ways of doing this, I chose this one.

````cmd
wsl --unregister Ubuntu
````

    C:\Users\sean>wsl --unregister Ubuntu
    Unregistering.
    The operation completed successfully.

Install Ubuntu again. When prompted, enter your non-root user and password.

````cmd
wsl --install Ubuntu
````

    C:\Users\sean>wsl --install Ubuntu
    Ubuntu is already installed.
    Launching Ubuntu...
    Installing, this may take a few minutes...
    Please create a default UNIX user account. The username does not need to match your Windows username.
    For more information visit: https://aka.ms/wslusers
    Enter new UNIX username: sean
    New password:
    Retype new password:
    passwd: password updated successfully
    The operation completed successfully.
    Installation successful!
    To run a command as administrator (user "root"), use "sudo <command>".
    See "man sudo_root" for details.

    Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.153.1-microsoft-standard-WSL2 x86_64)

    * Documentation:  https://help.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/advantage


    This message is shown once a day. To disable it please create the
    /home/sean/.hushlogin file.
    sean@MyLaptop:~$


