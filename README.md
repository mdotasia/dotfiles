# dotfiles

This repository serves two use cases:

1. [Personalize GitHub Codespaces containers with my own dotfiles](https://docs.github.com/en/github/developing-online-with-codespaces/personalizing-codespaces-for-your-account).
1. Bootstrap a MacBook, both for a new MacBook or to restore a MacBook. I use
   [Strap](https://github.com/MikeMcQuaid/strap) to bootstrap a MacBook using
   this dotfiles repository.

If you want to create your own dotfiles repo, start with
[Mike McQuaid's dotfiles](https://github.com/MikeMcQuaid/dotfiles).

## First use case: personalizing GitHub Codespaces containers

See
[Personalizing Codespaces for your account](https://docs.github.com/en/codespaces/customizing-your-codespace/personalizing-codespaces-for-your-account)
in the GitHub docs.

## Second use case: bootstrap a MacBook

Bootstrapping a MacBook is a combination of automated and manual steps. My goal
is to improve this workflow over time and eliminate manual steps whenever
possible.

Bootstrapping a MacBook relies on the following cloud services:

- [LastPass](https://www.lastpass.com/) to synchronize passwords and secrets
  across devices.
- [Google Chrome Sync](https://support.google.com/chrome/answer/185277?) to
  synchronize extensions and settings.
- [VS Code Settings Sync](https://code.visualstudio.com/docs/editor/settings-sync)
  to synchronize extensions and settings for VS Code.
- [Tresorit](https://tresorit.com/) to synchronize personal files.

### Setup assistant

On new MacBooks or after a factory reset the first thing you see is the setup
assistant:

- Do not link your local user account to iCloud and prevent local user account
  password reset with your Apple ID via iCloud.
- Turn on FileVault and write down the recovery key.
- Do not turn on iCloud keychain.

Above choices mean that you take full responsibility for your MacBook.

### Automated installation with Strap

Go to https://macos-strap.herokuapp.com/ and download a customized `strap.sh`.
To download the customized `strap.sh` you need give Strap access to your GitHub
account and authorize it as OAuth app. Strap will then customize the file that
you are about to download with your GitHub username and pull your dotfiles from
`https://github.com/<username>/dotfiles`.

For the GitHub authorizaton you need your GitHub password. If you keep it in a
password manager like [LastPass](https://www.lastpass.com/), your password might
be difficult or impossible to type in manually. With
[Apple's universal clipboard](https://support.apple.com/en-us/HT209460), you can
copy your GitHub password from a password manager on another device and paste
into you new machine.

Follow the instructions on https://macos-strap.herokuapp.com/ to bootstrap your
MacBook.

### Rerunning the automated installation

If the automated installation fails or if you make changes in your `dotfiles`
repository, you can run Strap again to install new dependencies and to copy new
dotfiles. Homebrew will skip anything that has been installed already. However,
Strap does not remove obsolete symlinks in `~/`.

### Environment variables with secrets and other secrets

Since all your dotfiles are public, you cannot store secrets in them. You can
create `~/.env` and define environment variables with secrets. File `~/.zshrc`
will source this file. `~/.env` has been added to `.gitignore` to prevent
accidental commit of secrets.

You can also implement a more sophisticated approach and store your secrets in
LastPass and retrieve them during installation and copy them into `~/.env`. This
how SSH keys are handled.

### Setting system preferences manually

Most system preferences that you would normally have to configure manually can
be scripted. This is what script `macos` is for. In macOS Catalina there are
settings that cannot be scripted and need to be configured manually:

#### Language & Region

| Setting                         | Value  | Comment                                                             |
| :------------------------------ | :----- | :------------------------------------------------------------------ |
| General → First day of the week | Monday | Scripted configuration of this setting is broken in macOS Catalina. |

#### Security & Privacy

| Setting                                                        | Value       | Comment                                                             |
| :------------------------------------------------------------- | :---------- | :------------------------------------------------------------------ |
| General → Require password X after sleep or screensaver begins | immediately | Scripted configuration of this setting is broken in macOS Catalina. |
| Privacy                                                        |             | Review all privacy settings.                                        |

#### Users & Groups

| Setting | Comment                 |
| :------ | :---------------------- |
| Image   | Choose preferred image. |

#### Sharing

Set computer name.

## How this works under the hood

### Local GitHub repository

Strap clones this repository to `~/.dotfiles` and symlinks any dotfile to `~/`.
If you change any dotfiles, you can diff the changes with Git in `~/.dotfiles`
and commit them.

### Scripts

All scripts in the `script` folder need to be owner executable. Note that the
owner executable flag is part of a GitHub commit and will be set when you clone
this repository.

### Brewfile

CLI tools and applications are installed with [Homebrew](https://brew.sh/) or
directly from the [macOS App Store](http://www.apple.com/mac/app-store).
`Brewfile` defines which tool or app is installed from which source.

You can think of `Brewfile` as your `package.json` for system dependencies
([this post](https://thoughtbot.com/blog/brewfile-a-gemfile-but-for-homebrew)
came up with this analogy). Strap uses
[`brew bundle`](https://github.com/Homebrew/homebrew-bundle) to install
dependencies the following taps:

1. [homebrew-core](https://github.com/Homebrew/homebrew-core),
1. [homebrew-cask](https://github.com/Homebrew/homebrew-cask) and
1. [macOS App Store](http://www.apple.com/mac/app-store) (via
   [mas-cli](https://github.com/mas-cli/mas)).

Tap [homebrew-cask-versions](https://github.com/Homebrew/homebrew-cask-versions)
is added, too, in order to support installing
[visual-studio-code-insiders](https://github.com/Homebrew/homebrew-cask-versions/blob/master/Casks/visual-studio-code-insiders.rb).

### Managing installed dependencies

Once your system dependencies are installed, you should manage them with
`brew bundle` and `~/Brewfile`.

### Managing SSH keys with LastPass

You can store all your SSH keys in LastPass as
[secure notes](https://helpdesk.lastpass.com/secure-notes/) using note type "SSH
keys". The `extract-lastpass-secrets` scripts shows how to retrieve stored keys
while running Strap.

### Configuring macOS

On a new macOS system you normally have to make a number of manual
configurations using the System Preferences app. Many of these manual
configurations can be scripted. This is what script `macos` is for.

macOS comes with the `defaults` command-line utility with which you can change
many system preferences. The challenge with `defaults` is that you need to
figure out the domain and key arguments for each setting. Unfortunately, these
are not documented well. Here are some pointers:

- A good starting point is
  [Mathias Bynen's `.macos` configuration file](https://github.com/mathiasbynens/dotfiles/blob/master/.macos).
- For any security and privacy related confiugrations this
  [macOS Security and Privacy Guide](https://github.com/drduh/macOS-Security-and-Privacy-Guide)
  is a great read.
- The [defaults-write.com website](https://www.defaults-write.com/) contains a
  collection of command-line configurations.
- And last, but not least, if above docs do not have what you were looking for,
  you can try to reverse engineer the domains and keys required for the
  `defaults` command as described in
  [this article](https://pawelgrzybek.com/change-macos-user-preferences-via-command-line/).
