# r6v4.github.io
page of user r6v4.    
email-0:linshunzhi@vip.163.com    

## h1d1 (http1.1 server powered by common-lisp)

```common-lisp
;;version:20260909(amd64)(v0.9.9)
;;md5:baf7e68cd9ee0a12b352958fe2ec485a
;;runtime:linux>=5.4.0,ldd>=2.27
;;github:https://github.com/r6v4/h1d1/releases/download/20260909/h1d1-2026-09-09_x86-64_linux5-ldd227_gencgc.7z
;;proton:https://drive.proton.me/urls/1TZKA9M7E4#81E6kQU2NfAc
;;lanzou:https://linshunzhi.lanzoub.com/igJ0v47ifqxe

```

first need to #sudo sh configure.sh when you are first time run on your system
second run the binary file h1d1-2026-09-09 (linux>=5.4 and glibc>=2.27)
with some options, 
    * is alway have, 
    + is when need, 
    % is will renew after 8 seconds,
    # is obsolete

    server ip address           --addr  127.0.0.1   *   
    server port                 --port  8080        *   
    server host name            --host  localhost   *   
    html directory              --menu  /tmp/       *   
    run thread number           --core  8           *   8
    renew cache time            --time  5minute     +       #
    cache max count             --size  10000000    +       #
    rocksdb configure file      --file  kvdb.txt    +   
    consed room before gc       --room  256MB       +   
    user kvdb right manage      --user  user.txt    + % 
    other cache list            --list  cache.txt   + % 
    web node configure file     --node  node.txt    + % 
    deny ipv4 address list      --deny  deny.txt    + % 
    0 no handshake timeout      --good  0           +   0   #
    0 not always return 404     --back  0           +   0
    1 is only local connect     --zone  0           +   0   #
    1 multi process server      --will  1           +   1
    crt file path               --crt   server.crt  *       #
    key file path               --key   server.key  *       #

Some notes:
    The minimum length of a single receive by the socket is 5byte, 
    which means that messages with a length less than 5 will be left for later processing, 
    The maximum length of single receive is 16KB,
    messages exceeding this limit will only be processed in the previous section.
    This version of the service does not support graceful exit
    The server will not respond to non-standard http requests if --back option not is 1.
    The newline for network messages is \r\n, and the newline for configuration files is \n
    Some options now only distinguish between 0 and non-zero, 
    for example --good 0 means turning off handshake timeout checking.
    --addr do not requires [] square brackets for ipv6
    --host is the domain name, such as localhost or example.com
    --list cache will not automatically update regularly as files change
    --file data files have two sets of storage engines, one is lmdb memory storage, 
        and the other is rocksdb disk storage.
    --room is the size of space allocated before memory cleaning. 
        If the value is too large or too small, it will affect the service.
    --good does not check tls handshake timeout when set to 0
    --core vcpu number, worker number, backlog number.eg, 8 is start 8 thread.
    --will when it is 1, multiple service processes are allowed to run at the same time. 
        When it is 0, the service will not be restarted.
    --back which is a switch left for performance testing and is turned off by default. 
        When the value is 1, a 404 page will be returned when encountering irregular network requests. 
        When the value is 0, this function is turned off and no response is made to irregular messages.
    --deny, set the list of ipv4 addresses that deny connections at the application layer
    --zone, when not set or the value is 0, normal connection can be made.
        When the value is 1, this function is enabled, and the server's data packets will not be routed, 
        so all connections need to be on the same machine, 
        that is, the client and server, including the back-end nodes, need to be in the local zone.
    --user, Each line has a token and an independent permission set. 
        The set is a string with a length of 8 BYTE. only byte is character 1 means have permission. 
        The function permissions represented from left to right are :box,let,set,put,get,see,del,rem. 
        An example of opening all permissions is 11111111
    --crt only supports single final certificate, not certificate chain
    --key, in the case of certificate encryption, 
        need to enter the certificate password in the terminal after the service is started.
    It is currently unable to connect normally with openssl related libraries.

Each h1d1 service sends a time stamp to a specified location 
through lmdb's shared memory every 0.2 seconds to indicate survival. 
Any process in the system can check this time stamp 
in env="/tmp/h1d1/"&db="TIME"&key="4 127.0.0.1 8080"&value="1234567890 123456".
When --will is 0 and other processes are detected to provide services, it will not start. 
In other cases, it will start normally.
This function can be used by the system to periodically detect whether the service is alive.
(time stamp not renew for more than 8 seconds is server offline)

in deny conf list:
ipv4-address-1
ipv4-address-2
ipv4-address-3
ipv4-address-4

in cache conf list:
url-0 file-path-0
url-1 file-path-1
url-2 file-path-2

in kvdb conf file:
/your/kvdb/url
your-password

for kvdb api http head: only need to send password once to log in
POST /your/kvdb/url HTTP/1.1\r\n
Host: your-host-name\r\n
Content-Length: body-length\r\n
X-Password: your-password\r\n
\r\n

for kvdb api http body: (<=16KB)
kvdb-part provides on-disk and on-memory interfaces
on-disk by rocksdb  use box set get del
on-memory by lmdb   use let put see rem
    
    box path ;to init a kvdb named path by open file or new
        send-example:box path \r\n
        send-length:11
        back-example:path
        back-length:4
    
    set path key-length val-length ;to set a kv in kvdb
        send-example:set path 3 5 \r\nKeyValue
        send-length:23
        back-example:Key
        back-length:3
    
    get path key-length ;to get the value of the key
        send-example:get path 3 \r\nKey
        send-length:16
        back-example:Value
        back-length:5
    
    del path key-length ;to delete the key
        send-example:del path 3 \r\nKey
        send-length:16
        back-example:Key
        back-length:3
    
    lmdb-env: 
        :IF-DOES-NOT-EXIST      :create
        :SYNCHRONIZED           nil
        :MAX-DBS                64
        :MAX-READERS            1024
        :MAP-SIZE               (* 1024 1024 64)
        :SUBDIR                 t
        :SYNC                   nil
        :META-SYNC              nil
        :READ-ONLY              nil
        :LOCK                   t
        :MEM-INIT               t
    
    let same as box
    put same as set
    see same as get
    rem same as del
    
test not pass on some client, maybe they ignore \r\n in end of post
\r\n means #\return and #\newline two char

when get the value is nil
Content-Length: 0
http body is ""

in user conf file:
token1 permission-set1
token2 permission-set2
token3 permission-set3

in node part conf file:
web-page-1 web-page-2 web-page-3 web-page-4
ipv4-node-1 ipv4-node-2 ipv4-node3 ipv4-node4
empty-line

for message in node part:
will-recv-message=Id(16bytes)Recv-message-body(from-17byte-to-about16KB)
need-send-message=Id(16bytes)Send-message-body(from-17byte-to-about8KB)
id is the first 16Bytes sequence of message
also is the identify of client session (alive connection)
support send large file to the node by http (requires "Content-Length" in header)
web node server example code at file node6000.lisp and node6001.lisp
these example need to install lisp-runtime quicklisp and libuv-devel (libuv1-dev)

this is the node api of h1d1 server, each vcpu has one thread, 
each thread has one connection which links to one of nodes
when node server is busy, send and recv will be unclear, so need many node server

news:
1.Fixed a node part session sequence conflict issue in version 0213 (20260214).
2.Added recognition and handling for UTF-8 URLs (20260906).
3.Fixed --list option, the code typos for permissions have been resolved.(20260909).

hint:
1. The total length of http messages does not exceed 128KB, 
    and the length of a single message sent does not exceed 16KB.
2. --zone --time --size --good --crt --key options are obsolete.
3. The software is still imperfect, 
    and some features will crash during stress testing.
4. vcpu64.so vpu affitify is drump.
