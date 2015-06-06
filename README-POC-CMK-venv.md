# Configuration for for CMK

```
# Written by setup of check_mk 1.2.2p3 at Mon Jun  1 18:30:07 IST 2015
bindir='/opt/shinken-poc/shpoc/bin'
confdir='/opt/shinken-poc/shpoc/etc/check_mk'
sharedir='/opt/shinken-poc/shpoc/share/check_mk'
docdir='/opt/shinken-poc/shpoc/share/doc/check_mk'
checkmandir='/opt/shinken-poc/shpoc/share/doc/check_mk/checks'
vardir='/opt/shinken-poc/shpoc/var/lib/check_mk'
agentslibdir='/opt/shinken-poc/shpoc/lib/check_mk_agent'
agentsconfdir='/opt/shinken-poc/shpoc/etc/check_mk'
nagiosuser='shinken'
wwwuser='shinken'
wwwgroup='shinken'
nagios_binary='/opt/shinken-poc/shpoc/bin/shinken-arbiter'
nagios_config_file='/opt/shinken-poc/shpoc/etc/shinken/shinken.cfg'
nagconfdir='/opt/shinken-poc/shpoc/etc/shinken/resource.d'
nagios_startscript='/opt/shinken-poc/shpoc/etc/init.d/shinken'
nagpipe='/opt/shinken-poc/shpoc/var/lib/shinken/nagios.cmd'
check_result_path='/opt/shinken-poc/shpoc/var/spool/checkresults'
nagios_status_file='/opt/shinken-poc/shpoc/var/status.dat'
check_icmp_path='/opt/shinken-poc/shpoc/lib/nagios/plugins/check_icmp'
url_prefix='/'
apache_config_dir='/opt/shinken-poc/shpoc/etc/apache2/conf.d'
htpasswd_file='/opt/shinken-poc/shpoc/etc/htpasswd.users'
nagios_auth_name='Nagios Access'
pnptemplates='/opt/shinken-poc/shpoc/share/check_mk/pnp-templates'
enable_livestatus='no'
libdir='/opt/shinken-poc/shpoc/lib/check_mk'
livesock='/usr/local/nagios/var/rw/live'
livebackendsdir='/usr/share/check_mk/livestatus'
enable_mkeventd='no'
```

# Edit modules/check_mk.py

```
def do_check_nagiosconfig():
    if monitoring_core == 'nagios':
        command = nagios_binary + " -v -c "  + nagios_config_file + " 2>&1"
        sys.stdout.write("Validating Nagios configuration...")
        if opt_verbose:
            sys.stderr.write("Running '%s'" % command)
        sys.stderr.flush()

        process = os.popen(command, "r")
        output = process.read()
        exit_status = process.close()
        if not exit_status:
            sys.stdout.write(tty_ok + "\n")
            return True
        else:
            sys.stdout.write("ERROR:\n")
            sys.stderr.write(output)
            return False
    else:
        return True
```

# Install Nagios Plugins

folder `/opt/shinken-poc/shpoc/lib/nagios/`

# Edit CMK configuration

change the `check_submission to PIPE`
