=========================
Server Installation Guide
=========================

Contents:

.. toctree::
   :maxdepth: 2

----------------------------
Standard operating procedure
----------------------------

SSH
~~~
    
    Install and setup the ssh service, if not already installed.

    ::

        apt-get install openssh-server
        sed -i 's/Port 22/Port 222/g' /etc/ssh/sshd_config
        /etc/init.d/ssh restart
    
    .. warning::
       Log in in new terminal before closing existing session, in case you screw up.

Create user names
~~~~~~~~~~~~~~~~~
    
    ::

        adduser izak
        adduser rijk
        adduser roche

    Copy ssh keys and password hash for users other than yourself from another server, eg vera.

Configure the host name
~~~~~~~~~~~~~~~~~~~~~~~

    edit /etc/hostname # Set the hostname, without the domain part

    edit /etc/hosts, set host name like this example:

    ::

        178.63.44.5  eubusiness.upfronthosting.co.za eubusiness

    Set the hostname:
    
    ::

        hostname -F /etc/hostname

    Test resolution:
    
    ::

        hostname
        eubusiness
    
    ::

        hostname --fqdn
        eubusiness.upfronthosting.co.za

    Add it to your ssh config:

    edit $HOME/.ssh/config, add a stanza like this:
    
    ::

       Host eubusiness
           HostName 178.63.44.5
           port 222

    Configure DNS for it:

    Go to myaccount.hetzner.co.za, log in and add a dns record for the new host.

Firewall
~~~~~~~~
    
    .. note::
        This step is not required on a virtual server

    apt-get install firehol

    Run ‘ip addr ls’ and make sure the network interface is named eth0.

    Edit /etc/firehol/firehol.conf
    
    ::

        ---- snip ----
        version 5

        server_ussh_ports="tcp/222"
        client_ussh_ports="default"

        FIREHOL_LOG_MODE="ULOG"

        interface eth0 world
                # Allow ssh access on port 222
                server "ussh icmp" accept
                server ident reject with tcp-reset
                client all accept
        ---- snip ----

    Add http, smtp and others if you need them. Adapt network interface name if required.

    Run:
    
    ::

        firehol try
        Type: commit

    If all that works, edit /etc/default/firehol and set START_FIREHOL=YES.

Sudo
~~~~

    apt-get install sudo

    Add any special users to the sudo group, eg:
    
    ::

        adduser roche sudo
        adduser rijk sudo
        adduser izak sudo

    Log in as your own user, test that you can sudo. Then lock the root account:
    
    ::

        sudo passwd -l root

Other Packages
~~~~~~~~~~~~~~
    
    Other useful packages.

    ::

        apt-get update
        apt-get install vim less
        update-alternatives --config editor # Set to vim.basic
        apt-get install build-essential zlib1g-dev libjpeg62-dev python2.6 python2.6-dev
        apt-get install libssl-dev subversion git-core python-virtualenv libxslt-dev libpcre3-dev

Apache
~~~~~~
    
    Install Apache

    ::

        apt-get install apache2

    Edit /etc/apache2/ports.conf and change listening and Virtual hosting ports to 81.

    Edit /etc/apache2/sites-enabled/000-default and change port to 81 on default virtual host

    Add this towards the end of the Virtual Host config:
    
    ::

        Alias /awstats-icon/ /usr/share/awstats/icon/
        <Directory /usr/share/awstats/icon>
                Options None
                AllowOverride None
                Order allow,deny
                Allow from all
        </Directory>    

    Restart apache:
    
    ::
    
        apachectl graceful

Exim
~~~~

    If all you need is an outgoing mailserver,
    
    ::

        apt-get instal exim4-daemon-light

    For more heavy duty mail deliver, authentication, or spam checking:

    ::

        apt-get install exim4-daemon-heavy
        dpkg-reconfigure exim4-config

Dovecot
~~~~~~~
    
    Install

    ::

        apt-get install dovecot-imapd dovecot-pop3d
        dpkg-divert --local --rename /etc/dovecot/dovecot.conf

    Example configs: https://svn.upfronthosting.co.za/svn/configs/dovecot
    
    ::

        addgroup --gid 5000 vmail
        adduser --gid 5000 --uid 5000 --gecos 'Virtual mail' --disabled-login --no-create-home --home \
           /var/mail/vmail vmail
        mkdir -p /var/mail/vmail/eubusiness.com
        touch /etc/dovecot/passwd
        chmod 640 /etc/dovecot/passwd

    Create /etc/dovecot/passwd in the format:

    ::

        emailaddress@eubusiness.com:{PLAIN}thepassword

    Create mailboxes in /var/mail/vmail/eubusiness.com/username using maildirmake.dovecot, eg

    ::

        maildirmake.dovecot /var/mail/vmail/eubusiness.com/johndoe/Maildir
        chmod -R vmail:vmail /var/mail/vmail/eubusiness.com/johndoe

Spamassassin
~~~~~~~~~~~~

Mailman
~~~~~~~

Varnish
~~~~~~~

    Varnish that ships with squeeze is already too old. Get version 2.1.5 from our repo

    http://public.upfronthosting.co.za/debian/squeeze-amd64/
    
    You need both libvarnish and varnish.
    
    Install with:
    
    ::
    
        dpkg -i.

    Configurations for some of our machines are in svn here, this is a good starting point:
    https://svn.upfronthosting.co.za/svn/configs

    The file in svn named “default” replaces /etc/default/varnish.

    When installing files from svn, also move the .svn directory to /etc/varnish to make
    future updates as easy as “svn up”.
    
    Start it up: 

    ::

        /etc/init.d/varnish start

HAProxy
~~~~~~~
    
    Install

    ::

        apt-get install haproxy
        dpkg-divert --local --rename /etc/haproxy/haproxy.cfg # Divert future upgrades of haproxy.cfg

    Configurations are in svn: https://svn.upfronthosting.co.za/svn/configs/haproxy

    Edit /etc/default/haproxy and set ENABLED=1

    Start it up:
    
    ::
    
        /etc/init.d/haproxy start

Awstats
~~~~~~~
    
    Intall

    ::

        apt-get install awstats

    Example of configuration file that will work with varnish, put this in /etc/awstats:
    https://svn.upfronthosting.co.za/svn/configs/awstats

    Enable varnish-ncsa. Edit /etc/default/varnishncsa and uncomment: VARNISHNCSA_ENABLED=1

    Start varnish-ncsa:
    
    ::
    
        /etc/init.d/varnishncsa start

    Add www-data to varnishlog group, so awstats can read the log files:

    ::

        adduser www-data varnishlog

    Add an empty fake-rotated file so that awstats work immediately:

    ::

        install -m 644 -o varnishlog -g varnishlog /dev/null /var/log/varnish/varnishncsa.log.1

    Run awstats once to check that it works:

    ::

        sudo -u www-data /usr/lib/cgi-bin/awstats.pl -config=www.eubusiness.com -update

Munin
~~~~~
    
    Install:

    ::

        apt-get install munin munin-node

    If you do this last, most of the plugins are set up automatically.
    If for example you do this BEFORE installing varnish, you will have to manually add
    the varnish plugins.

Nagios
~~~~~

Munin HAProxy graphs plugin
~~~~~~~~~~~~~~~~~~~~~~~~~~~
    
    Install

    ::

        sudo apt-get install munin-extra-plugins
        cd /etc/munin/plugins/
        ln -s /usr/share/munin/plugins/haproxy_ haproxy
        /etc/init.d/munin restart

    En om te toets of die plugin daar is:
    
    ::

        "telnet localhost 4949"  en tik "list"

Backups
~~~~~~~


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

