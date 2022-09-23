# Adding a permission

This grants permission to every ip in the kubernetes cluster. If your IP range changes change here accordingly.

```
CREATE USER 'forcelink'@'10.0.0.0/255.0.0.0' IDENTIFIED BY 'f0rcel3nk';
CREATE USER 'forcelink_granter'@'10.0.0.0/255.0.0.0' IDENTIFIED BY 'f0rcel3nk';
CREATE USER 'forcelink_control'@'10.0.0.0/255.0.0.0' IDENTIFIED BY 'f0rcel3nk';
CREATE USER 'forcelink_readonly'@'10.0.0.0/255.0.0.0' IDENTIFIED BY 'f0rcel3nk';
GRANT ALL PRIVILEGES ON *.* TO 'forcelink'@'10.0.0.0/255.0.0.0';
GRANT ALL PRIVILEGES ON *.* TO 'forcelink_granter'@'10.0.0.0/255.0.0.0' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'forcelink_control'@'10.0.0.0/255.0.0.0';
GRANT SELECT ON *.* TO 'forcelink_readonly'@'10.0.0.0/255.0.0.0';
GRANT GRANT OPTION ON *.* TO 'forcelink_granter'@'10.0.0.0/255.0.0.0';

FLUSH PRIVILEGES;
```
