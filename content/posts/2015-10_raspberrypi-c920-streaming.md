+++
date = "2015-10-10T15:09:06+02:00"
title = "Raspberry Pi and Logitech C920 Webcam live streaming using rtmp"
+++

The Logitech C920 Webcam is ideal for streaming since it provides a h264 output out-of-the box. Especially if your streaming box has limited computing power like a raspberry pi.
Most people use `motion` or other mjpeg webcam solutions, but i will show how to get 15-20 FPS live stream at 720p out of the raspberry pi model b.
This solution also includes the ability to use `jwplayer` to watch your stream using most webbrowsers.
These instructions may not be complete and only show you the way to go.
<!--more-->

Idea
----
- Read h264 stream from cam
- Stream to local nginx with rtmp module
- Create static website with jwplayer to view webcam

Requirements
------------

I'm using arch linux on my raspberry but you can adapt those requirements to your system like raspbian.

- ffmpeg
- gcc
- make
- lib32-openssl
- git
- v4l-utils

Compile nginx with rtmp module
-----
Get the [rtmp nginx module](https://github.com/arut/nginx-rtmp-module) (git clone) and the [nginx source](http://nginx.org/download/).

```bash
$ cd nginx-src/
$ ./configure --add-module=/path/to/nginx-rtmp-module
$ make
$ make install
```

Setup nginx
-----------

Create directory `/tmp/hls` and chmod 777.

Edit `/usr/local/nginx/conf/nginx.conf`:

```nginx
rtmp {
    server {
        listen 1935;
        chunk_size 4000;

        # HLS
        application hls {
            live on;
            hls on;
            hls_path /tmp/hls;
        }
    }
}

# HTTP can be used for accessing RTMP stats
http {
    server {
        listen      80;

        # This URL provides RTMP statistics in XML
        location /stat {
            rtmp_stat all;

            # Use this stylesheet to view XML as web page
            # in browser
            rtmp_stat_stylesheet stat.xsl;
        }

        location /stat.xsl {
            # XML stylesheet to view RTMP stats.
            # Copy stat.xsl wherever you want
            # and put the full directory path here
            root /path/to/stat.xsl/;
        }

        location /hls {
            # Serve HLS fragments
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            root /tmp;
            add_header Cache-Control no-cache;
        }
    }
}
events { worker_connections 1024; }
```

Setup stream webpage
-------------

Download [jwplayer](http://www.jwplayer.com/), the free version is all you need. There's no need for an api key, so you might get the jwplayer files elsewhere.

Extract the contents to `/usr/local/nginx/html/jwplayer`.

Modify `/usr/local/nginx/html/index.html`:
```html
<!DOCTYPE html>
<html>
<head>
<title>Cam</title>
<script src="jwplayer/jwplayer.js"></script>
<style>
    body {
        width: 35em;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
        #stream {
                width: 100%;
                height: 100%;
        }
</style>
</head>
<body>
<h1>Cam</h1>
<div id="stream">Loading...</div>
<script type="text/javascript">
        jwplayer("stream").setup({

playlist: [{
        sources: [{
                file:"rtmp://192.168.0.106:1935/hls/movie"
        }]
}],
width: 1280,
height: 720,
primary: "flash"
});

</script>
</body>
</html>
```
Remember to adjust the `playlist.sources.file` path to your ip address.


Start nginx: 
```bash
$ /usr/local/nginx/bin/nginx
```

Setup ffmpeg
------------

Change resolution of your output
```bash
$ v4l2-ctl --device=/dev/video0 --set-fmt-video=width=1280,height=720,pixelformat=1 
```
You have to change the resolution after every reboot and camera connect.

Stream to nginx
---------------

```
$ ffmpeg -r 15 -use_wallclock_as_timestamps 1 -copytb 0 -f v4l2 -vcodec h264 -i /dev/video0 -vcodec copy -f flv rtmp://127.0.0.1:1935/hls/movie
```

I use `-r 15` and `-use_wallclock_as_timestamps` which solves the `Incorrect dts` errors of ffmpeg. This reduces performance a bit and you may want to use the timestamps without modification.


Check your stream
-----------------

Check `http://yourip/` which should serves your `index.html`, starts `jwplayer` and reads your rtmp stream.

You could also watch the stream directly using vlc:

Open Networkstream -> `rtmp://yourip:1935/hls/movie`
