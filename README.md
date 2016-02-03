# fullstack-nanodegree-linux-server-configuration

## Create a new user named grader and grant this user sudo permissions.

  ```bash
$> adduser grader
$> usermod -a -G sudo grader
  ```

### Give ssh acccess to grader

Create a local key `grader_udacity_key.rsa`

  ```bash
$> ssh-keygen -t rsa
  ```

Upload the `grader_udacity_key.rsa.pub` to the server with the root key

  ```bash
$> cat ~/.ssh/grader_udacity_key.rsa | ssh -i ~/.ssh/udacity_key.rsa root@54.69.180.83 "mkdir /home/grader/.ssh && cat >> /home/grader/.ssh/authorized_keys"
  ```

**Logout from root account, then use the grader account**

  ```bash
$> exit
$> ssh -i ~/.ssh/grader_udacity_key.rsa grader@54.69.180.83
  ```
