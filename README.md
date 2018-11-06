Installation instructions
-------------------------

The following instructions should get you going to the extent that CLI  commands can be sent to the pipeline(s). The control VM installation starts on an empty Ubuntu 18.04 VM (http://releases.ubuntu.com/18.04/ubuntu-18.04.1-live-server-amd64.iso.
You may want to name the machine vbng-ctl, and as the salt pillar data uses this name (until you add more machines).

We decided to use saltstack (https://www.saltstack.com/) for provisioning, which requires a few initial steps to set up salt, but become helpful right after.

Add SaltStack Package Repo:

```bash
wget -O - https://repo.saltstack.com/apt/ubuntu/18.04/amd64/latest/SALTSTACK-GPG-KEY.pub | sudo apt-key add -
sudo sh -c 'echo deb http://repo.saltstack.com/apt/ubuntu/18.04/amd64/latest bionic main > /etc/apt/sources.list.d/saltstack.list'
sudo apt update
```

Be sure that the Universe repo is enabled:
```bash
sudo add-apt-repository universe
```

On the VM, install salt-master and salt-minion:

```bash
sudo apt install salt-master salt-minion
```

Clone the provisioning repos salt-top and salt-pillar: 

```bash
sudo git clone https://github.com/dpdk-vbng-cp/salt-top.git /srv/salt
sudo git clone https://github.com/dpdk-vbng-cp/salt-pillar.git /srv/pillar
```

Point `/etc/salt/master` to the top and pillar repos:

```bash
sudo perl -i -0pe 's/#file_roots:\n#  base:\n#    - \/srv\/salt\n#/file_roots:\n  base:\n    - \/srv\/salt\n/' /etc/salt/master
sudo perl -i -0pe 's/#pillar_roots:\n#  base:\n#    - \/srv\/pillar\n#/pillar_roots:\n  base:\n    - \/srv\/pillar\n/' /etc/salt/master
```

In `/etc/salt/minion` enable the *vbng-control* role:

```bash
sudo perl -i -0pe 's/#grains:\n#  roles:/grains:\n  roles:\n    - vbng-control/' /etc/salt/minion
```

Point the salt minion to the salt master at localhost:

```bash
sudo sed -ie 's/#master: salt/master: localhost/g' /etc/salt/minion
```

Restart the salt-master and salt-minion:

```bash
sudo systemctl restart salt-master salt-minion
```

Make sure the `/etc/salt/minion_id` starts with **vbng-ctl**, in a FQDN (e.g. *vbng-ctl.vbras.bisdn.de*). The master matches on *vbng-ctl** in the accepted key.

Let the local salt-master accept the key of the salt-minion:

```bash
sudo salt-key -A
```

And accept the following dialog: 
> The following keys are going to be accepted:
> Unaccepted Keys:
> **************************** (the minion name in /etc/salt/minion_id)
> Proceed? [n/Y] Y
> Key for minion **** accepted.

Apply the salt states to finally trigger the provisioning:

```bash
sudo  salt "*" state.apply
```
Wait for salt to finish the provisioning.

Change the file

/etc/default/redis-connector

to point to the correct IP address and port numbers for the ip_pipeline(s)

Theory of operation 
-------------------------------------
So far, our setup is the following:

A client machine (e.g., your own laptop) acts as ppp client. It has a VXLAN vtep with VNI 100 and a remote IP address of the CP virtual machine.
> sudo ip l add vx_ctl type vxlan vni 100 remote 172.16.248.9 dstport 8472
> sudo ip addr add 10.10.10.10/24 dev vx_ctl

The VXLAN tunnel ends on the control VM, and is directly consumed by accel-pppd. The VXLAN tunnel endpoint can be pulled in via regex into  accel-pppd as well, following this discussion:
https://wiki.mikbill.ru/billing/howto/accel_config_pppoe_ipoe_qinq 
(understanding Russian helps, but is not strictly needed)

On the control VM, the file /etc/accel-ppp.conf contains the following lines:

[redis]
host=127.0.0.1
port=6379
pubchan=accel-ppp
\#
\# select the event types to emit a message via redis
\#
\#ev_ses_starting=yes
\#ev_ses_finishing=yes
ev_ses_finished=yes
\#ev_ses_authorized=yes
\#ev_ctrl_starting=yes
\#ev_ctrl_started=yes
\#ev_ctrl_finished=yes
\#ev_ses_pre_up=yes
ev_ses_acct_start=yes
\#ev_config_reload=yes
\#ev_ses_auth_failed=yes
\#ev_ses_pre_finished=yes
\#ev_ip_changed=yes
\#ev_shaper=yes
\#ev_mppe_keys=yes
\#ev_dns=yes
\#ev_wins=yes
\#ev_force_interim_update=yes

These are the events of the internal PPP state machine of accel-pppd  that redis can subscribe to on the channel accel-ppp

We may uncomment more later.

Currently we are not yet using a RADIUS server, so the auth method is
set to 'auth-chap-md5', which means accel-pppd loads a file called

>/etc/ppp/chap-secrets

This contains username and password in the following way:
>intel * bng_admin * 

The script in

>/opt/bng-utils/dpdk-ip-pipeline-cli.py

then needs to be modified to add the CLI commands, I added for  instance in line 75:

tn['uplink'].write("pipeline PIPELINE0 port in 0 stats read clear")
tn['uplink'].read_all()
tn['downlink'].write("pipeline PIPELINE0 port in 0 stats read clear")
tn['downlink'].read_all()

When you have modified the addresses or the script itself, don't  forget to
> systemctl restart redis-connector

and if you want to observe the system messages
> journalctl -u redis-connector

I hope this gets you going.
