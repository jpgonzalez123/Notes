# How to configure multiple git accounts with multiple ssh keys in the same computer.

## Introduction
This is a quick guide based on my experiences on how to configure multiple git accounts in the same computer and avoid having to set local git configurations for each one of the repositories.

I hope this is useful to reduce time to configure. I always keep getting searching for the same pages (Set as references) each time I have to configure my machine.

## Solution
To configure multiple accounts on the same computer we need to:
1. Create the SSH keys

    ```bash
    # Create Personal SSH
    ssh-keygen -t ed25519 -C "john.Doe@example.com" -f "~/.ssh/id_ed25519_john_doe_example"
    # Create Professional SSH
    ssh-keygen -t ed25519 -C "jDoe69@enterprise.com" -f "~.ssh/id_ed25519jDoe69_enterprise"
    ```

1. Add Keys to SSH Agent

    ```bash
    # Add SSH keys to agent
    ssh-add ~/.ssh/id_ed25519_john_doe_example
    ssh-add ~/.ssh/id_ed25519jDoe69_enterprise
    ```
1. Add the SSH keys to each Source Control Service
1. Create a config file following [these instructions](https://stackoverflow.com/a/38454037/20231727)
    
    ``` bash
     $ nano ~/.ssh/config

    Host github.com # This is the alias to call when cloning for personal
        User git
        Hostname github.com # This is the real host
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/id_ed25519_john_doe_example # This is the created private key for personal account

    Host github.enterprise # This is the alias to call when cloning for enterprise
        User git
        Hostname github.com # This is the real host
        PreferredAuthentications publickey
        IdentitiesOnly yes
        IdentityFile ~.ssh/id_ed25519jDoe69_enterprise # This is the created private key for professional account
    ```

    Key points: 
    * Create an alias for each account
    * Use independent ssh files
    * Need to use the private Key, no the public key

1. Configure a file structure that will allow to separate the accounts logically by use:
    
    ``` bash
    # Personal use
    ~/repos/Personal/

    #Professional use
    ~/repos/Professional/ 
    ```
1. Set the identities for git depending on the file path.
    1. Create the `~/.gitconfig` file if it doesn't exists
    1. Set the default `user` and `email` for the *global* git configuration (Same as setting ` git config --global user.name "FIRST_NAME LAST_NAME"` and `git config --global user.email "MY_NAME@example.com"`)
    
    ```properties
    # Main User
    [user]
        name = John Doe
        email = john.Doe@example.com
    ```

1. Set the following `[includeIf "gitdir:~/repos/Professional/"]` set of credentials to identify the new configuration for all git repositories cloned on that parent folder:

    ```properties
    # Main User
    [user]
        name = John Doe
        email = john.Doe@example.com
    
    # Secondary user
    [includeIf "gitdir:~/repos/Professional/"]
        path = ~/repos/Professional/.ssh/.gitconfig
    ```
    Important notes on this step:
    * The file `~/repos/Professional/.ssh/.gitconfig` doesn't exists *yet*. Will be created later.
    * The path `~/repos/Professional/.ssh/.gitconfig` is not required to be specific, *I* personally do it to keep the structure on ssh configurations.
    * Alternatively you could specify the credentials on both specific paths and avoid using the `Global` configuration:

    ```properties
    # Personal user
    [includeIf "gitdir:~/repos/Personal/"]
        path = ~/repos/Personal/.ssh/.gitconfig

    # Professional user
    [includeIf "gitdir:~/repos/Professional/"]
        path = ~/repos/Professional/.ssh/.gitconfig
    ```
1. Create the respective `.gitconfig` that you have specified on the main `~/.gitconfig` file with new configuration you want to use for that repo:
    
    `~/repos/Personal/.ssh/.gitconfig`
    ```properties
    # ~/repos/Personal/.ssh/.gitconfig
    [user]
        name = John Doe
        email = john.Doe@example.com
    ```

    `~/repos/Professional/.ssh/.gitconfig`
    ```properties
    # ~/repos/Professional/.ssh/.gitconfig
    [user]
        name = John Doe
        email = jDoe69@enterprise.com
    ```

## All Set!

Use examples:
* Personal use:
    1. Use the host defined in the personal section of the ` ~/.ssh/config` file
    1. In case the credentials are being configured in a `[includeIf "gitdir:~<File Directory>"]` in the `~/.gitconfig`, the repository needs to be cloned as a child of the `<File Directory>`; it doesn't need to be on the first level, it can be at any depth level.

```bash
Git clone git@github.com:<Github User>/<Github Repo>
```

* Professional use:
    1. Use the host defined in the professional section of the ` ~/.ssh/config` file
    1. The repository needs to be cloned as a child of the `<File Directory>` of the professional `[includeIf "gitdir:~<File Directory>"]`; it doesn't need to be on the first level, it can be at any depth level.
    1. Note the host is the professional host (The section between the "@" and the colon ":")

```bash
Git clone git@github.enterprise:<Github User>/<Github Repo>
```

## References:
* [SSH configuration file](https://www.ssh.com/academy/ssh/config)
* [GIT Includes](https://git-scm.com/docs/git-config#_includes)
* [Best way to use multiple SSH private keys on one client](https://stackoverflow.com/a/38454037/20231727)
* [Very similar guide done by @bgauduch](https://gist.github.com/bgauduch/06a8c4ec2fec8fef6354afe94358c89e)

