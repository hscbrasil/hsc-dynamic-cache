# HSC Dynamic Cache

*HSC Dynamic Cache delivers ready-to-use Squid-3.5 "Store ID" helper. To improve the cache keys, there is an URL patterns file which can be customized by the user.*

Here is the list of patterns included:

* IMDb trailers
* Sourceforge downloads
* R7 vídeos
* Metacafe
* MSN videos
* Terra TV
* Globo vídeos
* Facebook
* YouTube
* Google Maps

## Requirements

* Perl
* Squid >= 3.5
* squidGuard or equivalent `url_rewrite_program` (optional)

## Installation

```bash
# cp hsc-dynamic-cache /usr/local/bin/
# cp hsc-dynamic-cache-db.txt /usr/local/etc/
# chmod 755 /usr/local/bin/hsc-dynamic-cache
# chown root.squid /usr/local/bin/hsc-dynamic-cache
# chmod 644 /usr/local/etc/hsc-dynamic-cache-db.txt
# chown root.squid /usr/local/etc/hsc-dynamic-cache-db.txt
# cp extras/simplerewrite /usr/local/bin/
# chmod 755 /usr/local/bin/simplerewrite
# chown root.squid /usr/local/bin/simplerewrite
```

## Dynamic caching

### Squid Configuration

Dynamic caching is enabled by setting the following configuration parameters in the squid.conf file:

```
#url_rewrite_program /usr/bin/squidGuard
url_rewrite_program /usr/local/bin/simplerewrite

acl rewritedoms dstdomain .dailymotion.com .video-http.media-imdb.com .dl.sourceforge.net .prod.video.msn.com .fbcdn.net .akamaihd.net vl.mccont.com .mais.uol.com.br .streaming.r7.com
acl yt url_regex -i googlevideo.*videoplayback
acl gmaps url_regex -i ^https?:\/\/(khms|mt)[0-9]+\.google\.[a-z\.]+\/.*
acl ttv url_regex -i terratv
acl globo url_regex -i ^http:\/\/voddownload[0-9]+\.globo\.com.*
acl dm url_regex -i dailymotion\-flv2
acl getmethod method GET

range_offset_limit none
quick_abort_min -1 KB

store_id_program /usr/local/bin/hsc-dynamic-cache -file /usr/local/etc/hsc-dynamic-cache-db.txt
store_id_extras "%>a/%>A %un %>rm myip=%la myport=%lp referer=%{Referer}>h"
store_id_children 40 startup=10 idle=5 concurrency=0
store_id_access deny !getmethod
store_id_access allow rewritedoms
store_id_access allow yt
store_id_access allow gmaps
store_id_access allow ttv
store_id_access allow globo
store_id_access allow dm
store_id_access deny all

refresh_pattern -i squid\.internal	10080	80%	79900 override-lastmod override-expire ignore-reload ignore-no-store ignore-must-revalidate ignore-private ignore-auth
```

## YouTube caching

There is no additional step required when using our `simplerewrite` custom helper.

### squidGuard Configuration

YouTube caching can also be enabled by setting the following configuration parameters in the squidGuard.conf file:

```
logdir /var/log/squidGuard

rewrite yt {
	s@(https://.*youtube.com/watch\?v=.*)@\1\&html5=1@i
	s@(https://.*youtube.com/v/.*)@\1\&html5=1@i
}

acl {
	default {
		pass	all
		rewrite	yt
		log	verbose squidGuard.log
	}
}
```
