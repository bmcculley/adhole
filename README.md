# adhole

AdHole is a simple transparent advertisement and tracking blocker intended for 
personal use.

It works by providing two elements: first a DNS server that, based on a list of 
domains, either relies the query to a real server, or returns its own IP 
address; and a micro HTTP server that serves an empty GIF image for any 
request. As such its functionality is akin to 
[AdBlock](https://chrome.google.com/webstore/detail/adblock/gighmmpiobklfepjocna
mgkkbiglidom?hl=en), but less pretty (you'll probably see empty space where ads 
used to be) and completely transparent (i.e. will work for any browser, on any 
OS on any machine, locally or over any network). Note that you can just as well 
block *any* domains.

I use AdHole in 'production' on my 
[RaspberryPi](http://www.raspberrypi.org/)-powered WiFi router, where it 
provides ad and tracking blocking to all my wireless toys.

My programming goal was to make it as simple and minimal as possible. Therefore 
instead of a full DNS stack (and especially parsing) I work directly on 
`[]byte` slices and try to reuse the data already on hand. It was also a nice 
little adventure in dealing with binary protocols the low-level way.

## Usage

    Usage: ./adhole [options] upstream proxy list.txt
    
    upstream - real upstream DNS address, e.g. 8.8.8.8
    proxy    - servers' bind address, e.g. 127.0.0.1
    list.txt - text file with domains to block
    
      -dport=53: DNS server port
      -hport=80: HTTP server port
      -t=100ms: upstream query timeout
      -v=false: be verbose

Note that you will need root privileges to run it on the default ports.

List format is simply: one domain name per line. All subdomains of a given 
domain will be blocked, so there is no need to use `*`. Domains should also not 
end with a dot. The parser should also be indifferent to line endings. Example 
list file:

    101com.com
    101order.com
    103bees.com
    123found.com
    123pagerank.com

To get a decent list of domains to block I recommend going 
[here](http://pgl.yoyo.org/adservers/). Remember to set 'text only, one domain 
per line' format before you export and save the list.

Example [systemd](http://www.freedesktop.org/wiki/Software/systemd/) service 
file:

    [Unit]
    Description=DNS blackhole
    After=network.target
    
    [Service]
    ExecStart=/home/user/adhole 192.168.0.3 192.168.0.21 /home/user/blacklist.txt
    
    [Install]
    WantedBy=multi-user.target

Thanks to the great [expvar](http://golang.org/pkg/expvar/) package you can 
monitor some statistics by visiting `http://proxy.addr/debug/vars`. The 
following items are relevant:

  * `statsQuestions` - number of received queries
  * `statsRelayed` - number of queries relayed to the real server
  * `statsBlocked` - number of queries blocked
  * `statsTimedout` - number of relayed queries that timed out
  * `statsServed` - number of HTTP requests served
  * `statsErrors` - number of errors encountered
  * `statsRules` - number of items read from the blacklist

You can also reload the blacklist file while AdHole is running by sending it a 
`SIGUSR1` (i.e. ``kill -SIGUSR1 `pidof adhole` ``).

## Bugs / Todo

  * Edge cases (multiple questions per query, anybody?)
  * Even less data shuffling
  * IPv4 only
  * Does Windows have `syscall.SIGUSR1`?

## Copyright

Copyright (c) 2014 Piotr S. Staszewski

Absolutely no warranty. See LICENSE.txt for details.
