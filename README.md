[Leia em português](README.pt-BR.md)

# Kubuntu: remote access through a tunnel

Opens SSH access to a Kubuntu laptop on another network through a temporary Cloudflare tunnel. No account, no router changes.

## Before you start

- Laptop plugged in, lid open, connected to the internet.

## Step 1: open the terminal

- Press **Ctrl+Alt+T**. If nothing opens, click the menu in the bottom left corner, type **Konsole** and press Enter.
- **Paste in the terminal with Ctrl+Shift+V** (plain Ctrl+V does not work), or right click and Paste. Then press Enter.

## Step 2: prepare access

Paste the whole block at once:

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
echo DONE
```

- The terminal asks for **the laptop password** (the login one). **Nothing shows while you type, not even asterisks.** Type it and press Enter.
- The last line must be **DONE**. If an error shows up, select the last lines with the mouse, copy with **Ctrl+Shift+C** and send them.

## Step 3: open the tunnel

Paste the whole block at once:

```bash
cloudflared tunnel --no-autoupdate --url ssh://localhost:22 > /tmp/tunnel.log 2>&1 &
for i in $(seq 90); do URL=$(grep -oE -m1 'https://[a-z0-9]+(-[a-z0-9]+)+\.trycloudflare\.com' /tmp/tunnel.log) && break; sleep 1; done
clear; echo; echo "SEND THIS LINE:"; echo; echo "   $USER ${URL:-ERROR}"; echo; echo "Keep this window open until remote access is finished."
```

- A line shows up with the user name and a `https://...trycloudflare.com` address. **Send that line.**
- **Do not close the window.** If it closes, the tunnel drops. To reopen it, repeat step 3. The address changes every time.
