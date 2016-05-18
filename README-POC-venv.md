# Proof of concept for installing Shinken with VirtualEnv
## Applies for LINUX systems only
### Tested for Ubuntu

1. `mkdir /opt/shinken-poc/`
2. `export SHINKEN-POC=/opt/shinken-poc/`
3. `cd $SHINKEN-POC`
4. `virtualenv shpoc`
5. `export SHPOCHOME=$SHINKEN-POC/shpoc`
6. `alias SHPOC='source $SHINKEN-POC/shpoc/bin/activate'`
7. SHPOC
8. `cd $SHPOCHOME`
9. `mkdir GITHUB`
10. `cd GITHUB`
11. `git clone https://github.com/naparuba/shinken.git`
12. `cd shinken`
13. `vim setup.py`
14. line: 284 (search for 'linux' in sys.platform)
```python
elif 'linux' in sys.platform or 'sunos5' in sys.platform:
    default_paths = {
        'bin':     install_scripts or "/usr/bin",
        'var':     "/var/lib/shinken/",
        'share':   "/var/lib/shinken/share",
        'etc':     "/etc/shinken",
        'run':     "/var/run/shinken",
        'log':     "/var/log/shinken",
        'libexec': "/var/lib/shinken/libexec",
        }
    data_files = [
        (
            os.path.join('/etc', 'init.d'),
            ['bin/init.d/shinken',
             'bin/init.d/shinken-arbiter',
             'bin/init.d/shinken-broker',
             'bin/init.d/shinken-receiver',
             'bin/init.d/shinken-poller',
             'bin/init.d/shinken-reactionner',
             'bin/init.d/shinken-scheduler',
             ]
            )
        ]

    if is_install:
        # warning: The default file will be generated a bit later
        data_files.append(
            (os.path.join('/etc', 'default',),
             ['build/bin/default/shinken']
             ))
```

Replace with: path to virtualenv : /opt/shinken-poc/shpoc

```python
elif 'linux' in sys.platform or 'sunos5' in sys.platform:
    default_paths = {
        'bin':     "/opt/shinken-poc/shpoc/bin",
        'var':     "/opt/shinken-poc/shpoc/var/lib/shinken/",
        'share':   "/opt/shinken-poc/shpoc/var/lib/shinken/share",
        'etc':     "/opt/shinken-poc/shpoc/etc/shinken",
        'run':     "/opt/shinken-poc/shpoc/var/run/shinken",
        'log':     "/opt/shinken-poc/shpoc/var/log/shinken",
        'libexec': "/opt/shinken-poc/shpoc/var/lib/shinken/libexec",
        }
    data_files = [
        (
            os.path.join('/opt/shinken-poc/shpoc/etc', 'init.d'),
            ['bin/init.d/shinken',
             'bin/init.d/shinken-arbiter',
             'bin/init.d/shinken-broker',
             'bin/init.d/shinken-receiver',
             'bin/init.d/shinken-poller',
             'bin/init.d/shinken-reactionner',
             'bin/init.d/shinken-scheduler',
             ]
            )
        ]

    if is_install:
        # warning: The default file will be generated a bit later
        data_files.append(
            (os.path.join('/opt/shinken-poc/shpoc/etc', 'default',),
             ['build/bin/default/shinken']
             ))
```

Line: 467 ( _chmodplusx function )

we would be linking 
/etc/init.d/shinken* files 
with 
`ln -s` 
with 
/opt/shinken-poc/shpoc/etc/init.d/shinken

>> Check for celery linking for multiple celery instances on same machine

```python
# if root is set, it's for package, so NO chown
if pwd and not root and is_install:
    # assume a posix system
    uid = get_uid(user)
    gid = get_gid(group)

    if uid is not None and gid is not None:
        # recursivly changing permissions for etc/shinken and var/lib/shinken
        for c in ['etc', 'run', 'log', 'var', 'libexec']:
            p = default_paths[c]
            recursive_chown(p, uid, gid, user, group)
        # Also change the rights of the shinken- scripts
        for s in scripts:
            bs = os.path.basename(s)
            recursive_chown(os.path.join(default_paths['bin'], bs), uid, gid, user, group)
            _chmodplusx( os.path.join(default_paths['bin'], bs) )
        _chmodplusx(default_paths['libexec'])

    # If not exists, won't raise an error there
    _chmodplusx('/etc/init.d/shinken')
    for d in ['scheduler', 'broker', 'receiver', 'reactionner', 'poller', 'arbiter']:
        _chmodplusx('/etc/init.d/shinken-'+d)
 ```

 Replace

```python
# if root is set, it's for package, so NO chown
if pwd and not root and is_install:
    # assume a posix system
    uid = get_uid(user)
    gid = get_gid(group)

    if uid is not None and gid is not None:
        # recursivly changing permissions for etc/shinken and var/lib/shinken
        for c in ['etc', 'run', 'log', 'var', 'libexec']:
            p = default_paths[c]
            recursive_chown(p, uid, gid, user, group)
        # Also change the rights of the shinken- scripts
        for s in scripts:
            bs = os.path.basename(s)
            recursive_chown(os.path.join(default_paths['bin'], bs), uid, gid, user, group)
            _chmodplusx( os.path.join(default_paths['bin'], bs) )
        _chmodplusx(default_paths['libexec'])

    # If not exists, won't raise an error there
    _chmodplusx('/opt/shinken-poc/shpoc/etc/init.d/shinken')
    for d in ['scheduler', 'broker', 'receiver', 'reactionner', 'poller', 'arbiter']:
        _chmodplusx('/opt/shinken-poc/shpoc/etc/init.d/shinken-'+d)
 ```

INI Files: Replace lines 368 forward to

```
if os.name != 'nt' and not is_update:
    for _file in daemonsini:
        inifile = _file
        outname = os.path.join('build', _file)
        # force the user setting as it's not set by default
        append_file_with(
            inifile, 
            outname, 
            """
            modules_dir={0}
            \n
            user={1}
            \n
            group={2}
            \n
            workdir={3}
            \n
            logdir={4}
            \n
            pidfile=%(workdir)s/brokerd.pid
            \n
            local_log=%(logdir)s/brokerd.log
            """.format (
                os.path.join(default_paths['var'], 'modules'),
                user, 
                group,
                default_paths['run'],
                default_paths['log']
                )
            )
        data_files.append( (os.path.join(default_paths['etc'], 'daemons'),
                            [outname]) )
```


# Start the installation

`python setup.py install --owner=<VE OWNER> --group=<VE GROUP>`

# Shinken INI and etc/init.d/shinken configuration

1. Shinken saerches for python<VERSION> 
2. `cd $SHPOCHOME/bin`
3. `ln -s python2.7 python` 

in setup.py the function

```python
def generate_default_shinken_file():
    # The default file must have good values for the directories:
    # etc, var and where to push scripts that launch the app.
    templatefile = "bin/default/shinken.in"
    build_base = 'build'
    outfile = os.path.join(build_base, "bin/default/shinken")

    #print('generating %s from %s', outfile, templatefile)

    mkpath(os.path.dirname(outfile))
    
    bin_path = default_paths['bin']

    # Read the template file
    f = open(templatefile)
    buf = f.read()
    f.close
    # substitute
    buf = buf.replace("$ETC$", default_paths['etc'])
    buf = buf.replace("$VAR$", default_paths['var'])
    buf = buf.replace("$RUN$", default_paths['run'])
    buf = buf.replace("$LOG$", default_paths['log'])
    buf = buf.replace("$SCRIPTS_BIN$", bin_path)
    # write out the new file
    f = open(outfile, "w")
    f.write(buf)
    f.close()
```

Takes care of the `shinken default configuration` in `etc/default/shinken`

```

## These vars will override the hardcoded ones in init script ##
ETC=$ETC$
VAR=$VAR$
BIN=$SCRIPTS_BIN$
RUN=$RUN$
LOG=$LOG$
```

would be replaced as 

```
## These vars will override the hardcoded ones in init script ##
ETC=/opt/shinken-poc/shpoc/etc/shinken
VAR=/opt/shinken-poc/shpoc/var/lib/shinken/
BIN=/opt/shinken-poc/shpoc/bin
RUN=/opt/shinken-poc/shpoc/var/run/shinken
LOG=/opt/shinken-poc/shpoc/var/log/shinken
```

Do verify

# Removing the shinken setup : the clean.sh

```shell
#!/bin/sh

sudo rm -fr /usr/local/lib/python2.*/dist-packages/Shinken-*-py2.6.egg
sudo rm -fr /usr/local/lib/python2.*/dist-packages/shinken
sudo rm -fr /usr/local/bin/shinken-*
sudo rm -fr /usr/bin/shinken-*
sudo rm -fr /etc/shinken
sudo rm -fr /etc/init.d/shinken*
sudo rm -fr /var/lib/shinken
sudo rm -fr /var/run/shinken
sudo rm -fr /var/log/shinken
sudo rm -fr /etc/default/shinken

sudo rm -fr build dist Shinken.egg-info
rm -fr test/var/*.pid
rm -fr var/*.debug
rm -fr var/archives/*
rm -fr var/*.log*
rm -fr var/*.pid
rm -fr var/service-perfdata
rm -fr var/*.dat
rm -fr var/*.profile
rm -fr var/*.cache
rm -fr var/rw/*cmd
#rm -fr /tmp/retention.dat
rm -fr /tmp/*debug
rm -fr test/tmp/livelogs*
rm -fr bin/default/shinken

# Then kill remaining processes
# first ask a easy kill, to let them close their sockets!
killall python2.6 2> /dev/null
killall python 2> /dev/null
killall /usr/bin/python 2> /dev/null

# I give them 2 sec to close
sleep 3

# Ok, now I'm really angry if there is still someboby alive :)
sudo killall -9 python2.6 2> /dev/null
sudo killall -9 python 2> /dev/null
sudo killall -9 /usr/bin/python 2> /dev/null

echo ""
```

Replace with 

```shell
#!/bin/sh
SPOCHOME=/opt/shinken-poc/shpoc
SPOCHPY=/opt/shinken-poc/shpoc/bin/python
sudo rm -fr $SPOCHOME/local/lib/python2.*/dist-packages/Shinken-*-py2.6.egg
sudo rm -fr $SPOCHOME/local/lib/python2.*/dist-packages/shinken
sudo rm -fr $SPOCHOME/local/bin/shinken-*
sudo rm -fr $SPOCHOME/bin/shinken-*
sudo rm -fr $SPOCHOME/etc/shinken
sudo rm -fr $SPOCHOME/etc/init.d/shinken*
sudo rm -fr $SPOCHOME/var/lib/shinken
sudo rm -fr $SPOCHOME/var/run/shinken
sudo rm -fr $SPOCHOME/var/log/shinken
sudo rm -fr $SPOCHOME/etc/default/shinken

sudo rm -fr build dist Shinken.egg-info
rm -fr test/var/*.pid
rm -fr var/*.debug
rm -fr var/archives/*
rm -fr var/*.log*
rm -fr var/*.pid
rm -fr var/service-perfdata
rm -fr var/*.dat
rm -fr var/*.profile
rm -fr var/*.cache
rm -fr var/rw/*cmd
#rm -fr /tmp/retention.dat
rm -fr /tmp/*debug
rm -fr test/tmp/livelogs*
rm -fr bin/default/shinken

# Then kill remaining processes
# first ask a easy kill, to let them close their sockets!
killall python2.6 2> /dev/null
killall python 2> /dev/null
killall /usr/bin/python 2> /dev/null
killall SPOCHPY 2> /dev/null

# I give them 2 sec to close
sleep 3

# Ok, now I'm really angry if there is still someboby alive :)
sudo killall -9 python2.6 2> /dev/null
sudo killall -9 python 2> /dev/null
sudo killall -9 /usr/bin/python 2> /dev/null
sudo killall SPOCHPY killall -9 2> /dev/null

echo ""
```

# Shinken INI file : .shinken.ini

1. create Shinken INI file in : /opt/shinken-poc/shpoc the SHPOCHOME

```ini
[paths]
# # set the paths according to your setup. defaults follow # the linux standard base
etc = /etc/shinken
lib = /var/lib/shinken
share = %(lib)s/share
cli = %(lib)s/cli
packs = %(etc)s/packs
modules = %(lib)s/modules
doc = %(lib)s/doc
inventory = %(lib)s/inventory
libexec = %(lib)s/libexec

[shinken.io]
# options for connection to the shinken.io website.  # proxy: curl style, should look as http://user:password@server:3128 # api_key: useful for publishing packages or earn xp after each install. create an account at http://shinken.io and go to http://shinken.io/~
proxy =
api_key =
```

Replace with 

```ini
[paths]
# # set the paths according to your setup. defaults follow # the linux standard base
etc = /opt/shinken-poc/shpoc/etc/shinken
lib = /opt/shinken-poc/shpoc/var/lib/shinken
share = %(lib)s/share
cli = %(lib)s/cli
packs = %(etc)s/packs
modules = %(lib)s/modules
doc = %(lib)s/doc
inventory = %(lib)s/inventory
libexec = %(lib)s/libexec

[shinken.io]
# options for connection to the shinken.io website.  # proxy: curl style, should look as http://user:password@server:3128 # api_key: useful for publishing packages or earn xp after each install. create an account at http://shinken.io and go to http://shinken.io/~
proxy =
api_key =
```

# Starting Shinken

1. cd $SPOCHOME
2. etc/init.d/shinken start

Will throw error

1. edit `etc/init.d/shinken`

```
## Default paths:
test "$BIN" || BIN=$(cd $curpath/.. && pwd)
test "$VAR" || VAR=$(cd $curpath/../../var && pwd)
test "$ETC" || ETC=$(cd $curpath/../../etc && pwd)
```

Put proper path to bin

```
## Default paths:
test "$BIN" || BIN=$(cd $curpath/../../bin && pwd)
test "$VAR" || VAR=$(cd $curpath/../../var && pwd)
test "$ETC" || ETC=$(cd $curpath/../../etc && pwd)
```

2. edit `etc/bin/shinken`

```python
if os.name != 'nt':
    DEFAULT_CFG = os.path.expanduser('~/.shinken.ini')
else:
    DEFAULT_CFG = 'c:\\shinken\\etc\\shinken.ini'
```

```python
if os.name != 'nt':
    DEFAULT_CFG = os.path.expanduser('/opt/shinken-poc/shpoc/.shinken.ini')
else:
    DEFAULT_CFG = 'c:\\shinken\\etc\\shinken.ini'
```

3. start : etc/init.d/shinken start

Fails

```
Reason 
[1433408303] ERROR: [Shinken] Your configuration is missing the path to the modules (modules_dir). I set it by default to /var/lib/shinken/modules. Please configure it

Reason
[1433408303] ERROR: [Shinken] [config] cannot open config file '<PATH TO VENV>/etc/shinken.cfg' for reading: [Errno 2] No such file or directory: '<PATH TO VENV>/etc/shinken.cfg'

Reason
[1433408303] ERROR: [Shinken] Cannot find python lib crypto: export to kernel.shinken.io isnot available

```

4. pip install pycrypto

5. edit `etc/init.d/` : `SHINKEN_DEFAULT_FILE="$ETC/shinken.cfg"`

6. 
