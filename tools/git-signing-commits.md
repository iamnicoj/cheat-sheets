# Git Signing commits

Tags: #macos #git #github 

Download [gpg suit for mac](https://gpgtools.org/)

From [here](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key)

You can use GPG to sign commits with a GPG key that you generate yourself.

GitHub uses OpenPGP libraries to confirm that your locally signed commits and tags are cryptographically verifiable against a public key you have added to your account on GitHub.com.

To sign commits using GPG and have those commits verified on GitHub, follow these steps:


```bash
# Make sure you select a length of 4096 bits
gpg --full-generate-key

# Get the keys
gpg --list-secret-keys --keyid-format=long

/Users/hubot/.gnupg/secring.gpg
------------------------------------
sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
uid                          Hubot <hubot@example.com>
ssb   4096R/4BB6D45482678BE3 2016-03-10

$ gpg --armor --export 3AA5C34371567BD2
# Prints the GPG key ID, in ASCII armor format

```

1 Copy your GPG key, beginning with `-----BEGIN PGP PUBLIC KEY BLOCK-----` and ending with `-----END PGP PUBLIC KEY BLOCK-----`.

[Adding a GPG key to your GitHub account](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account)

Then configure git like [this](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key):

```bash
git config --global --unset gpg.format

gpg --list-secret-keys --keyid-format=long

/Users/hubot/.gnupg/secring.gpg
------------------------------------
sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
uid                          Hubot <hubot@example.com>
ssb   4096R/4BB6D45482678BE3 2016-03-10

$ git config --global user.signingkey 3AA5C34371567BD2

```

Optionally, to configure Git to sign all commits by default, enter the following command:

```Shell
$ git config --global commit.gpgsign true
```

If you aren't using the GPG suite, run the following command in the `zsh` shell to add the GPG key to your `.zshrc` file, if it exists, or your `.zprofile` file:

```shell
$ if [ -r ~/.zshrc ]; then echo 'export GPG_TTY=$(tty)' >> ~/.zshrc; \
  else echo 'export GPG_TTY=$(tty)' >> ~/.zprofile; fi
```

1.  Optionally, to prompt you to enter a PIN or passphrase when required, install `pinentry-mac`. For example, using [Homebrew](https://brew.sh/):
    
    ```shell
    $ brew install pinentry-mac
    $ echo "pinentry-program $(which pinentry-mac)" >> ~/.gnupg/gpg-agent.conf
    $ killall gpg-agent
    ```

____

## Signing commits

```shell
$ git commit -S -m "YOUR_COMMIT_MESSAGE"
# Creates a signed commit
```

Past commits

```bash
git commit --amend --no-edit -s
```