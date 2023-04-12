# Simplify your workflow and streamline your development environment with ColimaBootstrapper

ColimaBootstrapper is a macOS automation script that simplifies your workflow and streamlines your development environment. 

It allows you to start Colima, a lightweight and easy-to-use Docker development environment, automatically on boot or at the start of your Mac laptop as a daemon. 



## Steps

1. Create an executable script to run in foreground and manage colima:

```bash
cat <<-EOF | sudo tee /usr/local/bin/colima-start-fg
#!/bin/bash

export PATH="/usr/local/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"

function shutdown() {
  colima stop
  exit 0
}

trap shutdown SIGTERM
trap shutdown SIGKILL
trap shutdown SIGINT

# wait until colima is running
while true; do
  colima status &>/dev/null
  if [[ \$? -eq 0 ]]; then
    break;
  fi

  colima start
  sleep 5
done

tail -f /dev/null &
wait \$!
EOF

sudo chmod +x /usr/local/bin/colima-start-fg
```

2. Create a launchd agent to run **colima** automatically:

```bash
cat > $HOME/Library/LaunchAgents/com.github.abiosoft.colima.plist <<-EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.github.abiosoft.colima</string>
    <key>Program</key>
    <string>/usr/local/bin/colima-start-fg</string>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <false/>
  </dict>
</plist>
EOF

launchctl load -w $HOME/Library/LaunchAgents/com.github.abiosoft.colima.plist
```
