# bash_script_node_and_docker_at_launch

<br>

[ linux bash ] This script installs node and docker on an EC2 at launch

<br>

```
#!/bin/bash

# send script output to /tmp so we can debug boot failures
exec > /tmp/userdata.log 2>&1

# Update all packages
yum -y update

# Get latest cfn scripts; https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html#cfninit
yum install -y aws-cfn-bootstrap

# preparing ec2 host machine 
cat > /tmp/install_script.sh << EOF 
      # START
      echo "Setting up NodeJS Environment"
      curl https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
      # Dot source the files to ensure that variables are available within the current shell
      . /home/ec2-user/.nvm/nvm.sh
      . /home/ec2-user/.bashrc
      # Install NVM, NPM, Node.JS
      nvm alias default v12.7.0
      nvm install v12.7.0
      nvm use v12.7.0

      # Install git
      sudo yum install -y git
      
      # install docker 
      sudo amazon-linux-extras install docker
      sudo service docker start
      sudo usermod -a -G docker ec2-user
      sudo chkconfig docker on
      sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
      sudo chmod +x /usr/local/bin/docker-compose
      sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose      
      sudo reboot      
      # Create log directory
      mkdir -p /home/ec2-user/app/logs      
EOF

# Runs the install script as the ec2-user.
chown ec2-user:ec2-user /tmp/install_script.sh && chmod a+x /tmp/install_script.sh
sleep 1; su - ec2-user -c "/tmp/install_script.sh"
```
