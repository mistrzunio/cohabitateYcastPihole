## configure ycast alongside pi-hole

dump from http://alriklubbers.com/2020/02/22/install-vtuner-alternative-alongside-pi-hole/

```
sudo  useradd ycast
sudo mkdir /etc/ycast 
sudo nano /etc/ycast/stations.yml 
sudo chown ycast:ycast /etc/ycast/stations.yml
```

then

```
sudo nano /etc/systemd/system/ycast.service 
```

and add the following content:

```
[Unit]
Description=YCast internet radio service
After=network.target

[Service]
Type=simple
User=ycast
Group=ycast
ExecStart=/usr/bin/python3 -m ycast -l 127.0.0.1 -p 8010 -c /etc/ycast/stations.yml

[Install]
WantedBy=multi-user.target
```

enable service:

```
sudo systemctl enable ycast.service
sudo systemctl start ycast.service
```

And check for any errors with 

```
systemctl status ycast.service
```

map vtuner dns to (static) IP address of your raspberry


```
sudo nano /etc/dnsmasq.d/vtuner.conf
```

IMPORTANT: replace 192.168.x.y  with IP of your rapsberry

```
address=/radiomarantz.vtuner.com/192.168.x.y 
address=/vtuner.com/192.168.x.y
```

then 

```
sudo lighttpd-enable-mod
```

enter: `proxy`

```
sudo nano /etc/lighttpd/conf-enabled/10-proxy.conf 
```

and replace the content with the following configuration (or edit appropriately):

```
# /usr/share/doc/lighttpd/proxy.txt

server.modules   += ( "mod_proxy" )

## Balance algorithm, possible values are: "hash", "round-robin" or "fair" (default)
# proxy.balance     = "hash" 


## Redirect all queries to files ending with ".php" to 192.168.0.101:80 
#proxy.server     = ( ".php" =>
#                     ( 
#                       ( "host" => "192.168.0.101",
#                         "port" => 80
#                       )
#                     )
#                    )

## Redirect all connections on www.example.com to 10.0.0.1{0,1,2,3}
#$HTTP["host"] == "www.example.com" {
#  proxy.balance = "hash"
#  proxy.server  = ( "" => ( ( "host" => "10.0.0.10" ),
#                            ( "host" => "10.0.0.11" ),
#                            ( "host" => "10.0.0.12" ),
#                            ( "host" => "10.0.0.13" ) ) )
#}


#YCAST
$HTTP["host"] =~ "vtuner.com" {
        proxy.server = ( "" =>  ( (
                                "host" => "127.0.0.1",
                                "port" => "8010"
                                 ) ) )
        proxy.forwarded = (
                        "for"          => 1,
                        "proto"        => 1,
                        "host"        => 1,
                        #"by"          => 1,
                        #"remote_user" => 1
    )

}
```

restart web server: 

```
sudo service lighttpd force-reload
systemctl status lighttpd.service
```
