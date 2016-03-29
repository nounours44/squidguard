## Introduction

this image is an [squidGuard](http://www.squidguard.org/) addition to [sameersbn/docker-squid](https://github.com/sameersbn/docker-squid). I find squidGuard very useful to limit access to certain internet pages and to reduce the risk for downloading dangerous software. A central filtering solution is preferred especially if you have a family with children and different devices.

**new: You can run this container with own white- and blacklists or with public available lists from an external source.** Recommended  blacklists are provided by [shallalist](http://www.shallalist.de/) - with some license restrictions especially for commercial use.

This image includes also automatic proxy discovery based on WPAT and DHCP. Here a Webserver is required that serves wpat.dat.

## Sample 1: black- and whitelists from [shallalist](http://www.shallalist.de/) 

create a docker-compose.yml file
```
squidguard:
  image: muenchhausen/docker-squidguard:latest
  environment:
    - UPDATE_BLACKLIST_URL=http://www.shallalist.de/Downloads/shallalist.tar.gz
  ports:
    - "3128:3128"
    - "80:80"
  expose:
    - 3128
    - 80
```
Setting the env Variable UPDATE_BLACKLIST_URL, the configuration in folder [sample-config-blacklist](https://github.com/muenchhausen/docker-squidguard/blob/master/sample-config-blacklist) will be used. Otherwise the [sample-config-simple](https://github.com/muenchhausen/docker-squidguard/blob/master/sample-config-simple) is used. In practise you need to configure your own black- and whitelists - see the next sample.

## Sample 2: own whitelists with docker-compose

create a docker-compose.yml file ( this [docker-compose.yml](https://github.com/muenchhausen/docker-squidguard/blob/master/docker-compose.yml) includes comments to all variations ) :
```
squidguard:
  image: muenchhausen/docker-squidguard:latest
  environment:
    - SQUID_CONFIG_SOURCE=/custom-config
    - SQUID_CONFIG_SOURCE_UID=1000          # only required if MAC OS is used: UserID for user proxy
  ports:
    - "3128:3128"
    - "80:80"
  expose:
    - 3128
    - 80
  volumes:
    - /Users/derk/myconfig:/custom-config
```

create a ```squidGuard.conf``` file in your local myconfig directory
```
dbhome /var/lib/squidguard/db
logdir /var/log/squidguard

dest mywhite {
        domainlist      /custom-config/whiteDomains
        urllist         /custom-config/whiteUrls
}

acl {
        default {
                pass    mywhite	none
                redirect http://localhost/block.html
                }
}
```

create a ```whiteDomains``` file in your local myconfig directory
```
debian.org
wikipedia.org
muenchhausen.de
```

create a ```whiteUrls``` file in your local myconfig directory
```
github.com/muenchhausen/
```

## Run and Test it! 

* enter the directory where your docker-compose.yml file is located and run simply
```
docker-compose stop && docker-compose rm -f && docker-compose build && docker-compose up --force-recreate
```

* open a second bash, run e.g.:
```curl --proxy 192.168.99.100:3128 https://en.wikipedia.org/wiki/Main_Page```

* test a blocked domain from the adv blacklist. This is blocked if UPDATE_BLACKLIST_URL is used:
```curl --proxy 192.168.99.100:3128 http://www.linkadd.de```

* test it in your Browser: Set docker host IP and port 3128 in your proxy settings or operating system proxy configuration.

* if you decided for the WPAT autoproxy variant, just do now a DHCP release and you get your proxy settings :)

## Additions

WPAT proxy settings allow your Operating system to find the settings automatically based on your DHCP settings:

```docker run --name='squidguard' -it --env WPAT_IP=192.168.99.100 --env WPAT_NOPROXY_NET=192.168.99.0 --env WPAT_NOPROXY_MASK=255.255.255.0 --rm -p 3128:3128 -p 80:80 muenchhausen/docker-squidguard:latest```

To use WPAT, add a cusom-proxy-server option 252 to your DHCP server. Use "http://${WPAT_IP}/wpat.dat" e.g. "http://192.168.59.103/wpat.dat" as your option value. See [squidGuard Wiki](http://wiki.squid-cache.org/SquidFaq/ConfiguringBrowsers#Automatic_WPAD_with_DHCP) for further details.

You can add these settings also to your compose file - see here: [docker-compose.yml](https://github.com/muenchhausen/docker-squidguard/blob/master/docker-compose.yml)


## recommended documentation

For Squid basis configuration, please refer to the documentation of [sameersbn/docker-squid](https://github.com/sameersbn/docker-squid).

A simple documentation of how to configure squidGuard blacklists can be found in the [squidGuard configuration documentation](http://www.squidguard.org/Doc/configure.html).


## Shell Access

For debugging and maintenance purposes you may want access the containers shell. Either add after the run command or tun e.g.

```docker exec -it dockersquidguard_squidguard_1 bash```

## Autostart the container

add the parameter --restart=always to your docker run command.

