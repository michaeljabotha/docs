# Creating a backup

{% hint style="info" %}
All this info comes from:\
https://docs.gitlab.com/ee/raketasks/backup\_restore.html
{% endhint %}

## Creating a full backup

Seeing as we have installed from omnibus (APT/YUM/DNF) you can run the following to create a backup

```
sudo gitlab-backup create
```

The backup will now be stored in /var/opt/gitlab/backups/

OR YOU CAN RUN WITH SKIPPING REGISTRY AND ARTIFACTS FOR SPEEDY BACKUP AND RESTORE

```
gitlab-backup create SKIP=artifacts,registry GZIP_RSYNCABLE=yes
```
