```
sudo netstat -tulnp | grep ssh
sudo ss -tulnp | grep ssh
```

```
sudo systemctl status ssh
sudo systemctl enable ssh
```

Look for this line:

```
sudo nano /etc/ssh/sshd_config
# Port 22
```

Change it (or add if missing) to:

```
Port 2222
```

Save the file (CTRL+X, then Y, then Enter).

```
sudo systemctl restart sshd
```

```
sudo systemctl restart ssh
```

```
sudo ufw allow 2222/tcp
sudo ufw reload
```
