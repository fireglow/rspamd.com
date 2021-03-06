---
layout: default
title: Rspamd milter
---

# Rmilter - rspamd milter

## Introduction

Rmilter is used to integrate rspamd and `milter` compatible MTA, for example [postfix](http://postfix.org) or [sendmail](http://sendmail.org).
Rmilter can also do other useful stuff: 

-   Clamav scanning (via unix or tcp socket).
-   Rspamd scanning
-   Spf checking (via libspf2) - deprecated
-   Greylisting with memcached upstream
-   Ratelimit with memcached upstream
-   Auto-whitelisting (internal and via memcached upstream)
-   Replies check (whitelisting replies to sent messages)
-   Passing messages and/or their headers to beanstalk servers


All `rmilter` configuration is placed in rmilter.conf file.

Configuration format
--------------------

The configuration file has format:

    name = value ;

Value may be:

-   String (may not contain spaces)
-   Quoted string (if string may contain spaces)
-   Numeric value
-   Flag (`y`, `Yes` or `n`, `No`)
-   IPv4 or network (eg. `127.0.0.1`, `192.168.1.0/24`)
-   Socket argument (eg. `host:port` or `/path/to/socket`)
-   Regexp (eg. `/Match\*/`)
-   List (eg. `value1, value2, value3`)
-   Recipients list (`user, user@domain, @domain`)
-   Time argument (eg. `10s`, `5d`)

Some directives MUST be specified only in specified sections. Section
definition looks like:

     section_name {
             section_directive;
             ...
     };

Common sections:

-   clamav - clamav definitions
-   spamd - rspamd definitions
-   limits - limits definitions
-   greylisting - greylisting definitions
-   rule - regexp rule definition (a section per rule)

Directives that can be defined in config file:

-   Global section:

    - `pidfile`: specify path to pidfile

        + Default: `/var/run/rmilter.pid`

    - `tempdir`: specify path to temporary directory. For maximum performance it is
    recommended to put it on memory file system.

        + Default: `$TMPDIR`

    - `bind_socket`: socket credits for local bind:

        1.  `unix:/path/to/file` - bind to local socket
        2.  `inet:[port@host]` - bind to inet socket

        + Default: `bind_socket = unix:/var/tmp/rmilter.sock`

    - `max_size`: maximum size of scanned message for clamav, spamd and dcc.

        + Default: `0 (no limit)`

    - `strict_auth`: strict checks for mails from authenticated senders

        + Default: `no`

    - `spf_domains`: list of domains that would be checked with spf

        + Default: `empty (spf disabled)`

    - `use_dcc`: flag that specify whether we should use dcc checks for mail

        + Default: `no`

    - `whitelist`: global recipients whitelist

        + Default: `no`

-   Clamav section:

    - `servers`: clamav socket definitions in format:

        1.  `/path/to/file`
        2.  `host[:port]`

		Sockets are separated by `,`

        + Default: `empty`

    - `connect_timeout`: timeout in miliseconds for connecting to clamav

        + Default: `1s`

    - `port_timeout`: timeout in miliseconds for waiting for clamav port response

        + Default: `4s`

    - `results_timeout`: timeout in miliseconds for waiting for clamav response

        + Default: `20s`

    - `error_time`: time in seconds during which we are counting errors

        + Default: `10`

    - `dead_time`: time in seconds during which we are thinking that server is down

        + Default: `300`

    - `maxerrors`: maximum number of errors that can occur during error_time to make
    us thinking that this upstream is dead

        + Default: `10`

-   Spamd section:

    - `servers`: spamd (or rspamd) socket definitions in format:

        1.  `/path/to/file`
        2.  `host[:port]`
        3.  `r:/path/to/file` - for rspamd protocol
        4.  `r:host[:port]` - for rspamd protocol

    	Sockets are separated by `,`.

        + Default: `empty (spam checks disabled)`

    - `connect_timeout`: timeout in miliseconds for connecting to spamd

        + Default: `1s`

    - `results_timeout`: timeout in miliseconds for waiting for spamd response

        + Default: `20s`

    - `error_time`: time in seconds during which we are counting errors

        + Default: `10`

    - `dead_time`: time in seconds during which we are thinking that server is down

        + Default: `300`

    - `maxerrors`: maximum number of errors that can occur during error_time to make
    us thinking that this upstream is dead

        + Default: `10`

    - `reject_message`: reject message for spam (quoted string)

        > *Default: `Spam message rejected; If this is not spam contact abuse team`

    - `spamd_soft_fail`: if action is not reject use it for other actions (flag)

        + Default: `true`

    - `spamd_greylist`: greylist message only if action is greylist (flag)

        + Default: `true`

    - `spam_header`: add specified header if action is add_header and
    spamd_soft_fail os turned on

        > *Default: `X-Spam`

    - `rspamd_metric`: rspamd metric that would define whether we reject message as spam
    or not (quoted string)

        > *Default: `default`

    - `whitelist`: list of ips or nets that should be not checked with spamd

        + Default: `empty`

    - `extended_spam_headers`: add extended spamd headers to messages, is useful for debugging or
    private mail servers (flag)

        + Default: `false`

-   Memcached section:

    - `servers_grey`: memcached servers for greylisting in format:
    `host[:port][, host[:port]]`
    It is possible to make memcached mirroring (for two servers only), its syntax is `{server1,server2}`

        + Default: `empty`

    - `servers_white`: memcached servers for whitelisting in format similar to that is
    used in *servers_grey*

        + Default: `empty`

    - `servers_limits`: memcached servers used for limits storing, can not be mirrored

        + Default: `empty`

    - `connect_timeout`: timeout in miliseconds for connecting to memcached

        + Default: `1s`

    - `error_time`: time in seconds during which we are counting errors

        + Default: `10`

    - `dead_time`: time in seconds during which we are thinking that server is down

        + Default: `300`

    - `maxerrors`: maximum number of errors that can occur during error_time to make
    us thinking that this upstream is dead

        + Default: `10`

    - `protocol`: protocol that is using for connecting to memcached (tcp or udp)

        + Default: `udp`

-   Beanstalk section:

    - `servers`: beanstalk servers for pushing headers in format:
    `host[:port][, host:port]`

        + Default: `empty`

    - `copy_server`: address of server to which rmilter should send all messages copies

        + Default: `empty`

    - `spam_server`: address of server to which rmilter should send spam messages
    copies

        + Default: `empty`

    - `connect_timeout`: timeout in miliseconds for connecting to beanstalk

        + Default: `1s`

    - `error_time`: time in seconds during which we are counting errors

        + Default: `10`

    - `dead_time`: time in seconds during which we are thinking that server is down

        + Default: `300`

    - `maxerrors`: maximum number of errors that can occur during error_time to make
    us thinking that this upstream is dead

        + Default: `10`

    - `id_regexp`: regexp that defines for which messages we should put the whole
    message to beanstalk, not only headers, now this regexp checks only
    `In-Reply-To` headers

        + Default: `empty`

    - `send_beanstalk_headers`: defines whether we should send headers to beanstalk servers (from
    servers option)

        + Default: `no`

    - `send_beanstalk_copy`: defines whether we should send copy of messages to beanstalk
    server (from copy_server option)

        + Default: `no`

    - `send_beanstalk_spam`: defines whether we should send copy of spam messages to beanstalk
    server (from spam_server option)

        + Default: `no`

    - `protocol`: protocol that is using for connecting to beanstalk (tcp or udp)

        + Default: `tcp`

-   Greylisting section:

    - `timeout (required)`: time during which we mark message greylisted

        + Default: `300s`

    - `expire (required)`: time during which we save a greylisting record

        + Default: `empty (greylisting disabled)`

    - `whitelist`: list of ip addresses or networks that should be whitelisted from
    greylisting

        + Default: `empty`

    - `awl_enable`: enable internal auto-whitelist mechanics

        + Default: `no`

    - `awl_pool`: size for in-memory auto whitelist

        + Default: `10M`

    - `awl_hits`: number of messages (from this ip) that passes greylisting to put
    this ip into whitelist

        + Default: `10`

    - `awl_ttl`: time to live for ip address in auto whitelist

        + Default: `3600s`

-   Limits section.

    Rate limits are implemented as leaked bucket, so first value is
    bucket burst - is peak value for messages in bucket (after reaching
    it bucket is counted as overflowed and new messages are rejected),
    second value is rate (how much messages can be removed from bucket
    each second). It can be schematically displayed:

                |------------------|          <----- current value
                |                  |
                |------------------|          <----- burst
                |                  |
                |                  |
                |                  |
                |                  |
                \                  /
                 ----------------- .....      <----- rate (speed of emptying)

    - `limit_whitelist_ip`: don't check limits for specified ips

        + Default: `empty`

    - `limit_whitelist_rcpt`: don't check limits for specified recipients

        + Default: `no`

    - `limit_bounce_addrs`: list of address that require more strict limits

        + Default: `postmaster, mailer-daemon, symantec_antivirus_for_smtp_gateways, null, fetchmail-daemon`

    - `limit_bounce_to`: limits bucket for bounce messages (only rcpt to)

        + Default: `5:0.000277778`

    - `limit_bounce_to_ip`: limits bucket for bounce messages (only rcpt to per one source ip)

        + Default: `5:0.000277778`

    - `limit_to`: limits bucket for non-bounce messages (only rcpt to)

        + Default: `20:0.016666667`

    - `limit_to_ip`: limits bucket for non-bounce messages (only rcpt to per one source
    ip)

        + Default: `30:0.025`

    - `limit_to_ip_from`: limits bucket for non-bounce messages (msg from, rcpt to per one
    source ip)

        + Default: `100:0.033333333`

-   DKIM section.

    Dkim can be used to sign messages by. Dkim support must be
    provided with opendkim library and `rmilter` must be configured
    with `--enable-dkim` option.

    - `header_canon`: canonization of headers (simple or relaxed)

        + Default: `simple`

    - `body_canon`: canonization of body (simple or relaxed)

        + Default: `simple`

    - `sign_alg`: signature algorithm (`sha1` and `sha256`)

        + Default: `sha1`
    
    - `auth_only`: sign mail for authorized users only

        + Default: `yes`

    - `domain`: domain entry must be enclosed in a separate section

    -   `key` - path to private key
    -   `domain` - domain to be used for signing (this matches with
        SMTP FROM data). If domain is `*` then rmilter tries to search key in the `key` path as `keypath/domain.selector.key` for any domain.
    -   `selector` - dkim DNS selector (e.g. for selector *dkim* and
        domain *example.com* DNS TXT record should be for
        dkim._domainkey.example.com).

Example configuration
---------------------

    # pidfile - path to pid file
    # Default: pidfile = /var/run/rmilter.pid

    pidfile = /var/run/rmilter/rmilter.pid;


    clamav {
            # servers - clamav socket definitions in format:
            # /path/to/file
            # host[:port]
            # sockets are separated by ','
            # Default: empty
            servers = clamav.test.ru, clamav.test.ru, clamav.test.ru;
            # connect_timeout - timeout in miliseconds for connecting to clamav
            # Default: 1s
            connect_timeout = 1s;

            # port_timeout - timeout in miliseconds for waiting for clamav port response
            # Default: 4s
            port_timeout = 4s;

            # results_timeout - timeout in miliseconds for waiting for clamav response
            # Default: 20s
            results_timeout = 20s;

            # error_time - time in seconds during which we are counting errors
            # Default: 10
            error_time = 10;

            # dead_time - time in seconds during which we are thinking that server is down
            # Default: 300
            dead_time = 300;

            # maxerrors - maximum number of errors that can occur during error_time to make us thinking that 
            # this upstream is dead
            # Default: 10
            maxerrors = 10;
    };

    spamd {
            # servers - spamd socket definitions in format:
            # /path/to/file
            # host[:port]
            # sockets are separated by ','
            # Default: empty
            servers = r:localhost:11333;
            # connect_timeout - timeout in miliseconds for connecting to spamd
            # Default: 1s
            connect_timeout = 1s;

            # results_timeout - timeout in miliseconds for waiting for spamd response
            # Default: 20s
            results_timeout = 20s;

            # error_time - time in seconds during which we are counting errors
            # Default: 10
            error_time = 10;

            # dead_time - time in seconds during which we are thinking that server is down
            # Default: 300
            dead_time = 300;

            # maxerrors - maximum number of errors that can occur during error_time to make us thinking that 
            # this upstream is dead
            # Default: 10
            maxerrors = 10;

            # reject_message - reject message for spam
            # Default: "Spam message rejected; If this is not spam contact abuse team"
            reject_message = "Spam message rejected; If this is not spam contact abuse team";

            # whitelist - list of ips or nets that should be not checked with spamd
            # Default: empty
            whitelist = 127.0.0.1/32, 192.168.0.0/16;
    };

    memcached {
            # servers_grey - memcached servers for greylisting in format:
            # host[:port][, host[:port]]
            # It is possible to make memcached mirroring, its syntax is {server1, server2}
            servers_grey = {localhost, memcached.test.ru}, memcached.test.ru:11211;

            # servers_white - memcached servers for whitelisting in format similar to that is used
            # in servers_grey
            # servers_white = {localhost, memcached.test.ru}, memcached.test.ru:11211;
            
            # servers_limits - memcached servers used for limits storing, can not be mirrored
            servers_limits = memcached.test.ru, memcached.test.ru:11211;

            # connect_timeout - timeout in miliseconds for waiting for memcached
            # Default: 1s
            connect_timeout = 1s;

            # error_time - time in seconds during which we are counting errors
            # Default: 10
            error_time = 10;

            # dead_time - time in seconds during which we are thinking that server is down
            # Default: 300
            dead_time = 300;

            # maxerrors - maximum number of errors that can occur during error_time to make us thinking that 
            # this upstream is dead
            # Default: 10
            maxerrors = 10;

            # protocol - protocol that is using for connecting to memcached (tcp or udp)
            # Default: udp
            protocol = tcp;
    };

    # bind_socket - socket credits for local bind:
    # unix:/path/to/file - bind to local socket
    # inet:port@host - bind to inet socket
    # Default: bind_socket = unix:/var/tmp/rmilter.sock;

    bind_socket = unix:/var/run/rmilter/rmilter.sock;

    # tempdir - path to directory that contains temporary files
    # Default: $TMPDIR

    tempdir = /spool/tmp;

    # max_size - maximum size of scanned mail with clamav and dcc
    # Default: 0 (no limit)

    max_size = 10M;

    # strict_auth - strict checks for mails from authenticated senders
    # Default: no

    strict_auth = no;

    # spf_domains - path to file that contains hash of spf domains
    # Default: empty

    spf_domains = rambler.ru, mail.ru;

    # use_dcc - whether use or not dcc system
    # Default: no

    use_dcc = yes;

    # whitelisted recipients
    # domain are whitelisted as @example.com
    whitelist = postmaster, abuse;

    # rule definition:
    # rule {
    #       accept|discard|reject|tempfail|quarantine "[message]"; <- action definition
    #       [not] connect <regexp> <regexp>; <- conditions
    #       helo <regexp>;
    #       envfrom <regexp>;
    #       envrcpt <regexp>;
    #       header <regexp> <regexp>;
    #       body <regexp>;
    # };

    # limits section
    limits {
            # Whitelisted ip
            limit_whitelist_ip = 194.67.45.4;
            # Whitelisted recipients
            limit_whitelist_rcpt =  postmaster, mailer-daemon;
            # Addrs for bounce checks
            limit_bounce_addrs = postmaster, mailer-daemon, symantec_antivirus_for_smtp_gateways, <>, null, fetchmail-daemon;
            # Limit for bounce mail
            limit_bounce_to = 5:0.000277778;
            # Limit for bounce mail per one source ip
            limit_bounce_to_ip = 5:0.000277778;
            # Limit for all mail per recipient
            limit_to = 20:0.016666667;
            # Limit for all mail per one source ip
            limit_to_ip = 30:0.025;
            # Limit for all mail per one source ip and from address
            limit_to_ip_from = 100:0.033333333;
    };

    beanstalk {
            # List of beanstalk servers, random selected
            servers = bot01.example.com:3132;
            # Beanstalk protocol
            protocol = tcp;
            # Time to live for task in seconds
            lifetime = 172800;
            # Regexp that define for which messages we should put the whole message to beanstalk
            # now only In-Reply-To headers are checked
            id_regexp = "/^SomeID.*$/";
    };

    dkim {
        domain {
            key = /usr/local/etc/dkim_example.key;
            domain = "example.com";
            selector = "dkim";
        };
        domain {
            key = /usr/local/etc/dkim_test.key;
            domain = "test.com";
            selector = "dkim";
        };
        header_canon = relaxed;
        body_canon = relaxed;
        sign_alg = sha256;
        auth_only = yes;
    };

NOTES
-----

There are several things that might be useful to notice.

The order of checks:
--------------------

1.  DKIM test from and create signing context (MAIL FROM)
2.  Ratelimit (RCPT TO)
3.  Greylisting (DATA)
4.  Ratelimit (EOM, set bucket value)
5.  Rules (EOM)
6.  SPF (EOM)
7.  Message size (EOM) if failed, skip clamav, dcc and spamd checks
8.  DCC (EOM)
9.  Clamav (EOM)
10. Spamassassin (EOM)
11. Beanstalk (EOM)
12. DKIM add signature (EOM)

Keys used in memcached:
-----------------------

-   *rcpt* - bucket for rcpt filter
-   *rcpt:ip* - bucket for rcpt_ip filter
-   *rcpt:ip:from* - bucket for rcpt_ip_from filter
-   *rcpt:* - bucket for bounce_rcpt filter
-   *rcpt:ip:* - bucket for bounce_rcpt_ip filter
-   *md5(from . ip . to)* - key for greylisting triplet (hexed string of
    md5 value)

Postfix settings
----------------

There are several useful settings for postfix to work with this milter:

    smtpd_milters = unix:/var/run/rmilter/rmilter.sock
    milter_mail_macros =  i {mail_addr} {client_addr} {client_name} {auth_authen}
    milter_protocol = 6
