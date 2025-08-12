+++
date = '2025-08-12T14:58:03+02:00'
draft = false
title = 'Setup Git Signed Commits'
+++

o easily validate my commits, I've configured Git to use signed commits across all my computers. 
Most of my configuration is consistent across all computers, managed within my dotfiles on GitHub.
**`~/.gitconfig`**:
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
As you may or may not see, this config imports a .gitsecrets file from my home directory. This secrets file stores my GPG signing key, Git username, email, and other sensitive information.
## Settin up the .gitsecrets file
Setting up the `.gitsecrets` file involves a few steps.
1. Determine the name we want to use for committing to GIT repositories. let's say we use the name `John Doe` with the email `John@example.com`
2. Now we need to create the gpg signing key for John Doe if it does not already exist, this is easiest done by running `gpg --gen-key` command, and entering the name, and email when prompted. To ensure the signing key can only be used by you, always provide a passphrase that must be entered when signing commits.
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

Once these steps are complete, you can sign your commits. However, to ensure commit verification, you should also export your public key and import it to your Git hosting provider, whether it's GitLab, GitHub, Bitbucket, or another service.
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
