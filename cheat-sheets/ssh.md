# SSH cheat sheet

A highly opinionated SSH cheat sheet:

* generate SSH key: `ssh-keygen -t rsa -C <email>`
* add SSH key to ssh-agent: `eval "$(ssh-agent -s)"; ssh-add ~/.ssh/id_rsa`
* copy SSH key: `xclip -sel clip < ~/.ssh/id_rsa.pub`

Source: [Github Help: generating SSH keys](https://help.github.com/articles/generating-ssh-keys/).
