# WSO Deployment Onboarding

## About this Repository

A Williams student was working on an Ansible role that would start up a service needed for the dev server, but never got around to setting it up right. Your task is to complete their code and extend their role to do a few new things. Follow the instructions to complete the project.

## Setting Up Your Dev Environment
You will need:
- A code editor (such as [VSCode](https://code.visualstudio.com) or [Emacs](https://www.gnu.org/software/emacs/))
- Git
- Ansible (which in turn requires Python)
- A VM manager (such as [VirtualBox](https://www.virtualbox.org) on Windows and Linux or [UTM](https://getutm.app) on macOS)

Click the links above to download the software components you don't already have. For Ansible, Python, and Git, you should install [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) on Windows, [Homebrew](https://brew.sh) on macOS, or use your distro's package manager on Linux.

Now you are ready to roll!

## Clone & Test
To clone the code you'll need, navigate into a directory of your choice and then run:
```console
git clone https://github.com/
```

The next step is to install the VMs. This is the longest step, but should be quick with a fast internet connection. Grab a [copy of AlmaLinux 10](https://almalinux.org/get-almalinux/) for your architecture and proceed with VM installation. Make a new virtual machine with at minimum, the following specs:
- One core or more
- 4028 MB of RAM or more
- At least 32GB of space
Most VM software will select these, but you might need to do it manually. 

If you aren't prompted for the values above, you will be able to change it after you finish the setup wizard. Customize them for the hardware limits of your machine, using half of what your machine's limits are. So if your machine has 8GB of RAM, your VM should use no more than 4GB, for example. However, VM disk size should never be set larger than 32GB, as that is wasteful.

You will need to edit the file `inventory/hosts.ini` with the IP address of your VM, as without it, Ansible won't be able to run. To find your VM's IP address, login as your user, and type `ip a`, then copy down the IP address for your interface. Make sure to edit your SSH config at `/etc/ssh/sshd_config` and change the value `PermitRootLogin` to `yes`, as that's what you'll be logging in as.

To start running your Ansible code, execute:
```
make run
```
Be ready to type the password for the `root` user. It will quit automatically when it's done running, or crash and burn if your code is buggy or you set up something wrong.

## Repository Structure
To prepare you to write code that runs on production, this one mirrors the layout of the actual [wso-config](https://github.com/WilliamsStudentsOnline/wso-config) as closely as possible:
``` shell
roles/ # where you'll be writing 99% of your Ansible code
inventory/
	hosts.ini # defines machine logins and ip addresses, replace this for your setup
site.yml # the main playbook, don't touch this
```
## Tasks
### Part 1
In your CS education at Williams, you are destined to use the amazing text editor known as Emacs. Unfortunately, Emacs is slow to start up, owing to the fact that it bundles a whole Lisp interpreter. To solve this, we can write an Ansible role that will start up `emacs-daemon` for all users. 

We could write this:
```yaml
- name: Start the Emacs server
  ansible.builtin.systemd_service:
    name: emacs-daemon
    enable: true
```
but that won't do, because Emacs sadly doesn't ship with a service file for daemon mode, and it won't run in the right scope (this is a user service). So we need to write our own service file, and deploy it. 

You'll first need to install Emacs, so log in to the VM, and run `dnf install emacs`.

Make a new role that copies the file `emacs-daemon.service` from your computer to the VM's `systemd` directory (`/etc/systemd/system/`). HINT: you'll need to edit the file in `roles/onboarding/files` and fix it first. Bonus points if you can do it in less than 3 lines (but not at all required, requires some shell wizardry).

Do you have the [Ansible documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_service_module.html) for this open yet? Ansible's documentation will be your new best friend, and you'll read it a lot.

### Part 2
Now we've got the daemon file on the server. But if we try and run it, `systemd` will complain that it doesn't recognize it. This is bad. Let's do the following things now in our role:
- Run the command `systemd daemon-reload`, and
- Ensure the service `emacs-daemon` is running.

There are many ways of solving a problem with Ansible. Between multiple solutions, you should favor the idempotent one. Not sure what that means? Basically, if you re-run `make run`, your code shouldn't break and it should work the first time as well, before you started making changes to state. Writing safe code that works on production requires getting the concept of idempotent code.

### Part 3
At this point, you've got a nice Emacs setup running. But perhaps our users might want some dev tools installed as well? Use your Ansible knowledge to ensure that the packages `ripgrep`, `go`, and `gcc` are installed on the VM. 

You may wish to read the documentation for `ansible.builtin.dnf` and the `loop:` macro. You must write this with the `loop:` macro, as it's used a lot within our codebase. 

### Part 4
Now that we have a nice dev environment going, we want to add a fun utility and spruce up the place a bit. Add the package `lolcat` and `cowsay` (you may need to run the command `dnf install epel-release` if it's not found).

There's a site online called [ZenQuotes](https://zenquotes.io) that lets you get a random sentence from a famous dead person. You can read about their API [here](https://docs.zenquotes.io/zenquotes-documentation/). I recommend using the program `jq` for parsing JSON.

Now, let's do a little trickery. Make an Ansible command that pipes a random [Albert Einstein](https://zenquotes.io/authors/albert-einstein) quote into `cowsay` into `lolcat`, and then pipes that into `/etc/motd` with a `cron` job. Solve this any way you like. You probably want to use the command `ansible.builtins.shell`, but you don't *have* to. Ansible is infinitely flexible, as this task will teach you. But a good solution is a readable one. Just because you could solve this in ~10 lines or less doesn't mean that you *should*.

### The End!
You are now ready to write real code! Remember:
- Documentation for Ansible is your friend
- Don't be afraid to fall back to shell commands for tasks, but be careful
- Splitting things up "one action per command in a role" makes life easier than having one monster long shell command
