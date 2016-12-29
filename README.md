# lpass-env

This is an (unofficial) script which wraps the LastPass CLI [lpass](https://github.com/lastpass/lastpass-cli) to manage environment variables like API keys, passwords, &c. Of course, you can use it for non-secret configuration variables too.

This script is intended as a replacement to putting variables in `~/.bash_profile`, and it's better because:

* Variables are not stored in a plaintext file, only in the LastPass vault.
* Variables are not injected into every shell environment, so malicious scripts can't read your secrets unless you are running them inside an `lpass-env shell`.
* Variables can easily be synced with your other machines.
* Variables can be shared with others using LastPass instead of copy/paste.
* Variables can be managed in the LastPass vault from any PC.

On the other hand, using LastPass introduces quite a bit of overhead compared to `~/.bash_profile`. If you aren't familiar with the [lpass](https://github.com/lastpass/lastpass-cli) tool, check that out first -- there's a bit of a learning curve.

## Installation

It's just a Bash script. You can install it however you want, but something like this seems pretty popular:

``` bash
curl https://raw.githubusercontent.com/luketurner/lpass-env/master/lpass-env -o /usr/local/bin/lpass-env && chmod +x /usr/local/bin/lpass-env
```

However, `lpass-env` depends on the LastPass CLI tool `lpass`, which must also be installed on your machine. See the [lpass docs](https://github.com/lastpass/lastpass-cli) for more information about that. (Spoiler for OS X users: `brew update && brew install lastpass-cli --with-pinentry`) 

## Usage

`lpass-env` is a tiny script with a simple interface, that calls into `lpass` to do the dirty work of actually managing credentials. The wrapper script itself is extremely short and (hopefully) self-documenting. However, it will probably take some practice to take advantage of it in your workflow. You can run `lpass-env help` for help and usage examples, or just look at them here:

```
Usage: lpass-env [action] [lpass-key-id]

Valid actions: help, shell, print, export

Examples

  Launch a subshell with env vars:
    $ lpass-env shell my/key/name

  Write the vars to a file:
    $ lpass-env print my/key/name > ~/my_vars.sh

  Export the vars into running shell:
    $ $(lpass-env export my/key/name)
```

### Managing credentials with lpass

The `lpass-env` script does not provide an easy way to add and edit credentials, because the existing `lpass edit` command is already ergonomic enough. Instead, `lpass-env` gives a read-only interface and assumes that it will only be used with LastPass credentials where the `notes` field contains environment variable declarations. The declarations should not include `export`, `readonly`, or (obviously) `local`.

Consider this example, which creates a new credential and then exports it into the current shell session:

``` bash
# Create a credential with two environment keys in the notes field
$ cat | lpass edit --notes my-dev-environment << EOF
MY_API_KEY=asdf
MY_API_SECRET=fdsa
EOF

# Exports the credentials into our environment
# Note that 'lpass-env export' returns executable code, which is why
# it's wrapped in $( ... )
$ $(lpass-env export my-dev-environment)

# Now, 'env' includes MY_API_KEY and MY_API_SECRET
$ env | grep MY_API
```

(An alternative way to get variables is to use the `lpass-env shell` to create a subshell with the right stuff in the `env` automatically: `lpass-env shell my-dev-environment`.)

### Organizing credentials

I have a practice of putting all my `lpass-env` credentials inside an `ENV` folder in my LastPass vault. Within that folder, I create LastPass credentials for all my API keys, access tokens, or whatever. (I have nested subfolders, too.)

It is possible to have more than one environment variable declared per credential, and I use that feature to declare codependent variables side-by-side (for example, if you have an access key plus a secret key, or a set of configuration variables for an app). However, if the variables are not codependent, I declare them in separate credentials so that I can limit which applications can see which credentials, and also to make sharing them easier.

---

Copyright (c) 2016 Luke Turner
Released under MIT License (SPDX:MIT)