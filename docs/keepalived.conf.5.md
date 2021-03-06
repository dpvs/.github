KEEPALIVED 5 "Apr 2016" Linux "File Formats Manual"
==========================================

NAME
----
keepalived.conf - configuration file for Keepalived

DESCRIPTION
----
`keepalived.conf` is the configuration file which describes all the Keepalived
keywords. Keywords are placed in hierarchies of blocks and subblocks, each layer
being delimited by '{' and '}' pairs.

Comments  start with  '#' or '!' to the end of the line and can start anywhere in a
line.

The keyword 'include' allows inclusion of other configuration files from within the
main configuration file.

PARAMETER SYNTAX
----
`<BOOL>` is one of on|off|true|false|yes|no

TOP HIERACHY
----
`GLOBAL CONFIGURATION`

`VRRPD CONFIGURATION`

`LVS CONFIGURATION`

GLOBAL CONFIGURATION
----
contains subblocks of `Global definitions`, `Static routes`, and `Static rules`

Global definitions
----
        global_defs      # Block id
        {
        notification_email    # To:
               {
               admin@example1.com
               ...
               }
        #  From:  fromaddress  that will be in the header (default keepalived@<local host name>)
        notification_email_from admin@example.com
        smtp_server 127.0.0.1 [<PORT>]
                                     # IP address or domain name
                                     # with optional port number (default 25)
        smtp_helo_name <HOST_NAME>   # name to use in HELO messages
                                     # defaults to local host name
        smtp_connect_timeout 30      # integer, seconds
        router_id my_hostname        # string identifying the machine,
                                     # (doesn't have to be hostname).
                                     # default: local host name
        vrrp_mcast_group4 224.0.0.18 # optional, default 224.0.0.18
        vrrp_mcast_group6 ff02::12   # optional, default ff02::12
        
        lvs_sync_daemon <INTERFACE> <VRRP_INSTANCE> [<SYNC_ID>]
                                     # Binding interface, vrrp instance and optional
                                     #syncid for lvs syncd
        
        # delay for second set of gratuitous ARPs after transition to MASTER
        vrrp_garp_master_delay 10    # seconds, default 5, 0 for no second set
        
        # number of gratuitous ARP messages to send at a time after transition to MASTER
        vrrp_garp_master_repeat 1    # default 5
        
        # delay for second set of gratuitous ARPs after lower priority advert received when MASTER
        vrrp_garp_lower_prio_delay 10
        
        #  number  of gratuitous ARP messages to send at a time after lower priority advert received when MASTER
        vrrp_garp_lower_prio_repeat 1
        
        # minimum time interval for refreshing gratuitous ARPs while MASTER
        vrrp_garp_master_refresh 60  # secs, default 0 (no refreshing)
        
        # number of gratuitous ARP messages to send at a time while MASTER
        vrrp_garp_master_refresh_repeat 2 # default 1
        
        # Delay in ms between gratuitous ARP messages sent on an interface
        vrrp_garp_interval 0.001  # decimal, seconds (resolution usecs). Default 0.
        
        # Delay in ms between unsolicited NA messages sent on an interface
        vrrp_gna_interval 0.000001  # decimal, seconds (resolution usecs). Default 0.
        
        # If a lower priority advert is received, don't send another advert. This causes
        # adherence to the RFCs. Defaults to false, unless strict_mode is set.
        vrrp_lower_prio_no_advert [<BOOL>]
        
        # Set the default VRRP version to use
        vrrp_version <2 or 3>     # default version 2
        
        # Specify the iptables chain for ensuring a version 3 instance
        # doesn't respond on addresses that it doesn't own.
        # Note: it is necessary for the specified chain to exist in
        # the iptables and/or ip6tables configuration, and for the chain
        # to be called from an appropriate point in the iptables configuration.
        # It will probably be necessary to have this filtering after accepting
        # any ESTABLISHED,RELATED packets, because IPv4 might select the VIP as
        # the source address for outgoing connections.
        vrrp_iptables keepalived     # default INPUT
        
        # or for outbound filtering as well
        # Note, outbound filtering won't work with IPv4, since the VIP can be  selected as the source address
        #  foran  outgoing connection. With IPv6 this is unlikely since the addresses are deprecated.
        vrrp_iptables keepalived_in keepalived_out
        
        # or to not add any iptables rules:
        vrrp_iptables
        
        # Keepalived may have the option to use ipsets in conjunction with iptables.
        # If so, then the ipset names can be specified, defaults as below.
        # If no names are specified, ipsets will not be used, otherwise any omitted
        # names will be constructed by adding "_if" and/or "6" to previously specified
        # names.
        vrrp_ipset [keepalived [keepalived_if [keepalived6 [keepalived_if6]]]]
        
        # The following enables checking that when in unicast mode, the source
        # address of a VRRP packet is one of our unicast peers.
        vrrp_check_unicast_src
        
        # Checking all the addresses in a received VRRP advert can be time consuming.
        # Setting this flag means the check won't be carried out if the advert is
        # from the same master router as the previous advert received.
        vrrp_skip_check_adv_addr     # Default - don't skip
        
        # Enforce strict VRRP protocol compliance. This will prohibit:
        #   0 VIPs
        #   unicast peers
        #   IPv6 addresses in VRRP version 2
        vrrp_strict
        
        # The following 4 options can be used if vrrp or checker processes
        #   are timing out. This can be seen by a backup vrrp instance becoming
        #   master even when the master is still running because the master or
        #   backup system is too busy to process vrrp packets.
        vrrp_priority <-20 to 19>    # Set the vrrp child process priority
             # Negative values increase priority.
        checker_priority <-20 to 19> # Set the checker child process priority
        vrrp_no_swap     # Set the vrrp child process non swappable
        checker_no_swap      # Set the checker child process non swappable
        
        # If Keepalived has been build with SNMP support, the following keywords are avail??? able
        # Note: Keepalived, checker and RFC support can be individually enabled/disabled
        snmp_socket  1.2.3.4:161   # specify socket to use for connecting to SNMP master agent (default 127.0.0.1:161)
        enable_snmp_keepalived     # enable SNMP handling of vrrp element  of  KEEPALIVED MIB
        enable_snmp_checker        # enable SNMP handling of checkerelement  of KEEPALIVED MIB
        enable_snmp_rfc            # enable SNMP handling of RFC2787 and  RFC6527  VRRP MIBs
        enable_snmp_rfcv2          # enable SNMP handling of RFC2787 VRRP MIB
        enable_snmp_rfcv3          # enable SNMP handling of RFC6527 VRRP MIB
        enable_traps               # enable SNMP traps
        }

       linkbeat_use_polling      # Poll to detect media link failure otherwise attempt to use ETHTOOL or MII interface



Static routes/addresses/rules
----

Keepalived can configure static addresses, routes, and rules.  These  addresses are
NOT moved by vrrpd, they stay on the machine.  If you already have IPs and routes on
your machines and your machines can ping each other, you don't need this section.

The syntax is the same as for virtual addresses and virtual routes.

   static_ipaddress  
   {  
   192.168.1.1/24 dev eth0 scope global  
   ...  
   }  
  
   static_routes  
   {  
   192.168.2.0/24 via 192.168.1.100 dev eth0  
   ...  
   }  
  
   static_rules  
   {  
   from 192.168.2.0/24 table 1  
   to 192.168.2.0/24 table 1  
   ...  
   }  

VRRPD CONFIGURATION
----

contains subblocks of VRRP script(s), VRRP synchronization group(s), VRRP gratuitous
ARP and unsolicited neighbour advert delay group(s) and VRRP instance(s)

VRRP script(s)
----
        # Adds a script to be executed periodically. Its exit code will be
        # recorded for all VRRP instances which are monitoring it with
        # non-zero weight.
        vrrp_script <SCRIPT_NAME> {
           script <STRING>|<QUOTED-STRING> # path of the script to execute
           interval <INTEGER>  # seconds between script invocations, default 1 second
           timeout <INTEGER>   # seconds after which script is considered to have failed
           weight <INTEGER:-254..254>  # adjust priority by this weight, default 2
           rise <INTEGER>        # required number of successes for OK transition
           fall <INTEGER>        # required number of successes for KO transition
        }

VRRP synchronization group(s)
----
        #string, name of group of IPs that failover together
        vrrp_sync_group VG_1 {
           group {
             inside_network   # name of the vrrp_instance (see below)
             outside_network  # One for each movable IP
             ...
           }
       
           # notify scripts and alerts are optional
           #
           # filenames of scripts to run on transitions
           # can be unquoted (if just filename)
           # or quoted (if it has parameters)
           # to MASTER transition
           notify_master /path/to_master.sh
           # to BACKUP transition
           notify_backup /path/to_backup.sh
           # FAULT transition
           notify_fault "/path/fault.sh VG_1"
       
           # for ANY state transition.
           # "notify" script is called AFTER the
           # notify_* script(s) and is executed
           # with 3 arguments provided by Keepalived
           # (so don't include parameters in the notify line).
           # arguments
           # $1 = "GROUP"|"INSTANCE"
           # $2 = name of the group or instance
           # $3 = target state of transition
           #  ("MASTER"|"BACKUP"|"FAULT")
           notify /path/notify.sh
       
           # Send email notification during state transition,
           # using addresses in global_defs above.
           smtp_alert
       
           global_tracking     # All VRRP share the same tracking config
        }


`VRRP gratuitous ARP and unsolicited neighbour advert delay group(s)`
specifies  the  setting of  delays  between sending gratuitous ARPs and unsolicited
neighbour advertisements. This is intended for when an upstream switch is unable  to
handle being flooded with ARPs/NAs.

Use  interface  when  the limits apply on the single physical interface.  Use inter???
faces when a group of interfaces are linked to the same switch and the limits  apply
to the switch as a whole.


If  the global vrrp_garp_interval and/or vrrp_gna_interval are set, any interfaces
that aren't specified in a garp_group will inherit the global settings.

       garp_group {
       # Sets the interval between Gratuitous ARP (in seconds, resolution microseconds) garp_interval <DECIMAL>
       # Sets the default interval between  unsolicited  NA  (in  seconds,  resolution microseconds)
       gna_interval <DECIMAL>
       # The physical interface to which the intervals apply
       interface <STRING>
       # A list of interfaces accross which the delays are aggregated.
       interfaces {
           <STRING>
           <STRING>
           ...
           }
       }

VRRP instance(s)
----
describes  the movable IP for each instance of a group in vrrp_sync_group.  Here are
described two IPs (on inside_network and on outside_network), on  machine  "my_host???
name",  which  belong  to  the  group VG_1 and which will transition together on any
state change.

        #You will need to write another block for outside_network.
        vrrp_instance inside_network {
           # Initial state, MASTER|BACKUP
           # As soon as the other machine(s) come up,
           # an election will be held and the machine
           # with the highest priority will become MASTER.
           # So the entry here doesn't matter a whole lot.
           state MASTER
       
           # interface for inside_network, bound by vrrp
           interface eth0
       
           # Use VRRP Virtual MAC.
           use_vmac [<VMAC_INTERFACE>]
       
           # Send/Recv VRRP messages from base interface instead of
           # VMAC interface
           vmac_xmit_base
       
           native_ipv6        # force instance to use IPv6 (when mixed IPv4 and IPv6  con???
              fig).
       
           # Ignore VRRP interface faults (default unset)
           dont_track_primary
       
           # optional, monitor these as well.
           # go to FAULT state if any of these go down.
           track_interface {
             eth0
             eth1
             eth2 weight <-254..254>
             ...
           }
       
           #  add  a  tracking script to  the interface (<SCRIPT_NAME> is the name of the
              vrrp_script entry)
           track_script {
               <SCRIPT_NAME>
               <SCRIPT_NAME> weight <-254..254>
           }
       
           # default IP for binding vrrpd is the primary IP
           # on interface. If you want to hide the location of vrrpd,
           # use this IP as src_addr for multicast or unicast vrrp
           # packets. (since it's multicast, vrrpd will get the reply
           # packet no matter what src_addr is used).
           # optional
           mcast_src_ip <IPADDR>
           unicast_src_ip <IPADDR>
       
           version <2 or 3>        # VRRP version to run on interface
                  #  default is global parameter vrrp_version.
       
           # Do not send VRRP adverts over a VRRP multicast group.
           # Instead it sends adverts to the following list of
           # ip addresses using unicast. It can be cool to use
           # the VRRP FSM and features in a networking
           # environment where multicast is not supported!
           # IP addresses specified can be IPv4 as well as IPv6.
           unicast_peer {
             <IPADDR>
             ...
           }
       
           # interface specific settings, same as  global  parameters; default  to  global
              parameters
           garp_master_delay 10
           garp_master_repeat 1
           garp_lower_prio_delay 10
           garp_lower_prio_repeat 1
           garp_master_refresh 60
           garp_master_refresh_repeat 2
           garp_interval 100
           gna_interval 100
       
           lower_prio_no_advert [<BOOL>]
       
           # arbitrary unique number from 0 to 255
           # used to differentiate multiple instances of vrrpd
           # running on the same NIC (and hence same socket).
           virtual_router_id 51
       
           # for electing MASTER, highest priority wins.
           # to be MASTER, make this 50 more than on other machines.
           priority 100
       
           # VRRP Advert interval in seconds (e.g. 0.92) (use default)
           advert_int 1
       
           #  Note:  authentication was removed from the VRRPv2 specification by RFC3768 in
              2004.
           #   Use of this option is non-compliant and can cause problems; avoid  using  if
              possible,
           #   except when using unicast, where it can be helpful.
           authentication { # Authentication block
               # PASS||AH
               # PASS - Simple password (suggested)
               # AH - IPSEC (not recommended))
               auth_type PASS
               # Password for accessing vrrpd.
               # should be the same on all machines.
               # Only the first eight (8) characters are used.
               auth_pass 1234
           }
       
           #addresses add|del on change to MASTER, to BACKUP.
           #With the same entries on other machines,
           #the opposite transition will be occurring.
           virtual_ipaddress {
               <IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPE> label <LABEL>
               192.168.200.17/24 dev eth1
               192.168.200.18/24 dev eth2 label eth2:1
           }
       
           #VRRP IP excluded from VRRP
           #optional.
           #For cases with large numbers (eg 200) of IPs
           #on the same interface. To decrease the number
           #of packets sent in adverts, you can exclude
           #most IPs from adverts.
           #The IPs are add|del as for virtual_ipaddress.
           # Can also be used if you want to be able to add
           # a mixture of IPv4 and IPv6 addresses, since all
           # addresses in virtual_ipaddress must be of the
           # same family.
           virtual_ipaddress_excluded {
            <IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPE>
            <IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPE>
               ...
           }
           # routes add|del when changing to MASTER, to BACKUP
           virtual_routes {
               #  src  <IPADDR>  [to]  <IPADDR>/<MASK> via|gw <IPADDR>  [or <IPADDR>] dev
              <STRING> scope <SCOPE> table <TABLE>
               src 192.168.100.1 to 192.168.109.0/24 via 192.168.200.254 dev eth1
               192.168.110.0/24 via 192.168.200.254 dev eth1
               192.168.111.0/24 dev eth2
               192.168.112.0/24 via 192.168.100.254
               192.168.113.0/24 via 192.168.200.254 or 192.168.100.254 dev eth1
               blackhole 192.168.114.0/24
               0.0.0.0/0 gw 192.168.0.1 table 100  # To set a default  gateway into  table
              100.
           }
       
           # rules add|del when changing to MASTER, to BACKUP
           virtual_rules {
               from 192.168.2.0/24 table 1
               to 192.168.2.0/24 table 1
           }
       
           accept    # Allow the non-master owner to process the packets destined to VIP
       
           # VRRP will normally preempt a lower priority
           # machine when a higher priority machine comes
           # online.  "nopreempt" allows the lower priority
           # machine to maintain the master role, even when
           # a higher priority machine comes back online.
           # NOTE: For this to work, the initial state of this
           # entry must be BACKUP.
           nopreempt
           preempt        # for backwards compatibility
       
           # See description of global vrrp_skip_check_adv_addr, which
           # sets the default value. Defaults to vrrp_skip_check_adv_addr
           skip_check_adv_addr [on|off|true|false|yes|no]  #  Default  on  if no word
              specified
       
           # See description of global vrrp_strict
           # If vrrp_strict is not specified, it takes the value of vrrp_strict
           # If strict_mode without a parameter is specified, it defaults to on
           strict_mode [on|off|true|false|yes|no]
       
           # Seconds after startup or seeing a lower priority master until preemption
           # (if not disabled by "nopreempt").
           # Range: 0 (default) to 1000
           # NOTE: For this to work, the initial state of this
           # entry must be BACKUP.
           preempt_delay 300 # waits 5 minutes
       
           # Debug level, not implemented yet.
           debug <LEVEL> # LEVEL is a number in the range 0 to 4
       
           # notify scripts, alert as above
           notify_master <STRING>|<QUOTED-STRING>
           notify_backup <STRING>|<QUOTED-STRING>
           notify_fault <STRING>|<QUOTED-STRING>
           notify_stop <STRING>|<QUOTED-STRING>      # executed when stopping vrrp
           notify <STRING>|<QUOTED-STRING>
           smtp_alert
        }
       
        # Parameters used for SSL_GET check.
        # If none of the parameters are specified, the SSL context will be auto generated.
        SSL {
           password <STRING>   # password
           ca <STRING>        # ca file
           certificate <STRING>  # certificate file
           key <STRING>        # key file
        }


LVS CONFIGURATION
----
contains subblocks of Virtual server group(s) and Virtual server(s)

The subblocks contain arguments for ipvsadm(8). Knowledge  of  ipvsadm(8)  will  be
helpful here.

Virtual server group(s)
----
        # optional
        # this groups allows a service on a real_server
        # to belong to multiple virtual services
        # and to only be health checked once.
        # Only for very large LVSs.
        virtual_server_group <STRING> {
               #VIP port
               <IPADDR> <PORT>
               <IPADDR> <PORT>
               ...
               #
               # <IPADDR RANGE> has the form
               # XXX.YYY.ZZZ.WWW-VVV eg 192.168.200.1-10
               # range includes both .1 and .10 address
               <IPADDR RANGE> <PORT># VIP range VPORT
               <IPADDR RANGE> <PORT>
               ...
               fwmark <INT>  # fwmark
               fwmark <INT>
               ...  }


Virtual server(s)
----
A virtual_server can be a declaration of one of

        vip vport (IPADDR PORT pair)

        fwmark <INT>

        (virtual server) group <STRING>

        #setup service
        virtual_server IP port |
        virtual_server fwmark int |
        virtual_server group string
        {
        # delay timer for service polling
        delay_loop <INT>

        # LVS scheduler
        lb_algo rr|wrr|lc|wlc|lblc|sh|dh
        # Enable One-Packet-Scheduling for UDP (-O in ipvsadm)
        ops
        # LVS forwarding method
        lb_kind NAT|DR|TUN
        # LVS persistence engine name
        persistence_engine <STRING>
        # LVS persistence timeout in seconds
        persistence_timeout <INT>
        # LVS granularity mask (-M in ipvsadm)
        persistence_granularity <NETMASK>
        # L4 protocol
        protocol TCP|UDP|SCTP
        # If VS IP address is not set,
        # suspend healthchecker's activity
        ha_suspend

        lvs_sched   # synonym for lb_algo
        lvs_method  # synonym for lb_kind

        # VirtualHost string for HTTP_GET or SSL_GET
        # eg virtualhost www.firewall.loc
        virtualhost <STRING>

        # On daemon startup assume that all RSs are down
        # and healthchecks failed. This helps to prevent
        # false positives on startup. Alpha mode is
        # disabled by default.
        alpha

        # On daemon shutdown consider quorum and RS
        # down notifiers for execution, where appropriate.
        # Omega mode is disabled by default.
        omega

        # Minimum total weight of all live servers in
        # the pool necessary to operate VS with no
        # quality regression. Defaults to 1.
        quorum <INT>

        # Tolerate this much weight units compared to the
        # nominal quorum, when considering quorum gain
        # or loss. A flap dampener. Defaults to 0.
        hysteresis <INT>

        # Script to execute when quorum is gained.
        quorum_up <STRING>|<QUOTED-STRING>

        # Script to execute when quorum is lost.
        quorum_down <STRING>|<QUOTED-STRING>

        # IP family for a fwmark service (optional)
        ip_family inet|inet6

	# virtual service privated establish state timeout.
	est_timeout <INT>

        # setup realserver(s)

        # RS to add when all realservers are down
        sorry_server <IPADDR> <PORT>
        # applies inhibit_on_failure behaviour to the
        # preceding sorry_server directive
        sorry_server_inhibit

        # one entry for each realserver
        real_server <IPADDR> <PORT>
           {
        # relative weight to use, default: 1
        weight <INT>
        # Set weight to 0 when healthchecker detects failure
        inhibit_on_failure

        # Script to execute when healthchecker
        # considers service as up.
        notify_up <STRING>|<QUOTED-STRING>
        # Script to execute when healthchecker
        # considers service as down.
        notify_down <STRING>|<QUOTED-STRING>

        uthreshold <INTEGER> # maximum number of connections to server
        lthreshold <INTEGER> # minimum number of connections to server

        # pick one healthchecker
        # HTTP_GET|SSL_GET|TCP_CHECK|SMTP_CHECK|MISC_CHECK

        # HTTP and SSL healthcheckers
        HTTP_GET|SSL_GET
        {
            # An url to test
            # can have multiple entries here
            url {
              #eg path / , or path /mrtg2/
              path <STRING>
              # healthcheck needs status_code
              # or status_code and digest
              # Digest computed with genhash
              # eg digest 9b3a0c85a887a256d6939da88aabd8cd
              digest <STRING>
              # status code returned in the HTTP header
              # eg status_code 200
              status_code <INT>
            }
            # number of get retries
            nb_get_retry <INT>
            # delay before retry
            delay_before_retry <INT>

            # ======== generic connection options
            # Optional IP address to connect to.
            # The default is the realserver IP
            connect_ip <IP ADDRESS>
            # Optional port to connect to
            # The default is the realserver port
            connect_port <PORT>
            # Optional interface to use to
            # originate the connection
            bindto <IP ADDRESS>
            # Optional source port to
            # originate the connection from
            bind_port <PORT>
            # Optional connection timeout in seconds.
            # The default is 5 seconds
            connect_timeout <INTEGER>
            # Optional fwmark to mark all outgoing
            # checker packets with
            fwmark <INTEGER>

            # Optional random delay to start the initial check
            # for maximum N seconds.
            # Useful to scatter multiple simultaneous
            # checks to the same RS. Enabled by default, with
            # the maximum at delay_loop. Specify 0 to disable
            warmup <INT>
        } #HTTP_GET|SSL_GET

        # TCP healthchecker
        TCP_CHECK
        {
            # ======== generic connection options
            # Optional IP address to connect to.
            # The default is the realserver IP
            connect_ip <IP ADDRESS>
            # Optional port to connect to
            # The default is the realserver port
            connect_port <PORT>
            # Optional interface to use to
            # originate the connection
            bindto <IP ADDRESS>
            # Optional source port to
            # originate the connection from
            bind_port <PORT>
            # Optional connection timeout in seconds.
            # The default is 5 seconds
            connect_timeout <INTEGER>
            # Optional fwmark to mark all outgoing
            # checker packets with
            fwmark <INTEGER>

            # Optional random delay to start the initial check
            # for maximum N seconds.
            # Useful to scatter multiple simultaneous
            # checks to the same RS. Enabled by default, with
            # the maximum at delay_loop. Specify 0 to disable
            warmup <INT>
            # Retry count to make additional checks if check
            # of an alive server fails. Default: 1
            retry <INT>
            # Delay in seconds before retrying. Default: 1
            delay_before_retry <INT>
        } #TCP_CHECK

        # SMTP healthchecker
        SMTP_CHECK
        {
            # ======== generic connection options
            # Optional IP address to connect to.
            # The default is the realserver IP
            connect_ip <IP ADDRESS>
            # Optional port to connect to
            # the default is port 25
            connect_port <PORT>
            # Optional interface to use to
            # originate the connection
            bindto <IP ADDRESS>
            # Optional source port to
            # originate the connection from
            bind_port <PORT>
            # Optional per-host connection timeout.
            # Default is outer-scope connect_timeout
            connect_timeout <INTEGER>
            # Optional fwmark to mark all outgoing
            # checker packets with
            fwmark <INTEGER>

            # An optional host interface to check.
            # If no host directives are present, only
            # the IP address of the realserver will
            # be checked.
            host {
              # ======== generic connection options
              # Optional IP address to connect to.
              # The default is the realserver IP
              connect_ip <IP ADDRESS>
              # Optional port to connect to
              # the default is port 25
              connect_port <PORT>
              # Optional interface to use to
              # originate the connection
              bindto <IP ADDRESS>
              # Optional source port to
              # originate the connection from
              bind_port <PORT>
              # Optional per-host connection timeout.
              # Default is outer-scope connect_timeout
              connect_timeout <INTEGER>
              # Optional fwmark to mark all outgoing
              # checker packets with
              fwmark <INTEGER>
           }

           # Number of times to retry a failed check
           retry <INTEGER>
           # Delay in seconds before retrying
           delay_before_retry <INTEGER>
           # Optional string to use for the SMTP HELO request
           helo_name <STRING>|<QUOTED-STRING>

           # Optional random delay to start the initial check
           # for maximum N seconds.
           # Useful to scatter multiple simultaneous
           # checks to the same RS. Enabled by default, with
           # the maximum at delay_loop. Specify 0 to disable
           warmup <INT>
        } #SMTP_CHECK

        # MISC healthchecker, run a program
        MISC_CHECK
        {
            # External script or program
            misc_path <STRING>|<QUOTED-STRING>
            # Script execution timeout
            misc_timeout <INT>

            # Optional random delay to start the initial check
            # for maximum N seconds.
            # Useful to scatter multiple simultaneous
            # checks to the same RS. Enabled by default, with
            # the maximum at delay_loop. Specify 0 to disable
            warmup <INT>

            # If set, the exit code from healthchecker is used
            # to dynamically adjust the weight as follows:
            #  exit status 0: svc check success, weight
            #    unchanged.
            #  exit status 1: svc check failed.
            #  exit status 2-255: svc check success, weight
            #    changed to 2 less than exit status.
            #  (for example: exit status of 255 would set
            #    weight to 253)
            misc_dynamic
        }
           } # realserver defn
        } # virtual service



AUTHOR
----
Joseph Mack.  
Yu Bo.  
Information derived from doc/keepalived.conf.SYNOPSIS, doc/samples/keepalived.conf.*
and Changelog by Alexandre Cassen for keepalived-1.1.4, and  from  HOWTOs  by  Adam
Fletcher and Vince Worthington.

SEE ALSO
----
ipvsadm(8), ip --help.

