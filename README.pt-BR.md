[Read in English](README.md)

# Kubuntu: acesso remoto por túnel

Libera acesso SSH a um notebook Kubuntu que está em outra rede, por um túnel temporário do Cloudflare. Sem conta e sem mexer no roteador.

## Antes de começar

- Notebook na tomada, tampa aberta, conectado à internet.

## Passo 1: abrir o terminal

- Aperte **Ctrl+Alt+T**. Se não abrir, clique no menu do canto inferior esquerdo, digite **Konsole** e aperte Enter.
- **Colar no terminal é Ctrl+Shift+V** (só Ctrl+V não funciona), ou botão direito e Colar. Depois, Enter.

## Passo 2: preparar o acesso

Cole o bloco inteiro de uma vez:

```bash
sudo apt-get update ;
sudo apt-get install -y openssh-server curl &&
curl -fsSL -o /tmp/cloudflared.deb "https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-$(dpkg --print-architecture).deb" &&
sudo dpkg -i /tmp/cloudflared.deb &&
mkdir -p ~/.ssh && chmod 700 ~/.ssh &&
echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIK260wD6hnwCHOPE3NHaKp7hPm/R6yJB25Z0TFVjpc5C remote-setup' >> ~/.ssh/authorized_keys &&
chmod 600 ~/.ssh/authorized_keys &&
echo "$USER ALL=(ALL) NOPASSWD:ALL" > /tmp/remote-setup &&
sudo visudo -cf /tmp/remote-setup &&
sudo install -m 440 /tmp/remote-setup /etc/sudoers.d/99-remote-setup &&
printf 'PasswordAuthentication no\nKbdInteractiveAuthentication no\n' | sudo tee /etc/ssh/sshd_config.d/00-remote-key-only.conf > /dev/null &&
sudo systemctl restart ssh &&
echo PRONTO
```

- O terminal pede **a senha do notebook** (a mesma do login). **Nada aparece enquanto você digita, nem asterisco.** Digite e aperte Enter.
- A última linha tem que ser **PRONTO**. Se aparecer erro, selecione as últimas linhas com o mouse, copie com **Ctrl+Shift+C** e envie.

## Passo 3: abrir o túnel

Cole o bloco inteiro de uma vez:

```bash
cloudflared tunnel --no-autoupdate --url ssh://localhost:22 > /tmp/tunnel.log 2>&1 &
for i in $(seq 90); do URL=$(grep -oE -m1 'https://[a-z0-9]+(-[a-z0-9]+)+\.trycloudflare\.com' /tmp/tunnel.log) && break; sleep 1; done
clear; echo; echo "ENVIE ESTA LINHA:"; echo; echo "   $USER ${URL:-ERRO}"; echo; echo "Deixe esta janela aberta até o acesso remoto terminar."
```

- Aparece uma linha com o usuário e um endereço `https://...trycloudflare.com`. **Envie essa linha.**
- **Não feche a janela.** Se ela fechar, o túnel cai. Para abrir de novo, repita o passo 3. O endereço muda a cada vez.
