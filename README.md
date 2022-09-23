# Migrating Gitlab

## The Old Server

Stop services that run background tasks

```
sudo gitlab-ctl stop unicorn
sudo gitlab-ctl stop sidekiq
```

Create the backup

```bash
sudo gitlab-rake gitlab:backup:create
```

Create a folder to which you can move the newly created backup and config files.

```bash
mkdir ~/gitlab-old
sudo cp /etc/gitlab/gitlab.rb ~/gitlab-old
sudo cp /etc/gitlab/gitlab-secrets.json ~/gitlab-old
sudo cp -R /etc/gitlab/ssl ~/gitlab-old
```

Copy the latest backup from /var/opt/gitlab/backups/

```bash
sudo cp /var/opt/gitlab/backups/X_gitlab_backup.tar ~/gitlab-old
```

Move you folder to the new server via usb, scp or whatever else

## The New Server

{% hint style="warning" %}
Ensure that the new server is running the same version as you backed up file, otherwise it will not restore.
{% endhint %}

Move the config files into their respective locations

```bash
sudo cp ~/gitlab-old/gitlab* /etc/gitlab/
sudo cp -r ~/gitlab-old/ssl /etc/gitlab/
```

Run the gitlab reconfigure command to enure that the configs have taken effect.

```bash
sudo gitlab-ctl reconfigure
```

Stop background processes

```
sudo gitlab-ctl stop unicorn
sudo gitlab-ctl stop sidekiq
```

Copy the backup tar file to /var/opt/gitlab/backups/

```
sudo cp ~/gitlab-old/X_gitlab_backup.tar /var/opt/gitlab/backups/
sudo chown git:git /var/opt/gitlab/backups/X_gitlab_backup.tar 
```

Start the backup restore process

```
sudo gitlab-rake gitlab:backup:restore BACKUP=X
```

Restart gitlab

```
sudo gitlab-ctl start
sudo gitlab-rake gitlab:check SANITIZE=true

#you might need to run the following again
sudo gitlab-ctl start unicorn
sudo gitlab-ctl start sidekiq
```

{% hint style="info" %}
You might need to change permissions on the registry folder

```
chmod -R registry:registry /var/opt/gitlab/gitlab-rails/shared/registry
```
{% endhint %}
