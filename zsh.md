# Setting up zsh

### Install zsh and addons
```sh
yay -S zsh zsh-autosuggestions oh-my-zsh-git spaceship-prompt-git
```

### Link addons to oh-my-zsh framework
```sh
sudo ln -s /usr/share/zsh/plugins/zsh-autosuggestions /usr/share/oh-my-zsh/custom/plugins/zsh-autosuggestions
sudo ln -s /usr/lib/spaceship-prompt/ /usr/share/oh-my-zsh/custom/themes/spaceship-prompt
sudo ln -s spaceship-prompt/spaceship.zsh-theme /usr/share/oh-my-zsh/custom/themes/spaceship.zsh-theme
```

### Patch and link the oh-my-zsh zshrc
```sh
sudo patch /usr/share/oh-my-zsh/zshrc zshrc.patch
sudo ln -s /usr/share/oh-my-zsh/zshrc /etc/zsh/zshrc
```

### Change default shell to zsh
```sh
chsh -s /bin/zsh
```
