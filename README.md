# simplify-bitwarden-cli-commands

The purpose of this is to provide the code snippets I use within .bashrc to simplify and extend Bitwarden CLI commands.

The rationale behind this is that the default Bitwarden CLI commands return complex results that likely require formatting, in-turn greatly affecting productivity.

## Prerequisites

1. You must be able to execute Bash (unix shell) commands. Most linux distributions use Bash as their shell scripting language by default.
     - At the time of this writing, I'm using Ubuntu 20.04 LTS 64bit.

2. You must be able to access your .bashrc script.
     - Usually the .bashrc script file is found in the home (~) directory.

3. You must have Bitwarden CLI installed.

4. This Git uses Nano as the default text editor - feel free to use any editor of your choice, but if you do, please take note of my Nano calls and update this accordingly.

5. It's strongly recommended you have xclip installed, otherwise you will need to edit the xclip calls to a clipboard manager of your choice (E.G clip.exe).

## Step 1 - Open .bashrc

Use the followiung commands to open up your .bashrc script.

```
# Open up the .bashrc script from your home directory
nano ~/.bashrc
```

## Step 2 - Modify .bashrc with new functions / alias

The following functions will allow usage of simplified commands to return specific data from Bitwarden.

```
# Create an alias to unlock the Bitwarden vault by adding the current session key to the Bitwarden session environment variable
alias bwu='export BW_SESSION=$(bw unlock --raw > ~/.bw_session && cat ~/.bw_session)'

# Create a function to get a password for a specific item in the Bitwarden vault and add it to the clipboard
function bwgp () { local test=$(export BW_SESSION=~/.bw_session) && bw get password "$1" | xclip; }

# Create a function to get a Time-based One Time Password (TOTP) for the specific Bitwarden item and add it to the clipboard
function bwgt () { local test=$(export BW_SESSION=~/.bw_session) && bw get totp "$1" | xclip; }

# Create a function to get a specified item from the Bitwarden vault and output in formatted JSON
function bwgi () { local test=$(export BW_SESSION=~/.bw_session) && bw get item --pretty "$1"; }

# Create a function to get the ID, name, and username of a list of items filtered by a search string
function bwli () { local test=$(export BW_SESSION=~/.bw_session) && bw list items --search "$1" --pretty | egrep -i 'name|"id":'; }

# Create a function to get the value of any notes stored with a specific item in the Bitwarden vault and add it to the clipboard
function bwgn () { local test=$(export BW_SESSION=~/.bw_session) && bw get item "$1" | grep -oP '(?<="notes":")[^"]*' | xclip;
```


>**Please note:** It is not recommended to export the session key to your environment to avoid session keys persisting on an unprotected disk.

>**Please note:** When using the search function, enclosing strings in quotes will attempt to match the whole search string within any text within Bitwarden items and return these items that match. You may need to ensure your Bitwarden items are uniquely named.

## Step 3 - Update .bashrc to clean up any older Bitwarden sessions

At the bottom of your script, use the following code to clean any previous session keys, reset the session key, and unlock the Bitwarden vault:

```
# Remove bitwarden sessions older than a day
if [[ $(find ~/.bw_session -mtime +1 -print) ]]; then rm ~/.bw_session; fi

# If bitwarden session already set don't overwrite
if [ -n "$BW_SESSION" ]; then echo "Bitwarden set";

# Else if there is a session file set from there
elif [ -f ~/.bw_session ]; then export BW_SESSION=$(cat ~/.bw_session);

# Otherwise unlock to start new session
else bwu; fi
```

## Step 4 - Save and close the text editor

Within Nano, press ```CTRL+X``` to exit, and if any changes were made, you will be prompted to save the file (type "Y") and then choose the filename to save the file to. Press enter to save to the current file.

## Step 5 - Reload the .bashrc script

You must reload the .bashrc script to enact the changes we've made:

```
# Reload the .bashrc script
. ~/.bashrc
```

**Useful tip:** If you added any items to Bitwarden from the GUI while you have already have a session active, you can resync these changes with ```bw sync```.
