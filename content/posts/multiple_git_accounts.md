---
title: "Effortlessly Manage Multiple Git Accounts"
date: 2020-05-13T16:14:59-04:00
draft: false
---

## Introduction

Managing multiple git accounts becomes essential when you have multiple accounts that you want to contribute to. This tutorial will guide you through the process of seamlessly switching between two git accounts without any hassle.

## Steps

Follow these steps to effectively manage multiple git accounts:

### 1. Create Git Accounts

Start by creating two git accounts, either on platforms like GitHub or GitLab, or using existing accounts from your workplace. For this tutorial, we will use two accounts created on GitHub.com.

### 2. Generate SSH Keys

Next, you need to generate SSH keys and add them to your macOS.

#### For Account-1:
Generate the SSH key by running the following command in the terminal:
```
ssh-keygen -t ed25519 -C "john.doe@gmail.com"

> Enter file in which to save the key (/Users/uday/.ssh/id_ed25519): github_id
```

#### For Account-2:
Generate another SSH key using the following command:
```
ssh-keygen -t ed25519 -C "john.smith@gmail.com"

> Enter file in which to save the key (/Users/uday/.ssh/id_ed25519): vegeta_id
```

### 3. Add Config File

Now, create a config file that includes the configurations for both accounts:

```
% vi config

# Account-1
Host github.com
  HostName github.com
  AddKeysToAgent yes
  IdentityFile ~/.ssh/github_id

# Account-2
Host github.com-vegeta
  HostName github.com
  AddKeysToAgent yes
  IdentifyFile ~/.ssh/vegeta_id
```

### 4. Add SSH Private Key to the ssh-agent and Store Passphrase

Add the SSH private keys to the ssh-agent and store the passphrase in the keychain:

```
% ssh-add --apple-use-keychain ~/.ssh/github_id
> Identity added: /Users/uday/.ssh/github_id (john.doe@gmail.com)

% ssh-add --apple-use-keychain ~/.ssh/vegeta_id
> Identity added: /Users/uday/.ssh/vegeta_id (john.smith@gmail.com)
```

### 5. Add SSH Public Key to Your GitHub Account

Copy the public key to your clipboard using the following command:
```
% pbcopy < github.pub
```

Go to your GitHub account settings, add the copied text as an SSH key with a new name. You will be prompted for a password before making this change.

By following these steps, you can effortlessly manage multiple git accounts and switch between them without any issues.
