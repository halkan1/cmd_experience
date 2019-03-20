# cmd_experience
*Description on getting a better cmd experience on Windows with cmder and Windows Subsystem for Linux*


## Introduction
This guide will proceed to show how one can setup Windows Subsystem for Linux (hereby referred to as WSL) and the console emulator cmder to enhance the command-line experience on Windows

We will use the following:
* WSL
* [oh my zsh](https://github.com/robbyrussell/oh-my-zsh)
* [Powerline](https://github.com/powerline/powerline)
* [ChromaTerm--](https://github.com/hSaria/ChromaTerm--)
* [cmder](https://github.com/cmderdev/cmder)

## WSL installation
Follow the guide [here](https://docs.microsoft.com/en-us/windows/wsl/install-win10) to setup WSL. I chose Ubuntu as I prefer that so this guide will be based on that.
After WSL is installed dont forget to run:
```bash
sudo apt-get update
sudo apt-get upgrade
```

## Install zsh
Now that we have a working WSL installation, lets install zsh.
```bash
sudo apt-get install zsh
```

After installing zsh type `zsh` and you will be asked to choose some configuration. This will be done via oh my zsh during installation so choose option `0` for know to create the config file and prevent this message form showing up again.

## Install oh my zsh
Git should be installed by default but you can verify that by typing `git --version`. If you get a reply you do not need to install git.
Should you need to install git though do so by issuing:
```bash
sudo apt-get install git
```

Then use `curl` to install oh my zsh:
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

This install script will clone the repo and reaplace the existing `~/.zshrc` with a template from `oh my zsh`.

## Configuring zsh/oh my zsh
First, we need to make sure `zsh` is executed by default for Bash on Ubuntu. This is not mandatory, but if not done you need to type `zsh` every time. For this, edit the .bashrc file with nano: `nano ~/.bashrc` and paste this right after the first comments:
```bash
if test -t 1; then
    exec zsh
fi
```
Save it `Ctrl + shift X` and restart your Ubuntu shell. You should be on zsh by default now

## Changing the Theme of oh-my-zsh
oh-my-zsh has several nice Themes. It's worth checking them out. For this tutorial, I'm going to use the awesome [agnoster](https://github.com/agnoster/agnoster-zsh-theme).

Edit the `~/.zshrc` again with nano: `nano ~/.zshrc`:
```bash
# Find and change this
ZSH_THEME="robbyrussell"

# To this
ZSH_THEME="agnoster"
```
Save it and restart your Ubuntu shell again.

## Installing missing Powerline Fonts
We need to install the Powerline fonts in our Windows to make the agnoster theme work. Follow these steps:
1. Clone the powerline repository on Windows
    ```bash
    git clone https://github.com/powerline/fonts.git
    ```
1. Open an admin PowerShell, navigate to the root of the repo and run this:
    ```powershell
    .\install.ps1
    ```
This will install all the fonts on your Windows. You might get an error from PowerShell blocking you from running the script. You will then need to open a powershell prompt as admin and enter:
```powershell
Get-ExecutionPolicy
```
Take note of the return output so that you can set it back once you have run the install script for the fonts. Then enter the following to change the execution policy:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
```

## Changing directory colors
The directory colors for zsh is awful. If you followed along, by now you should have an ugly yellow or dark blue background on folders when `ls/ll`. Luckily, we can change that by installing a Solarized Color Theme from [here](https://github.com/seebi/dircolors-solarized). Follow these steps:
1. Pick a theme from the GitHub repo (I'm using dircolors.ansi-dark since I use a dark shell).
1. Download the file making sure to put it in the user's home:
    ```bash
    # using dircolors.ansi-dark
    curl https://raw.githubusercontent.com/seebi/dircolors-solarized/master/dircolors.ansi-dark --output ~/.dircolors
    ```
1. Edit your `~/.zshrc` and paste this:
    ```bash
    # set colors for LS_COLORS
    eval `dircolors ~/.dircolors`
    ```

## Setting Bash on Ubuntu task in ConEmu
Open ConEmu, and go to `Settings`. Navigate on the left-menu: `Startup > Tasks`. There, click at the `+` button at the bottom.
1. Add a name for the task. Anything will suffice. I used `bash::ubuntu` to group Ubuntu into the bash tasks.
1. On `Task parameters` choose an icon for the task. I picked the Ubuntu icon app that is buried under some very long path. but any .ico will work. You can leave it blank if you don't care.
1. For the `command` use this `%windir%\system32\bash.exe ~ -cur_console:p`. This will start bash under the user home directory. Since we already configured `zsh` to run by default, this is enough.

## Installing ChromaTerm--
I wanted to use ChromaTerm to get highlighting when using ssh to access my Cisco devices at work. You will need some dependencies in order to compile the source code. Run the following command to install them if they are needed:
```bash
sudo apt install build-essential gcc pcre2-utils libpcre2-dbg libpcre2-dev
```
When the dependencies are installed proceed with the following to install and setup ChromaTerm--:
```bash
git clone https://github.com/hSaria/ChromaTerm--.git
cd chromaterm/src/
./configure
make
make install
```
In my case the compiled binary `ct` was, after creation in `/usr/local/bin`, owned by root so I changed that with:
```bash
sudo chown 'user':'group' /usr/local/bin/ct
```

You should now be able to verify that ChromaTerm is working by issuing:
```bash
echo "Jul 14 12:28:19: Message from 1.2.3.4" | ct
```
The out put should be colored.

## Seting up Cisco SSH Highlighting
A ChromaTerm config file will be created in users home folder `~` but I used the one provided by user ´vista_df´ (found in the following [Reddit thread](https://www.reddit.com/r/networking/comments/89e7ms/cisco_syntaxkeyword_highlighting_on_linux/)). And also here is the [direct link](https://gist.github.com/vista-/88c90110dd320be4c78da4f55783b41a).
I have also added that file to this repo to ensure that it doesn't disappear.

Rename the original `.chromatermrc` and copy the on in this repo to your home folder by issuing the following when in you homefolder (`cd ~`):
```bash
mv .chromatermrc chromatermrc.bak && wget https://raw.githubusercontent.com/halkan1/cmd_experience/master/assets/.chromatermrc
```

### Creating an function for ChromaTerm
Now you could use ChromaTerm like this:
```bash
ssh 8.8.8.8 | ct
```

And it will work but it would be nicer to create an alias for it. To do so add the following to your .zshenv file:
```bash
function ctssh {
    ssh $@ | ct 
}
```
If you, like me, did not have a .zshenv file simply create it in you users home directory.
```bash
touch ~/.zshenv
```
With this function you will be able to use `ssh` with `ct` by issuing `ctssh 8.8.8.8` which is a bit easier.
