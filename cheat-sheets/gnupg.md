# gnupg cheat sheet

A highly opinionated GPG cheat sheet.

## Keys

* **generate key**: `gpg --full-generate-key`
    * _key type_: `1` (RSA and RSA)
    * _key size_: `4096`
    * _key expiration_: `0` (never expires)
    * _confirmation_: `y` (for **y**es)
    * _real name_: `Edmund Blackadder`
    * _email address_: `e.blackadder@cunning.plan.org`
    * _comment_: N/A
    * _confirmation_: `O` (for **O**kay)
    * _passphrase_: `******` (definitely not `1234561`)
* **list secret keys**: `gpg --list-secret-keys --keyid-format=long`

## Signed Git Commits

* **set signing key**: `git config --global user.signingkey <full fingerprint>`
* **enable signed commits**: `git config --global commit.gpgsign true`

## Backup keys

* **export public key**: `gpg --export --armor e.blackadder@cunning.plan.org > ~/gpg-public-key.asc`
* **export private key**: `gpg --export-secret-keys --armor e.blackadder@cunning.plan.org > ~/gpg-private-key.asc`
* **encrypt them**:
    * `gpg --symmetric --cipher-algo AES256 ~/gpg-public-key.asc`
    * `gpg --symmetric --cipher-algo AES256 ~/gpg-private-key.asc`
* _store encrypted copies in 2-3 different places: encrypted external drive + password manager + encrypted cloud storage._

## Import keys on device

* **decrypt them**:
    * `gpg --decrypt gpg-public-key.asc.gpg > gpg-public-key.asc`
    * `gpg --decrypt gpg-private-key.asc.gpg > gpg-private-key.asc`
* **import them**:
    * `gpg --import gpg-public-key.asc`
    * `gpg --import gpg-private-key.asc`
* **set ultimate trust**: `gpg --edit-key e.blackadder@cunning.plan.org`
    * `gpg> trust`
    * _trust_: `5` (ultimately)
    * _confirmation_: `y` (for **y**es)
    * `gpg> quit`

## Delete unencrypted keys

* **Ubuntu**: `shred -vfz -n 3 ~/gpg-private-key.asc ~/gpg-public-key.asc`
* **macOS**: `rm -P ~/gpg-private-key.asc ~/gpg-public-key.asc`
