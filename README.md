# foreman_ansible
Steps you can follow to install foreman to be used with ansible without installing puppet. It also provides a tested and working copy of callback plugin file.
Disclaimer: I am not claiming that this is the best way to achieve this but this is a way which works. I also evaluated ARA but personally i like foreman better as it gives me more flexibility.

# Note:
This guide is tested with Foreman 1.15.1 and Ansible version 2.3.1. It can be used in both Ansible Push and Pull mode.

# How to install Foreman and Ansible without puppet ?
Execute following commands in the same sequence as root.
```
echo "deb http://deb.theforeman.org/ xenial 1.15" > /etc/apt/sources.list.d/foreman.list
echo "deb http://deb.theforeman.org/ plugins 1.15" >> /etc/apt/sources.list.d/foreman.list
apt-get -y install ca-certificates
wget -q https://deb.theforeman.org/pubkey.gpg -O- | apt-key add -
apt-get install software-properties-common
apt-add-repository ppa:ansible/ansible
apt-get update
apt-get install ansible
wget https://apt.puppetlabs.com/puppetlabs-release-pc1-xenial.deb
dpkg -i puppetlabs-release-pc1-xenial.deb
apt-get update && apt-get -y install foreman-installer
hostnamectl set-hostname <fqdn>
foreman-installer --puppet-server=false --foreman-proxy-puppet=false --foreman-proxy-puppetca=false --no-enable-puppet --enable-foreman-proxy-plugin-ansible
#Note: Apache won't start due to missing SSL ca-certificates
vi /etc/apache2/sites-enabled/05-foreman-ssl.conf [Correct the SSL configuration by deploying proper cert's and making sure configuration is correct]
service apache2 restart
```

# Integrate Ansible with Foreman
```
1. Copy the foreman1.py file at some path on your system and make a note of the path for using in later steps. I have added it to the repository itself and used this path later on "./lib/callback/foreman/".
2. Edit your ansible.cfg file either at /etc/ansible/ansible.cfg or under your repository root from where you will execute ansible playbooks.
Under default add following entries.
bin_ansible_callbacks = True
callback_whitelist = foreman1
callback_plugins   = <path from step 1>:/usr/lib/python2.7/dist-packages/ansible/plugins/callback/
3. Create a foreman user with following permissions.
view_config_reports
upload_config_reports
view_facts
upload_facts
view_hosts
create_hosts
edit_hosts
build_hosts
view_reports
upload_reports
4. Now make sure you have following variables exported and set for the user who will execute Ansible.
export FOREMAN_URL=<your foreman URL>
export FOREMAN_SSL_VERIFY=0
export FOREMAN_USER=<foreman username>
export FOREMAN_PASSWORD=<password for the user mentioned under foreman_user>
```

# You have successfuly installed and setup foreman to be used with Ansible runs.
1. You can use following under cron to remove all reports which has just skipped steps and no meaningful change to track.
```
foreman-rake reports:expire days=0 status=50331648
```
2. You can use following under cron to remove errored reports.
```
foreman-rake reports:expire days=14 status=4096
```
