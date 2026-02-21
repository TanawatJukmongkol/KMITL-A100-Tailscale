# Intro

This guide is for those who cannot connect to the server through VPN tunnel, and you want to use A100 outside of KMITL.

# Creating the account (on your computer):

1: Create a free tailscale account for your group. Skip the automatic setup.

2: Go to https://login.tailscale.com/admin/machines

3: Select "Add device" > "Linux server"

4: Set up the server (no tags, ephemeral, exit node, Auth key expiration: 15 days)

5: Generate install script (take note of --auth-key=<your auth key>, it will be used later).

6: Go to [admin page](https://login.tailscale.com/admin/machines), then name the machine as kmitl-a100 (optional, but will be usful when ssh with the domain name)

7: On the machine address drop down, copy  xxxxxxx.ts.net domain (this will be your ssh login hostname, instead of your IP)

# On the KMITL A100 server:

1: Go to the university, and connect to the KMITL's network (make sure that you're on the same LAN as the server).

2: Login to your group's server [listed here](
https://docs.google.com/spreadsheets/d/1jjBZNh3XVcz6CDOFnLqIepYEmnl2-u9oQt0ke3wbnrU/edit?gid=1149126098#gid=1149126098).

3: Add this to /etc/profile at the end:

```bash
if ! pgrep "tailscaled" >/dev/null 2>&1; then
  echo "Starting tailscale reverse proxy daemon..."
  nohup tailscaled \
    --tun=userspace-networking \
    --socks5-server=127.0.0.1:1055 \
    --state=/var/run/tailscale/tailscale.state \
    --socket=/var/run/tailscale/tailscaled.sock > /dev/null 2>&1 &
fi
```

then run,

```bash
curl -fsSL https://tailscale.com/install.sh | sh

source /etc/profile

tailscale up --auth-key=<your auth key> --advertise-exit-node --advertise-routes=10.100.16.0/24
```

Now, create a new user account:

```bash
# Create the default non-root user
adduser a100

# Adding the user to sudo (admin / root) group.
usermod -aG sudo a100

# To change the user's password (if you change your mind later)
passwd a100
```

You're done on the server. Go to your work laptop / computer.

# On your computer:

1: Download and install tailscale

2: Open up your terminal, and run

``` bash
tailscale login
```

follow the login link, and authenticate with the same account.

Done. You can now access the machine from anywhere, any time.

# How to SSH into the server:

```bash
ssh a100@kmitl-a100.tailxxxxxx.ts.net
```

# BONUS

To share with your teammates, go to admin page, and find ... on your A100 instance. Click "Share..." > Copy share link > Reusabe link to share with your colleague :D

You can also install [live share](https://www.youtube.com/watch?v=A2ceblXTBBc) extension, for easy live collaboration as well!

# Want HTTPS?

If you want HTTPS, you can proxy port by running this command on the server:

```bash
sudo tailscale serve --bg http://localhost:<port to proxy>
```

