# Instructions to setup Ubuntu 16.04 Desktop to open ssh:// links

1. Create `~/.local/share/applications/ssh.desktop` with the following:
```bash
[Desktop Entry]
Version=1.0
Type=Application
Name=SSH Handler
Terminal=true
Exec=bash -c '(URL="%U" HOST="${URL#ssh://}"; ssh $HOST)'
Icon=utilities-terminal
StartupNotify=false
MimeType=x-scheme-handler/ssh;
```

2. Run the following command:
```bash
xdg-mime default ssh.desktop x-scheme-handler/ssh
```

`ssh://` links should now open up a terminal window.
