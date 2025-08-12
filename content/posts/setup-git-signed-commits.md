+++
date = '2025-08-12T14:58:03+02:00'
draft = false
title = 'Setup Git Signed Commits'
+++

To make it easier to validate what commits I have made, I have set up git to use signed committs on all of my computers. To simplify my setup, most of my configuration is the same across all computers, found in my dotfiles (here)[https://github.com/NorthernLightsDevel/dotfiles/blob/main/.gitconfig]
**Here is my gitconfig**:
```ini,config
[include]
	path = ~/.gitsecrets
[alias]
	s = status
	cm = commit -S -m
	visual = !gitextensions browse .&
	amend = commit --amend
	co = checkout
	br = branch
	ca = commit -a -S -m
	unstage = reset HEAD --
	rst = reset HEAD --hard
	cb = checkout -B
	history = log --oneline --graph --decorate --all
	untrack = update-index --assume-unchange
	track = update-index --no-assume-unchange
	test = log --oneline --graph
	update = "!f() { git fetch -u origin \"$1:$1\"; };f"
	feature = "!f() { git worktree add ../$1 --checkout feature/$1;f"
	bugfix = "!f() { git worktree add ../$1 --checkout bugfix/$1;f"
[merge]
	ff = no
	tool = nvimdiff
[diff]
	guitool = kdiff3
	tool = nvimdiff
[core]
	autocrlf = false
	editor = nvim
	eol = lf
[pull]
	rebase = true
	ff = only
[fetch]
	prune = false
[rebase]
	autoStash = false
[branch "master"]
	mergeOptions = --no-ff
[filter "lfs"]
	clean = git-lfs clean -- %f
	smudge = git-lfs smudge -- %f
	process = git-lfs filter-process
	required = true
[commit]
    gpgSign = true
[init]
	defaultBranch = main
[rerere]
	enabled = false
[url "ssh://git@github.com/"]
	insteadOf = https://github.com/
```
As you may or may not see, this config imports a .gitsecrets file from my home directory. This secrets file is where gpg signing key, git username, email and other secrets are stored.
to set up the .gitsecrets file, we need to execute a couple of steps.
1. Determine the name we want to use for committing to GIT repositories. let's say we use the name `John Doe` with the email `John@example.com`
2. Now we need to create the gpg singing key for John Doe if it does not already exist, this is easiest done by running `gpg --gen-key` command, and entering the name, and email when prompted. Because we want to ensure that the signing key can only be used by us, we should always provide a passphase we need to enter when signing commits.
3. edit `~/.gitsecrets` and add the key info e.g.
   ```ini,config
   [user]
   email = john@example.com
   name = John Doe
   signingKey = C68206831C3DFE79
   ```
   **To get the signingKey ID** run the command `gpg --list-secret-keys --keyid-format=long john@example.com` which would output something like this
   ```
   sec   ed25519/C68206831C3DFE79 2025-08-12 [SC] [expires: 2028-08-11]
         FDF3B250D19953EE4AA95C76C68206831C3DFE79
   uid                 [ultimate] John Doe <john@example.com>
   ssb   cv25519/25798C317462CC6D 2025-08-12 [E] [expires: 2028-08-11]

   ```
   where the key id is part of the sec line.

If you followed these steps, you should be able to sign your committs, however, signing commits if they are not verified might not make much sense, so we also want to export our key and import it to our git host server, wether it's gitlab, github, bitbucket or something else.
- **Export public key**:
  ```bash
  gpg --armor --export john@example.com
  ```
  Example output:
  ```pubkey
  -----BEGIN PGP PUBLIC KEY BLOCK-----
  
  mDMEaJs86RYJKwYBBAHaRw8BAQdA5JLR4Rzc+kD2o+yU9H6ZYj/9OMS5ZpmLdq/Q
  tgrvTLC0G0pvaG4gRG9lIDxqb2huQGV4YW1wbGUuY29tPoiWBBMWCgA+FiEE/fOy
  UNGZU+5KqVx2xoIGgxw9/nkFAmibPOkCGwMFCQWjmoAFCwkIBwIGFQoJCAsCBBYC
  AwECHgECF4AACgkQxoIGgxw9/nkzggD8CXPkxAcefKW7PC580+bcTMRf5b4jhGO9
  a7U5iLZOVfkBALSIcxW++ipRVh/uWRv/ryKrng+4CgqtPGJlPjDoZ9wEuDgEaJs8
  6RIKKwYBBAGXVQEFAQEHQP50DhBUofX5je+n/ljYiGLmlrKqx9Icnc+LnHhk6VVY
  AwEIB4h+BBgWCgAmFiEE/fOyUNGZU+5KqVx2xoIGgxw9/nkFAmibPOkCGwwFCQWj
  moAACgkQxoIGgxw9/nlVQQD/XdWjs7FEsRirSP1/RbOKEZ+psTxpPBctzCDPp4lE
  SwMBAOLyQgjxJGIDgb9GhJ7ND3knOdvOwG1xGEwy6W6x16cH
  =BYeb
  -----END PGP PUBLIC KEY BLOCK-----
  ```
The secret key block can now be copied, and pasted to github in the keys section [https://github.com/settings/keys] I would strongly advice to also add an ssh key, to be able to use git over ssh.
