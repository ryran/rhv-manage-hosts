# rhv-manage-hosts

The `rhv-manage-hosts` script relies on `ovirt-shell` from the **ovirt-engine-cli** package (only tested with RHV 4.0) and can be run manually or as a systemd service. In the latter case, it can be run on any machine to block that machine's shutdown until certain actions are taken in RHV.

In the simplest case, it might only run on a RHV host to ensure that said host goes into maintenance mode before the OS shuts down.

It can also be run simultaneously on multiple hosts along with the RHV manager (and storage hosts) in order to prevent shutdowns until all hosts have gone into maintenance mode (not to mention, auto-activating things on OS startup).

Help page:

```
Usage: rhv-manage-hosts activate|deactivate|daemon
Automatically put RHV hosts into maintenance mode for graceful shutdowns
Automatically re-activate RHV hosts from maintenance mode for easy startups

activate:
  1) From a RHV host, activate only that host.
       [Or hosts listed space-separated in MANAGE_HOSTS environment variable]
     From RHVM or some other machine, activate all hosts.
       [Or hosts in MANAGE_HOSTS env var]
  2) Wait for all datacenters.
       [Or datacenters in MANAGE_DCS env var]
  3) Start all VMs.
       [Or VMs in MANAGE_VMS env var]
  
deactivate:
  1) From a RHV host, gracefully shutdown only VMs running on that host.
       [Or VMs in MANAGE_VMS]
     From RHVM or some other machine, gracefully shutdown all VMs.
       [Or VMs in MANAGE_VMS]
  2) From a RHV host, put only that host into maintenance mode.
       [Or hosts in MANAGE_HOSTS]
     From RHVM or some other machine, put all hosts into maintenance mode.
       [Or hosts in MANAGE_HOSTS]

daemon:
  1) Run "rguard -1" cmd to configure systemd to disable shutdown/reboot.
  2) Run "activate" [described above].
  3) Stay resident, waiting for systemd-logind to report "Power key pressed"
     journal event [as triggered by an ACPI shutdown].
  4) Run "deactivate" [described above].
  5) Run "rguard -0" cmd to unblock shutdown/reboot.
  
  The daemon mode requires reboot-guard (https://github.com/ryran/reboot-guard)
  to block and unblock shutdown, so ensure that is installed. Daemon mode is
  also meant to be run from a service manager like systemd. Take a look at
  /lib/systemd/system/rhv-manage-hosts.service & /etc/sysconfig/rhv-manage-hosts.

SETUP / ENVIRONMENT VARIABLES:
  * Set RHVM variable to FQDN of the RHVM; default: "rhvm.$(dnsdomainname)"
  * Set RHVM_USER variable as desired; default: "admin@internal"
  * Set RHVM_USER_PASSWORD variable or save password to: ~/.rhvm.passwd
  * To keep activate/deactivate from operating on any VMs, set: MANAGE_VMS=" "
    The same applies to the MANAGE_HOSTS and MANAGE_DCS variables.

Version info: rhv-manage-hosts v0.3 last mod 2017/04/12
See <https://github.com/ryran/rhv-manage-hosts> to report issues or contribute
```