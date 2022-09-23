# Adding a runner to Gitlab

### Install the runner

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

### Add the runners

{% hint style="warning" %}
Please ensure you have docker installed on your system.
{% endhint %}

Grab the registration variable from [https://gitlab.acumensoft.net/admin/runners](https://gitlab.acumensoft.net/admin/runners)

```
export REGISTRATION_TOKEN=<REPLACE_zVt9btQZk6bd5BiKC_KW>
```

Now you can add as many runners as you wish. Please select the docker runtime, not shell or any other. Then user docker:latest as image. Do not add any tags.

```
sudo gitlab-runner register --url https://gitlab.acumensoft.net/ --registration-token $REGISTRATION_TOKEN
```

After adding the runners you need to make one more config change. Open the file located at /etc/gitlab-runner/config.toml. Please ensure everything under \[\[runners.docker]] is the same, but leave everything else as is. Set the concurrency to the amount of runners that you added.

{% hint style="danger" %}
DO NOT COPY AND PASTE THE BELOW TO CREATE RUNNERS
{% endhint %}

```
concurrent = 3

[[runners]]
  name = "deepthought-docker-3"
  url = "https://gitlab.acumensoft.net/"
  token = "vadgMNgwrcqdXWMoC3hT"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "docker:latest"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
    shm_size = 0
```

You now have some runners installed on your system

Also add our proxy by creating a file in /etc/docker/daemon.json

```
{
    "insecure-registries": ["nexus.acumensoft.net:5051"],
        "registry-mirrors": ["http://nexus.acumensoft.net:5051"]
}

```
