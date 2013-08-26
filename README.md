bbwait: Because ':wq' is the only `vi` command I will ever use again.
======

**TL;DR:** Use [BBEdit]'s command line tool (bbedit) as your $EDITOR in Terminal or iTerm.

## How It Works ##

There's nothing really all that complicated about it, the `bbedit` command does all the heavy lifting, `bbwait` just calls it with a few specific parameters:


		bbedit \
		--language 'Unix Shell Script' \
		--create-unix \
		--separate-windows \
		--resume \
		--wait \
		-- "$@"


* Since this will be called from the shell, it seems safe to assume that's what we should use for a file type. 

* --separate-window makes sure that the file gets its own window.

* --resume will bring Terminal/iTerm to the front when the BBEdit window closes *(note: this has nothing to do with Mac OS X's "resume" feature from 10.7 or later.)*

* --wait tells the script to wait for the file to be closed in BBEdit before continuing. **This is extremely important for visudo and using `bbwait` as your EDITOR and/or VISUAL command.**

## visudo ##

The easiest way to use `bbwait` with `visudo` is to add this line to your `/etc/sudoers` file:

		Defaults 	editor=/usr/local/bin/bbwait

You can do that by opening the file in BBEdit or by using `visudo` (the latter being recommended because it will do the syntax checking). 

Then, to edit the `sudoers` file with `bbwait`, go to Terminal/iTerm and enter: 

		sudo visudo

and you will see this prompt for your administrator password:

![BBEdit authorization prompt]

The file will still be 'locked' until you try to edit it, at which point BBEdit will prompt you to unlock the file, as shown here:

![BBEdit Unlock Prompt]

Then you can edit the file as usual. When you are done, save the file, close the window, and Terminal/iTerm should come forward and show you this if you made changes:

			bbwait: bbedit exited successfully.

If you do not see any message from `visudo` then it did not encounter any errors. If you open the file but do not make any changes, you will see this:

		bbwait: bbedit exited successfully.
		visudo: /private/etc/sudoers.tmp unchanged


## If you bought BBEdit from the Mac App Store…

Be sure to download and install the command line tools from [BBEdit's website][1].

## If you bought BBEdit direct… ##

If you didn't install the Command Line Tools when you first installed BBEdit, you can use the "Install Command Line Tools…" menu in BBEdit:

![Install Menu]

If you're not sure if you installed them or not, just select the menu item, and it will tell you if they are already installed and up-to-date.

## SSH (optional) ##

BBEdit is great when you are logged in via the GUI, but what happens if you connected to your Mac via SSH?

`bbwait` can help here too, but it requires that you add something like this to your shell initialization file (i.e. .zshrc or .bashrc):

		PPID_NAME=$(/bin/ps -cp ${PPID} | awk -F' ' '/sshd/{print $NF}')

		if [ "$PPID_NAME" = "sshd" ]
		then
			 SSH=yes
		else
			SSH=no
		fi
		export SSH


Note: Obviously you could use the PPID_NAME check to set your EDITOR/VISUAL variable directly based on whether or not you are connecting via ssh:

		PPID_NAME=$(/bin/ps -cp ${PPID} | awk -F' ' '/sshd/{print $NF}')

		if [ "$PPID_NAME" = "sshd" ]
		then
			SSH=yes
			EDITOR='/usr/bin/nano'
		else
			SSH=no
			EDITOR='/usr/local/bin/bbwait'
		fi
		export SSH

but if you have added

		Defaults 	editor=/usr/local/bin/bbwait

to /etc/sudoers then `bbwait` will be used regardless of your EDITOR setting.

If `bbwait` sees that SSH is set to `yes` it will look for other, command-line based editors. By default it will look for `nano`, then `pico`, and then (*shudder*) `vi`. You can adjust what it uses by editing the choices under

		if [ "$SSH" = "yes" ]
		then

in the `bbwait` script. The advantage to doing it this way is that you can keep `bbwait` as your \$EDITOR and it will do the right thing.

Note: if you want that SSH switch to work for `visudo` you need to add this line to /etc/sudoers 

		Defaults	env_keep += "SSH"

Otherwise it will use BBEdit even when you are logged in via ssh.


## If you don't use BBEdit… ##

Although this is written for BBEdit, it could easily be adjusted for [TextWrangler]. In fact, it would probably work with any other GUI *text editor* by replacing the `bbedit` command with your preferred editor's command-line tool. If your app does not provide one, you could try something like this:


		open -a AppNameHere -W -n --args "$@"

but I have not tested that.


<!-- Reference Links -->

[1]: http://www.barebones.com/support/bbedit/cmd-line-tools.html

[BBEdit]: http://www.barebones.com/products/bbedit/

[Install Menu]: BBEdit-InstallCommandLineTools-Menu.jpg

[BBEdit authorization prompt]: bbwait-prompt-1-authenticate.jpg

[BBEdit Unlock Prompt]: bbwait-prompt-2-unlock.jpg

[TextWrangler]: http://www.barebones.com/products/textwrangler/
