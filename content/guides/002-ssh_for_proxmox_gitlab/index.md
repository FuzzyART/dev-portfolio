---

title: "Using SSH Keys with a Local GitLab Installation"
date: 2026-08-22
draft: false
------------

When using a local GitLab installation, SSH access requires a little more configuration than a standard GitLab.com setup.

In this setup, GitLab is available through the VM's IP address, while Git over SSH is exposed on port `2222`.

This means that the SSH repository address shown by GitLab may not be the most convenient address to use locally.

## Create an SSH key

Create a new Ed25519 SSH key:

```bash
ssh-keygen -t ed25519 -C "comment"
```

When prompted for the location, store the key in the `.ssh` directory.

For example:

```text
~/.ssh/gitlab_local
```

This creates two files:

```text
~/.ssh/gitlab_local
~/.ssh/gitlab_local.pub
```

The private key is:

```text
~/.ssh/gitlab_local
```

The public key is:

```text
~/.ssh/gitlab_local.pub
```

The private key should never be shared.

## Add the public key to GitLab

Display the public key:

```bash
cat ~/.ssh/gitlab_local.pub
```

Copy the complete output.

In GitLab, open your user settings and go to:

**Preferences → SSH Keys**

Depending on the GitLab version, this may also be accessible through your profile settings.

Add the contents of:

```text
~/.ssh/gitlab_local.pub
```

to the SSH key field.

Give the key a descriptive title, such as:

```text
My local workstation
```

Then save the key.

The public key is now associated with your GitLab user account.

## Configure SSH

There are two ways to configure the SSH connection.

The first is a simple configuration that applies the GitLab key to all SSH hosts. The second, and more flexible approach, creates a dedicated SSH host entry for GitLab.

### Simple configuration

Add the following to:

```text
~/.ssh/config
```

```sshconfig
Host *
    IdentityFile ~/.ssh/gitlab_local
    AddKeysToAgent yes
```

With this configuration, the `gitlab_local` key is used for all SSH connections.

The repository can then be cloned using the GitLab VM's IP address and SSH port:

```bash
git clone ssh://git@192.168.x.x:2222/<group_name>/<repo_name>.git
```

Replace:

* `192.168.x.x` with the IP address of the GitLab VM
* `<group_name>` with the GitLab group
* `<repo_name>` with the repository name

For example:

```bash
git clone ssh://git@192.168.1.24:2222/my-group/my-project.git
```

This works because Git is explicitly told to connect to port `2222`.

## Advanced SSH configuration

A better solution is to create a dedicated SSH configuration entry for GitLab.

Add this to:

```text
~/.ssh/config
```

```sshconfig
Host gitlab
    IdentityFile ~/.ssh/gitlab_local
    AddKeysToAgent yes
    HostName 192.168.1.24
    User git
    Port 2222
```

The important part here is that `gitlab` is an SSH alias.

The actual connection parameters are hidden behind the alias:

```text
Host gitlab
    HostName 192.168.1.24
    User git
    Port 2222
```

When SSH sees:

```text
gitlab
```

it actually connects to:

```text
git@192.168.1.24:2222
```

using the `gitlab_local` identity.

The repository can therefore be cloned with:

```bash
git clone ssh://gitlab/<group_name>/<repo_name>.git
```

For example:

```bash
git clone ssh://gitlab/my-group/my-project.git
```

This is much cleaner than repeatedly specifying the IP address and port.

## Why not use GitLab's SSH URL directly?

This is the slightly confusing part of a local GitLab installation.

GitLab generates repository URLs based on its configured external URL and SSH configuration. The web address and the address used by your local SSH client don't necessarily look the same.

In this setup, GitLab's web interface is accessed through:

```text
http://192.168.1.24
```

while SSH is exposed through:

```text
192.168.1.24:2222
```

The SSH configuration maps the convenient local name:

```text
gitlab
```

to those connection details.

So instead of having to remember:

```bash
ssh://git@192.168.1.24:2222/<group_name>/<repo_name>.git
```

the SSH alias allows:

```bash
ssh://gitlab/<group_name>/<repo_name>.git
```

The latter is also easier to change later. If the GitLab VM gets a different IP address, only `~/.ssh/config` needs to be updated.

## Test the SSH connection

Before cloning a repository, test the connection directly:

```bash
ssh -T gitlab
```

With the advanced configuration, SSH should use:

```text
Host: gitlab
HostName: 192.168.1.24
User: git
Port: 2222
IdentityFile: ~/.ssh/gitlab_local
```

If the connection is successful, GitLab should recognize the SSH key associated with your account.

For troubleshooting, use verbose SSH output:

```bash
ssh -vT gitlab
```

This is particularly useful for checking which key SSH is attempting to use and whether it is connecting to the expected host and port.

## Recommended configuration

For a local GitLab installation, I prefer the dedicated SSH host configuration:

```sshconfig
Host gitlab
    IdentityFile ~/.ssh/gitlab_local
    AddKeysToAgent yes
    HostName 192.168.1.24
    User git
    Port 2222
```

It keeps the Git commands short:

```bash
git clone ssh://gitlab/<group_name>/<repo_name>.git
```

and keeps the GitLab-specific connection details in one place.

This also avoids changing the SSH identity used for unrelated servers, which is a drawback of the `Host *` configuration.

