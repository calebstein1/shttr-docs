# Discover Shttr

## What is Shell on the Shttr?

Shell on the Shttr is a forward-looking web app framework built using mature, time and battle-tested technologies such as CGI and POSIX shell.
This rock stable foundation is then combined with new technologies like containerization to allow for rapid deployment and unparalleled maintainability.

## Installing Shttr

Shell on the Shttr is managed entirely by the Shttr CLI, which can be downloaded from the frontpage of [shttr.io](https://shttr.io).
You'll need to make sure you have certain dependencies installed on your computer prior to running Shttr CLI.
Specifically, you'll need Docker and NodeJS installed.
On MacOS, you'll also need GNU Find, which can be installed from MacPorts with `sudo port install findutils`.
If you wish to use the Shttr Static Site Generator with the `pd` module, you'll need Pandoc installed as well.
Once the dependencies are met, you'll download install.sh from [shttr.io](https://shttr.io), and run the following commands in your terminal:
```Shell
cd ~/Downloads
chmod +x install.sh
sudo ./install.sh
```
If you downloaded install.sh to a directory other than `~/Downloads`, or if you use a program other than `sudo` to gain elevated privileges, you'll have to adjust the commands accordingly.
It's a good idea to keep install.sh around because it can be used to update ShttrCLI, though you can always download it again to update if you delete it.
Now you can run `shttr` in your terminal.
The first run will download the latest Shell on the Shttr runtime, and then you're ready to go!