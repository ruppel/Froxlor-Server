# Froxlor Server With Ansible

This is an evolution from https://github.com/ruppel/SecureServerWithAnsible.

I was kind of disappointed from ISPConfig and wanted to move to froxlor.

By now, this is not finished!!! It's still under construction!

This is based on Ubuntu Server 20.04 (focal fossa) minimal installation.

# Setup your ansible controller machine

- Install ansible (minimum version 2.9).
  See `install-ansible-controller.sh` for simple script that installs ansible on ubuntu

# Setup ansible host

- Have a non-root user on the host, who is allowed to call `sudo`
  (On Ubuntu `adduser username`, give password and data and do a `usermod -aG sudo username`)
- Copy your public ssh key to file `.ssh/authorized_keys` to use Public/Private Key SSH Connection
- Do the ssh connection to the server (to check if it works and to get the fingerprint of the server)

# Setup variables

- Copy file `inventory_example.yml` to `myinventory.yml`
- Change values in `myinventory.yml` to your needs, especially `ansible_host`, `ansible_user`, `ansible_ssh_pass`, and `ansible_become_password`
- Change vars in file `myinventory.yml` to your needs

# Bootstrap ansible host

- `ansible-playbook -i myinventory.yml bootstrap.yml`
  This checks the ssh port and changes it if needed and installs needed packages for ansible.

# Check the ansible connection

- `ansible-playbook -i myinventory.yml pingtest.yml`

# Do the froxlor server setup

- `ansible-playbook -i myinventory.yml froxlor.yml`

# BY NOW, HERE IS THE END OF THE CURRENT IMPLEMENTATION! THERE WILL BE MORE IN FUTURE!!

# Use Let's Encrypt for ISPConfig

Now we follow the Tutorial from https://www.howtoforge.com/tutorial/securing-ispconfig-3-with-a-free-lets-encrypt-ssl-certificate/

Thanks to ahrasis!

- In browser go to `www.myserverdomainname.com:8080` (use the FQDN from your configuration!!) (froxlor.hostname)
- Login with name `admin`and the password you configured in your configuration (froxlor.admin_password)
- Go to "Sites" and add a new website with name `myserverdomainname.com` Be sure to activate `SSL` and `Let's Encrypt SSL` and `Auto-Subdomain` to `www.` and save the site
- Wait a minute to let Let's Encrypt do it's work
- Call `https://www.myserverdomainname.com` in your browser. The SSL connection should be secured by Lets Encrypt
- `ansible-playbook -i myinventory.yml use-lets-encrypt.yml`
- Create the incrontab entry on the server as given in the debug output
- restart apache on the server as given in the debug output

# Activate firewall (ufw) in the ISPConfig Web-UI

- Go to `www.myserverdomainname.com:8080` (use the FQDN from your configuration!!) and login in as admin (see above)
- Go to `System`->`Firewall`
- Click on `Add Firewall record`
- Choose your server
- Edit the TCP Ports. **WARNING Be sure to add your specified SSH Port**. You might want to delete some of the ports not used.
- Be sure that the state is `Active`
- Click on `Save`
- After a few seconds the firewall should be active

# Configure your servers and emails

- You should now configure your servers and email adresses using the ISPConfig Web-UI
- The mailadresses used in the `myinventory.yml` file should be present

# Hardening (and some other stuff) of the server

- The next step will do some hardening and if something is wrong might close your server for **any** ssh connection.
- So be sure to have an open ssh connection in parallel to your server to correct things. Open cennections will stay open, even if SSHD is restarted with some other configuration.
- Be sure to have set up your SSH for using public / private Key authentication!!
- Be sure to have a non-root user for SSH connections!!
- `ansible-playbook -i myinventory.yml server-hardening.yml`
