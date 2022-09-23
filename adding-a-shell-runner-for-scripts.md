# Adding a shell runner for scripts

Install runners

```
# Download the binary for your system
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

# Give it permissions to execute
sudo chmod +x /usr/local/bin/gitlab-runner

# Create a GitLab CI user
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

# Install and run as service
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start
```

Add a runner

```
sudo gitlab-runner register --url https://gitlab.acumensoft.net/ --registration-token zVt9btQZk6bd5BiKC_KW --name $(hostname)_shell --executor shell --tag-list $(hostname) --output-limit 1048576
```

Set runner to run as root

```
sudo nano /etc/systemd/system/gitlab-runner.service
```

Should look something like this. Please note the user = root on line 10 and the addition of line 12 and 13

```
[Unit]
Description=GitLab Runner
ConditionFileIsExecutable=/usr/local/bin/gitlab-runner

After=syslog.target network.target

[Service]
StartLimitInterval=5
StartLimitBurst=10
ExecStart=/usr/local/bin/gitlab-runner "run" "--working-directory" "/home/gitlab-runner" "--config" "/etc/gitlab-runner/config.toml" "--service" "gitlab-runner" "--user" "root"

User=root
Group=root

Restart=always

RestartSec=120
EnvironmentFile=-/etc/sysconfig/gitlab-runner

[Install]
WantedBy=multi-user.target


```

Restart systemctl and the runners

```
sudo systemctl daemon-reload
sudo systemctl restart gitlab-runner
```

Go the runner on gitlab and restrict it to the project that you would like (Infrastructure\_scripts)
