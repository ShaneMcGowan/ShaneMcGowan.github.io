---
title: How to use different SSH Keys for different Bitbucket (git) Repositories
published: true
description: A guide on how to use configure multiple ssh keys for use with GitHub
tags: git, bitbucket, ssh, github
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/8t2c9ur26cm7umrcs6mz.png
---

Bitbucket doesn't allow you to use the same SSH Key on more than one account. This means if you have more than 1 account but need to access the repositories on both (in my case a work account and a personal account) you will need to configure your SSH agent to allow you to choose which SSH Key to use when cloning over SSH.

Navigate to your SSH Directory
```bash
cd ~/.ssh
```

Ensure SSH Agent is running
```bash
eval $(ssh-agent)
```

Create your SSH Keys
```bash
# Run the following replacing EMAIL_ADDRESS with the email address associated with your bitbucket account
ssh-keygen -t rsa -b 4096 -C "EMAIL_ADDRESS"

# When prompted to enter a file name, enter a name to help you differentiate your two keys
Enter file in which to save the key (/c/Users/shane/.ssh/id_rsa): key-1
```

(optional) Remove previous SSH Keys for your SSH Agent (this does not delete the actual key from your ~/.ssh directory)
```bash
ssh-add -D
```

Add your newly generated SSH Keys to your SSH Agent
```bash
ssh-add ~/.ssh/key-1
ssh-add ~/.ssh/key-2
```

Check that your SSH Keys have been added
```bash
ssh-add -l

# Sample output
4096 SHA256:Mo+oiomJASfjlksc6Qj0KcQpUXRVpyvsEIHdXNYDL1c user@website1.com (RSA)
4096 SHA256:6Jt908009upoASFjjkljsflkj9iaGebl7pNr6vuezPI user@website2.com (RSA)
```

Create your SSH Config file
```bash
touch config
```

The format of our SSH Config
```bash
Host bitbucket-key-1 # Friendly name
  HostName bitbucket.org # The server we opening the SSH connection with
  User git # The SSH user (which for bitbucket is git eg. git@bitbucket.org)
  IdentityFile ~/.ssh/id_rsa # The private key file
  IdentitiesOnly yes # Tells SSH to only use keys provided by IdentityFile above
```

Create your different SSH Configs
```bash
# Bitbucket - SSH Key 1
Host bitbucket-key-1
  HostName bitbucket.org  
  User git
  IdentityFile ~/.ssh/key-1-id_rsa
  IdentitiesOnly yes

# Bitbucket - SSH Key 2
Host bitbucket-key-2
  HostName bitbucket.org
  User git
  IdentityFile ~/.ssh/key-2-id_rsa
  IdentitiesOnly yes
```

*Important* At this point, reload your terminal

Now when cloning via SSH we replace the usual host name (bitbucket.org) with our new "friendly" name
```bash
# Usual format
git clone USER@HOST:ACCOUNT_NAME/REPOSITORY_NAME.git
# Example
git clone git@bitbucket.org:username/repository.git

# Format using our new SSH Host config (replace bitbucket.org with the user friendly host name from our .config file)
git clone git@bitbucket-key-1:username/repository.git

```