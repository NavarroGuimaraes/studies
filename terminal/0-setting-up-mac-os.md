# Guia Definitivo: Setup Zsh + Oh My Zsh + Starship no macOS

Aposto que muita gente configurou o temrinal igual a Diego Fernandes (Rocketseat) no começo dos seus cursos de Node.JS,etc. Este guia detalha o passo a passo atualizado para configurar um ambiente de terminal similiar ao que ele usava, com dicas que fui acumulando pela internet e me fizeram aumentar a produtividade no dia a dia.

## 0. Pré-requisitos

- [**iTerm2**](<(https://iterm2.com)>) (opcional, pode usar o app terminal de sua preferencia).
- [**Homebrew**](https://brew.sh) instalado.
- **Git** instalado (rode o comando `xcode-select --install` no terminal).
- **Nerd Font instalada:** Baixe a [FiraCode Nerd Font](https://www.nerdfonts.com/font-downloads), instale no macOS e configure no iTerm2 em `Settings > Profiles > Text > Font`. Isso é obrigatório para os ícones do Starship funcionarem. A configuração pode ser levemente diferente caso você escolha outro terminal.

## 1. Instalar o Oh My Zsh

O macOS já vem com o Zsh por padrão nas versões mais recentes. Vamos instalar o framework Oh My Zsh para gerenciar as configurações.

No terminal, rode:

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

_(Se perguntar se deseja definir o Zsh como shell padrão, digite `Y`)._

## 2. Instalar o Starship (Prompt)

O Starship é um prompt ultrarrápido escrito em Rust que mostra o contexto do diretório atual (versão do Node, Java, branch do Git, etc).

1. Instale via Homebrew:

```bash
brew install starship
```

2. Mais tarde, vamos ativá-lo no arquivo `.zshrc`.

## 3. Instalar os Plugins Essenciais

Vamos clonar os repositórios do **zsh-autosuggestions** (sugestões baseadas no histórico) e **zsh-syntax-highlighting** (cores para comandos válidos/inválidos) direto para a pasta de plugins do Oh My Zsh.

Rode os dois comandos abaixo:

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

## 4. Configurar o arquivo `~/.zshrc`

Agora precisamos dizer ao Zsh para carregar tudo o que instalamos. Abra o arquivo de configuração no seu editor preferido:

```bash
nano ~/.zshrc
```

ou

```bash
code ~/.zshrc
```

Faça as seguintes alterações:

**A. Ativar os plugins:**
Encontre a linha que começa com `plugins=()` e substitua por:

```bash
plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
  sudo
  extract
  web-search
)
```

Um outro plugin que gosto bastante de usar é o [`zsh-interactive-cd`](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/zsh-interactive-cd). Ele requer o a instalação do [fzf](https://github.com/junegunn/fzf).

**B. Configurar o NVM (Node Version Manager):**
Para garantir que os comandos `node` e `npm` funcionem sempre que o terminal abrir (sem conflitos), adicione o seguinte bloco no final do arquivo:

```bash
# Configuração do NVM no Zsh
export NVM_DIR="$HOME/.nvm"
[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"
[ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"
```

**C. Inicializar o Starship e adicionar Aliases Úteis:**
Adicione isso logo após o bloco do NVM no final do arquivo:

```bash
# Aliases de produtividade
alias c="clear"
alias x="exit"
alias dev="npm run dev"
alias mvn-clean="mvn clean install"
alias spring-run="mvn spring-boot:run"

# Inicialização do Starship (deve ser a última linha)
eval "$(starship init zsh)"
```

Salve o arquivo (`Ctrl + O`, `Enter`, `Ctrl + X` no Nano) e reinicie o terminal.

---

## 5. Troubleshooting Simples (Resolução de Problemas)

Se algo der errado, não entre em pânico. Aqui estão os problemas mais comuns e como resolvê-los:

- **Problema:** Símbolos estranhos, caixinhas com interrogação ou fontes quebradas no terminal (`[?]`).
  - **Causa:** O iTerm2 não está usando uma Nerd Font.
  - **O que buscar no Google:** _"how to set nerd font in iterm2 mac"_
  - **Solução Rápida:** Certifique-se de baixar uma Nerd Font, instalar no Mac (catálogo de fontes) e selecioná-la nas configurações do iTerm2 (`Cmd + ,` > Profiles > Text > Font).

- **Problema:** Mensagem de erro _"zsh compinit: insecure directories"_.
  - **Causa:** Permissões incorretas em algumas pastas do Homebrew ou Zsh.
  - **O que buscar no Google:** _"zsh compinit insecure directories fix macbrew"_
  - **Solução Rápida:** Geralmente, rodar `chmod -R 755 /usr/local/share/zsh` ou `chmod -R 755 /opt/homebrew/share/zsh` resolve.

- **Problema:** Comando `node` ou `nvm` não encontrado ao abrir o terminal.
  - **Causa:** O caminho do NVM no arquivo `.zshrc` pode estar apontando para o lugar errado.
  - **O que buscar no Google:** _"nvm command not found zsh mac m1"_
  - **Solução Rápida:** Verifique se o caminho no `.zshrc` bate com o local de instalação do seu NVM (geralmente `/opt/homebrew/opt/nvm/nvm.sh` em Macs com Apple Silicon).

- **Problema:** Terminal ficou muito lento para abrir.
  - **Causa:** Excesso de plugins pesados ou problemas de rede com ferramentas como o `git` dentro de diretórios grandes.
  - **O que buscar no Google:** _"oh my zsh slow startup fix"_

---

Prontinho! Só jogar no Docs e formatar como achar melhor. Se precisar adicionar mais alguma etapa de alguma ferramenta específica do seu fluxo, é só falar!
