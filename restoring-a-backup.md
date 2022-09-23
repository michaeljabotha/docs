# Restoring a Backup

{% hint style="info" %}
All this info comes from:\
https://docs.gitlab.com/ee/raketasks/backup\_restore.html
{% endhint %}

## **Restoring** a full backup

Stop background processes

```
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
# Verify
sudo gitlab-ctl status
```

Start the restore process

```
# This command will overwrite the contents of your GitLab database!
sudo gitlab-backup restore BACKUP=X_X.X.X-ee
```

{% hint style="warning" %}
If this is a new server please remember to copy overwrite the following files:

```
/etc/gitlab/gitlab.rb
/etc/gitlab/gitlab-secrets.json

#then run
sudo gitlab-ctl reconfigure
```
{% endhint %}

Then run the following to finalize

```
sudo gitlab-ctl restart
sudo gitlab-rake gitlab:check SANITIZE=true
```

{% hint style="info" %}
You might need to change permissions on the registry folder

```
chmod -R registry:registry /var/opt/gitlab/gitlab-rails/shared/registry
```
{% endhint %}
