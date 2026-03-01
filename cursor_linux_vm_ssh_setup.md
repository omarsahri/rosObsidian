# Setting Up Cursor to Code on a Linux VM via SSH

Yes — the clean setup is:

1. **Enable SSH on the Linux VM**
2. **Make the VM reachable from your host**
3. **Create an SSH key on your main machine**
4. **Connect from Cursor**

## 1) Install SSH inside the Linux VM
On the VM:

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

Check the VM IP:

```bash
ip a
```

Look for something like `192.168.x.x` or `10.x.x.x`.

If the firewall is on:

```bash
sudo ufw allow OpenSSH
sudo ufw status
```

## 2) Make sure your host can reach the VM
This is the part that usually blocks people.

### If your VM is in Bridged Adapter mode
Your VM gets its own IP on the network.
That is the easiest setup.

Then from your host machine test:

```bash
ssh your_linux_username@VM_IP
```

### If your VM is in NAT mode
You usually need **port forwarding** from host port to guest port 22.

Example:
- Host port: `2222`
- Guest port: `22`

Then you connect with:

```bash
ssh -p 2222 your_linux_username@127.0.0.1
```

## 3) Create an SSH key on your host machine
On your **main machine** where Cursor runs:

```bash
ssh-keygen -t ed25519 -C "cursor-vm"
```

Press Enter through the defaults.

Then copy the key to the VM.

### If bridged:

```bash
ssh-copy-id your_linux_username@VM_IP
```

### If NAT with port 2222:

```bash
ssh-copy-id -p 2222 your_linux_username@127.0.0.1
```

If `ssh-copy-id` is unavailable, do it manually:

```bash
cat ~/.ssh/id_ed25519.pub
```

Then on the VM put that content into:

```bash
~/.ssh/authorized_keys
```

and fix permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## 4) Test SSH before using Cursor
From your host:

### Bridged

```bash
ssh your_linux_username@VM_IP
```

### NAT

```bash
ssh -p 2222 your_linux_username@127.0.0.1
```

If this works in the terminal, Cursor is much easier.

## 5) Add it to your SSH config
On your host machine, edit:

```bash
~/.ssh/config
```

Example for **bridged**:

```ssh
Host linux-vm
    HostName 192.168.1.50
    User omar
    IdentityFile ~/.ssh/id_ed25519
```

Example for **NAT + forwarded port**:

```ssh
Host linux-vm
    HostName 127.0.0.1
    Port 2222
    User omar
    IdentityFile ~/.ssh/id_ed25519
```

Then test:

```bash
ssh linux-vm
```

## 6) Connect from Cursor
In Cursor:
- Open the command palette
- Search for **Remote-SSH**
- Connect to host
- Choose `linux-vm`

If you hit extension problems, make sure you are using Cursor’s maintained Remote SSH extension.

## Minimal checklist
Use this exact order:

```bash
# on VM
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
ip a
```

```bash
# on host
ssh-keygen -t ed25519
ssh-copy-id your_linux_username@VM_IP
ssh your_linux_username@VM_IP
```

Then add `~/.ssh/config` and connect in Cursor.

## Simplest recommended path
For a local Linux VM on your machine, I recommend:

- **Bridged networking** if possible
- SSH key authentication
- Test in terminal first
- Then connect through Cursor
