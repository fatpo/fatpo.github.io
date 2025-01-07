```shell
#!/bin/bash

# 检查系统类型
OS_TYPE=$(uname)

# 1. 卸载现有 Zsh（如果有的话）
echo "卸载现有 Zsh..."
if [ "$OS_TYPE" == "Darwin" ]; then
  # MacOS 使用 Homebrew 卸载 Zsh
  if command -v brew &>/dev/null; then
    brew uninstall zsh
  else
    echo "未安装 Homebrew，跳过卸载。"
  fi
elif [ -f /etc/debian_version ]; then
  # Ubuntu/Debian 系统
  sudo apt remove --purge -y zsh
elif [ -f /etc/redhat-release ]; then
  # CentOS/RHEL 系统
  sudo yum remove -y zsh
elif [ -f /etc/arch-release ]; then
  # Arch 系统
  sudo pacman -R --noconfirm zsh
else
  echo "未知系统类型，跳过卸载。"
fi

# 2. 安装 Zsh
echo "安装 Zsh..."
if [ "$OS_TYPE" == "Darwin" ]; then
  # MacOS 使用 Homebrew 安装 Zsh
  if command -v brew &>/dev/null; then
    brew install zsh
  else
    echo "Homebrew 未安装，请先安装 Homebrew。"
  fi
elif [ -f /etc/debian_version ]; then
  # Ubuntu/Debian 系统
  sudo apt update
  sudo apt install -y zsh
elif [ -f /etc/redhat-release ]; then
  # CentOS/RHEL 系统
  sudo yum install -y zsh
elif [ -f /etc/arch-release ]; then
  # Arch 系统
  sudo pacman -S --noconfirm zsh
else
  echo "未知系统类型，跳过安装。"
fi

# 3. 设置 Zsh 为默认 Shell
echo "设置 Zsh 为默认 Shell..."
chsh -s $(which zsh)

# 4. 安装 Oh My Zsh
echo "安装 Oh My Zsh..."
if [ ! -d "$HOME/.oh-my-zsh" ]; then
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
else
  echo "Oh My Zsh 已经安装。"
fi

# 5. 安装 Zsh 插件：zsh-autosuggestions, zsh-syntax-highlighting, zsh-completions
echo "安装插件：zsh-autosuggestions, zsh-syntax-highlighting, zsh-completions..."

# zsh-autosuggestions（命令记忆）
if [ ! -d "$ZSH_CUSTOM/plugins/zsh-autosuggestions" ]; then
  git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
else
  echo "zsh-autosuggestions 插件已安装。"
fi

# zsh-syntax-highlighting（命令高亮）
if [ ! -d "$ZSH_CUSTOM/plugins/zsh-syntax-highlighting" ]; then
  git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
else
  echo "zsh-syntax-highlighting 插件已安装。"
fi

# zsh-completions（命令补全）
if [ ! -d "$ZSH_CUSTOM/plugins/zsh-completions" ]; then
  git clone https://github.com/zsh-users/zsh-completions.git $ZSH_CUSTOM/plugins/zsh-completions
else
  echo "zsh-completions 插件已安装。"
fi

# 6. 配置 .zshrc 文件启用插件
echo "配置 .zshrc 启用插件..."
if ! grep -q "plugins=(" $HOME/.zshrc; then
  echo 'plugins=(git zsh-autosuggestions zsh-syntax-highlighting zsh-completions)' >> $HOME/.zshrc
else
  # 如果已存在插件配置，则添加插件
  sed -i '/plugins=(/c\plugins=(git zsh-autosuggestions zsh-syntax-highlighting zsh-completions)' $HOME/.zshrc
fi

# 7. 启用命令补全和历史记忆
echo "启用命令补全和历史记忆功能..."
# 启用历史记忆
echo 'HISTFILE=~/.zsh_history' >> ~/.zshrc
echo 'HISTSIZE=10000' >> ~/.zshrc
echo 'SAVEHIST=10000' >> ~/.zshrc
echo 'setopt HIST_IGNORE_ALL_DUPS' >> ~/.zshrc
echo 'setopt HIST_FIND_NO_DUPS' >> ~/.zshrc
echo 'setopt SHARE_HISTORY' >> ~/.zshrc

# 启用自动补全
echo 'autoload -Uz compinit' >> ~/.zshrc
echo 'compinit' >> ~/.zshrc

# 启用命令补全增强功能
echo 'setopt AUTO_PARAM_SLASH' >> ~/.zshrc
echo 'setopt COMPLETE_IN_WORD' >> ~/.zshrc
echo 'setopt MENU_COMPLETE' >> ~/.zshrc

# 8. 重新加载 .zshrc 文件
echo "重新加载 .zshrc 配置..."
source ~/.zshrc

# 9. 输出完成信息
echo "Zsh、Oh My Zsh 和插件安装完成！"
echo "请关闭当前终端窗口并重新打开，或执行 'source ~/.zshrc' 以立即生效。"

```
