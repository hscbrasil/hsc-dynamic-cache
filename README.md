# hsc-dynamic-cache

## Squid Configuration

```
url_rewrite_program /usr/bin/squidGuard

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

## squidGuard Configuration

```
rewrite yt {
	s@(https://.*youtube.com/watch\?v=.*)@\1\&html5=1@i
}

acl {
	default {
		pass	all
		rewrite	yt
		log	verbose squidGuard.log
	}
}
```
