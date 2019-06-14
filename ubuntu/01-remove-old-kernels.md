# Remove old Linux kernels

```
sudo dpkg --list 'linux-image*' | awk '{ if ($1=="ii") print $2}' | grep -v `uname -r` | grep -v 'linux-image-generic' | grep -v 'linux-image-extra' | xargs sudo apt-get purge -y
sudo apt-get autoremove
sudo update-grub
```

Good to know:

* `uname -a` gives you the current kernel version
* `sudo dpkg --list 'linux-image*' | grep ii` gives you the list of installed kernels

Reference: [Safest way to clean up boot partition](https://gist.github.com/ipbastola/2760cfc28be62a5ee10036851c654600)
