# Introduction

We still need to have a Rosetta environment for the legacy Python development where some dependency of your project doesn't have Apple Silicon ports.
This has required a complicated setup where different packages support different architectures (arm64 and x86_64) coexist in one laptop.

but NOT many people have been successful because Rosetta environments were messing up the native brew installation with the x86 packages. 
e need to find a way to isolate the legacy environment from the native environment, thus to better support new architecture, but also to keep on supporting our legacy efficiently.

For this purpose, many directions have been investigated including running a Linux VM on top of the Docker Desktop Rosetta Emulation support. 
Such directions based on emulation have not been successful for some cases because not all of the HW instructions that the Compiled Linux distributions rely on are supported by the underlying emulation layer. 
To make it work, we need to do lots of investigations. 

Thus, in this doc, We’d rather focus on the easiest and most proven way to provide you with the Rosetta experience while still achieving the goal of isolating such an environment 
from the entire Native setup of your M1/M2 machine.

# Preparing your Rosetta Account and Terminal

## Account Preparation

As the easiest isolation method of your Rosetta environment, let’s create another account as a “Standard” (Non-admin) account 
just to prevent the activity from the account from messing up the native installations under non-user directories. 

<img src="https://github.com/bjlee72/devops/assets/4746751/edb69e11-95cc-4882-b3fb-78a669c1437a" width=400 />
<img src="https://github.com/bjlee72/devops/assets/4746751/a132761f-3d8e-4b8d-a1b7-76840ee222c3" width=400 />


You can do this from your “Users & Groups” of “System Settings”.

## Terminal

After the account preparation, “Duplicate” the iTerm app in your “Applications” folder and Name it “Rosetta iTerm”. Once you have done it, you might want to [change its icon](https://discussions.apple.com/thread/255174964?sortBy=best) just like what I did, to reduce confusion.

<img src="https://github.com/bjlee72/devops/assets/4746751/942a7194-3824-4200-b410-b2973bfa2f91" width=300 />

<img src="https://github.com/bjlee72/devops/assets/4746751/62d7ac4f-b62a-44b1-b17f-b3ed2379c305" width=300 />

<img src="https://github.com/bjlee72/devops/assets/4746751/24469bee-b12e-4818-bec5-a8553e8d4ba4" width=300 />

After we do this, we need to change the App’s info just like this (Click “Get Info” and choose “Open using Rosetta”).

<img src="https://github.com/bjlee72/devops/assets/4746751/1b2f23fd-f638-4fb2-a143-bb5dab49e3f7" width=450 />

<img src="https://github.com/bjlee72/devops/assets/4746751/7ad5913d-0e4a-41b5-9980-ee6fec3bfa9c" width=450 />

And we also create another iTerm profile for it to have a very distinguishing look. In my case, I created a new “Rosetta” profile.

![image](https://github.com/bjlee72/devops/assets/4746751/936a0a06-3024-4337-961f-c1be1c6873b6)

Please change its “General” setting for it to immediately do `su - rosetta` when we open a new terminal window forcing us to sign in to the isolated account. 
This will help prevent mistakes like not signing into the right account.

![image](https://github.com/bjlee72/devops/assets/4746751/acef94b5-2f11-4ddc-a060-501f3c82aade)

And please set the “Rosetta” profile as the “**Default**” profile. Then when you click the icon, you will immediately see the bottom left image. 
Please sign in with the account password of Rosetta. Also, please make sure your terminal is properly set up by running the `arch` command and seeing `i386` as an outcome.

<img src="https://github.com/bjlee72/devops/assets/4746751/3f12981c-c1cd-4e1a-a6c1-95452e24dffa" width=450 />

<img src="https://github.com/bjlee72/devops/assets/4746751/143b4e1c-44a3-4085-b57e-e3d640eaf6fe" width=450 />

# Installations

Now we have a completely isolated account and terminal from your native setup. Because this terminal is running on top of Rosetta, 
all the commands from the terminal run on top of the emulated x86_64 architecture. 

## Brew installation

Because this is a non-admin account, we need to do a “user” installation. That is, all the brew packages will be installed on your new account directory under `/Users/rosetta`.

```bash
git clone https://github.com/Homebrew/brew homebrew
eval "$(homebrew/bin/brew shellenv)"
brew update --force --quiet
chmod -R go-w "$(brew --prefix)/share/ssh"
```

And please make the following changes in your `.zshrc` file.

```bash
eval "$(~/homebrew/bin/brew shellenv)"
```

After making this change, probably you need to do `. ~/.zshrc` from your terminal.

## A few important package installations

### pyenv

Let’s set up the python related packages first because this will be required for the `gcloud` set up which will be explained later.

```bash
brew install openssl readline sqlite3 xz zlib # Some essential packages to build & install Python
curl https://pyenv.run | bash
```

Load `pyenv` automatically by adding the following to your shell configuration, such as `~/.zshrc`, `~/.bashrc`, or `~/.bash_profile` :

```bash
# ADD PYENV
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

Once this is properly done, you need to install lower and legacy python version that you want to install with Rosetta support, such as `3.7.13`.

```bash
pyenv install 3.7.13
pyenv install 3.10.13 # if you need to have higher version of the python as well for the tools such as gcloud
pyenv global 3.10.13
```

### wget, gh, and gcloud

Let’s install additional packages that you might need: wget, gh (GitHub CLI), and Google Cloud SDI (gcloud). 
gcloud is optional (it was needed for my own purpose).

```bash
brew install wget gh
```

The above will install `wget` and `gh`. It will takes time and it’s normal because sometimes they need to compile and build the packages from your laptop. 
(Typically by doing `./configure` `make` and `make test`). 

Once `wget` is installed, let’s download Google Cloud SDK (including gcloud). Please do this at your home directory  `/Users/rosetta`. 

```bash
wget https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-462.0.0-darwin-x86_64.tar.gz
tar zxvf google-cloud-cli-462.0.0-darwin-x86_64.tar.gz
```

And you need to include the following 2 lines in your `.zshrc`. 

```bash
export GCLOUD_ROOT="$HOME/google-cloud-sdk"
export PATH="$GCLOUD_ROOT/bin:$PATH"
```

After the change, let’s do `. ~/.zshrc`. (If you think this repetitive job is annoying, please add `alias so='. ~/.zshrc'` in your `.zshrc` file and use it in the future.

