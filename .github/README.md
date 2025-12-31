<div align="center">
    <img src="https://raw.githubusercontent.com/neovim/neovim.github.io/master/static/logos/neovim-logo-300x87.png">
</div>

<h1 align="center">nvim-docker</h1>

Run [Neovim](https://neovim.io/) using Docker—directly in your terminal or in your browser using [ttyd](https://github.com/tsl0922/ttyd).

This image is based on [Arch Linux](https://hub.docker.com/_/archlinux) and includes:
- [Zsh](https://www.zsh.org/) with [Oh My Zsh](https://ohmyz.sh/)
- [ttyd](https://github.com/tsl0922/ttyd) for web-based access, [built with NerdFont](https://github.com/lanjelin/nerd-ttyd)
- Easy first-run setup for popular Neovim distros

## Why Use nvim-docker?
- **No system dependencies**: Run Neovim without installing anything except Docker.
- **Supports most LSPs, DAPs, linters, and formatters**.
- **Built for UnRaid, but works on other Linux-based systems**.
- **Slim version available**: Use `ghcr.io/lanjelin/nvim-docker:slim` for a smaller image.

---

## 🚀 Installation

To persist your Neovim configuration, mount `/root` to a volume or a folder on the host.

### 🛠 Supported Neovim Distros

You can specify the `NVIM_DISTRO` environment variable on first run to install one of the following:
- [LazyVim](https://www.lazyvim.org/)
- [AstroNvim](https://astronvim.com/)
- [NvChad](https://nvchad.com/)
- [LunarVim](https://www.lunarvim.org/)
- [SpaceVim](https://spacevim.org/)
- [Kickstart](https://github.com/nvim-lua/kickstart.nvim)
- [NormalNvim](https://normalnvim.github.io/)

If `.config/nvim` exists, no changes will be made.  
Want another distro? Open an issue or pull request!

---

## ▶️ First Run

The container installs **Oh My Zsh** and your chosen Neovim distro on first launch.  
To initialize the setup, run:

```bash
docker run -it --rm \
  -v /mnt/user/appdata/nvim-docker:/root \
  -e NVIM_DISTRO="lazyvim" \
  ghcr.io/lanjelin/nvim-docker:latest
```

---

## 🖥 CLI Usage

To integrate `nvim-docker` as your `nvim` command, add this function to your `.bashrc` or `.zshrc`:

```bash
nvim () {
  if [ ! $# -eq 0 ]; then
    [[ -d "$1" ]] || [[ -f "$1" ]] || touch "$1"
    [[ -f "$1" ]] && NWD="$(dirname $1)" || NWD="$(realpath $1)"
    docker run -it --rm --name nvim-docker-cli-$(date +%s) \
    -w "/edit$NWD" \
    -v "$(realpath $1)":"/edit$(realpath $1)" \
    -v /mnt/user/appdata/nvim-docker:/root \
    ghcr.io/lanjelin/nvim-docker:latest \
    nvim "/edit$(realpath $1)"
  else
    docker run -it --rm --name nvim-docker-cli-$(date +%s) \
    -w "/root" \
    -v /mnt/user/appdata/nvim-docker:/root \
    ghcr.io/lanjelin/nvim-docker:latest \
    nvim
  fi
}
```
After adding this, reload your shell:
```bash
source ~/.zshrc  # or `source ~/.bashrc`
```

Or add this function to your `~/.config/fish/config.fish`:
```fish
function nvim
    if set -q argv[1]
        test -d "$argv[1]" || test -f "$argv[1]" || touch "$argv[1]"
        test -f "$argv[1]" && set NWD "$(dirname $argv[1])" || set NWD "$(realpath $argv[1])"
        docker run -it --rm --name nvim-docker-cli-$(date +%s) \
        -w "/edit$NWD" \
        -v "$(realpath $argv[1])":"/edit$(realpath $argv[1])" \
        -v /mnt/user/appdata/nvim-docker:/root \
        ghcr.io/lanjelin/nvim-docker:latest \
        nvim "/edit$(realpath $argv[1])"
    else
        docker run -it --rm --name nvim-docker-cli-$(date +%s) \
        -w "/root" \
        -v /mnt/user/appdata/nvim-docker:/root \
        ghcr.io/lanjelin/nvim-docker:latest \
        nvim
    end
end
```
After adding this, reload your shell:
```fish
source ~/.config/fish/config.fish
```

---

## 🛠 UnRaid Setup

If you've followed my guide for [ZSH and Oh-My-Zsh with persistent history](https://github.com/Lanjelin/unraid/tree/main/zsh-omz-persistent#zsh-and-oh-my-zsh-with-persistent-history), adding the above would work seamlessly. If not, there are a few ways to make this work.

### Preferred Method (User Scripts)

1. Download the function file:
```bash
wget -O /mnt/user/appdata/nvim-docker/.nvim-docker.rc https://raw.githubusercontent.com/Lanjelin/nvim-docker/main/.nvim-docker.rc
```
2. Create a new user script in UnRaid's User Scripts plugin.
3. Set it to **"At First Array Start Only"** and paste:
```bash
cat /mnt/user/appdata/nvim-docker/.nvim-docker.rc >> /root/.bash_profile
```

### Alternative Method (Boot-time Setup)

This will result in errors if you try to run `nvim` before the array is started.

1. Download the file:
```bash
wget -O /boot/config/.nvim-docker.rc https://raw.githubusercontent.com/Lanjelin/nvim-docker/main/.nvim-docker.rc
```
2. Modify the `go` file to load it on boot:
```bash
echo "cat /boot/config/.nvim-docker.rc >> /root/.bash_profile" >> /boot/config/go
```

---

## 🌐 Running Neovim in the Browser

This image includes **ttyd**, allowing you to run Neovim (or Zsh) in a web browser.

### UnRaid template

To quickly add this in UnRaid:
```bash
wget -O /boot/config/plugins/dockerMan/templates-user/my-nvim-docker.xml \
https://raw.githubusercontent.com/Lanjelin/docker-templates/main/lanjelin/nvim-docker.xml
```
Then go to **Docker → Add Container**, select `nvim-docker` from the **Template dropdown**.

### Docker
```bash
docker run -d --name nvim-docker \
  -w "/edit" \
  -p 7681:7681 \
  -v /path/to/project:/edit \
  -v /mnt/user/appdata/nvim-docker:/root \
  ghcr.io/lanjelin/nvim-docker:latest \
  ttyd.nerd -W -t fontFamily="JetBrains" nvim "/edit"
```
### Docker Compose
```yaml
services:
  nvim-docker:
    container_name: nvim-docker
    working_dir: /edit
    ports:
      - 7681:7681
    volumes:
      - /path/to/project:/edit
      - /mnt/user/appdata/nvim-docker:/root
    image: ghcr.io/lanjelin/nvim-docker:latest
    command: ttyd.nerd -W -t fontFamily="JetBrains" nvim "/edit"
```

### Zsh in the browser
For full shell access instead of Neovim:
```bash
docker run -d --name nvim-docker \
  -w "/edit" \
  -p 7681:7681 \
  -v /path/to/project:/edit \
  -v /mnt/user/appdata/nvim-docker:/root \
  ghcr.io/lanjelin/nvim-docker:latest \
  ttyd.nerd -W -t fontFamily="JetBrains" zsh
```

---

## 🔒 Security Considerations

This container runs as **root**, allowing it to modify files and directories it accesses.  
For security:
- **Do not expose it to the internet** without proper precautions.
- Consider running it in an isolated network.

---

## 📢 Contributions & Feedback

Want a feature or another Neovim distro?  
Feel free to open an [issue](https://github.com/Lanjelin/nvim-docker/issues) or submit a pull request!
