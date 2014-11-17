[![Stories in Ready](https://badge.waffle.io/rcarmo/raspi-cluster.png?label=ready&title=Ready)](https://waffle.io/rcarmo/raspi-cluster)
# raspi-cluster

![Cabled](https://raw.github.com/rcarmo/raspi-cluster/master/pics/ethernet.jpg)

## What?

A few months back, I decided I'd build a small cluster of [Raspberry Pi][rpi] boards, and this repository is used for versioning design notes, configuration files and sundry.

## Why?

I wanted something challenging to do in terms of distributed processing, and lacked dedicated hardware to do it. There's a lot to be learned even from simple, unsophisticated solutions, and virtual machines can only do so much.

## How?

The cluster consists of five nodes: a master and four slaves. The master acts as a gateway, DHCP and NFS server and the slave nodes get their IP address and `/srv/jobs` directory from it.

All slave nodes are identical -- _completely_ identical, except for hostname and MAC address, and there is no need to log in and configure things manually for each node.

Here's a couple more shots of it, with the 5-port PSU and the initial assembly:

![Power cords](https://raw.github.com/rcarmo/raspi-cluster/master/pics/micro_usb.jpg)
![First assembly](https://raw.github.com/rcarmo/raspi-cluster/master/pics/first_assembly.jpg)

In retrospect I probably ought to have gone for longer USB cables and moved all of the cabling to the USB side (since it leaves the SD card slot clear), but I also need to be able to see the activity lights, and the Pi isn't exactly designed for easy stacking.

A larger cluster is certainly feasible, but 5 boards is as much as I can power with the PSU I have.

## Hardware

This is a partial list of the stuff I'm using (Amazon UK affiliate links):

* [5x Raspberry Pi Model B](http://www.amazon.co.uk/Raspberry-Pi-Model-512MB-RAM/dp/B008PT4GGC/ref=as_li_tf_tl?ie=UTF8&tag=thtaofma-21&linkCode=as2&camp=1634&creative=6738) boards (duh!)
* [7x Class 10 SD Cards](http://www.amazon.co.uk/gp/product/B003VNKNEG/ref=as_li_tf_tl?ie=UTF8&tag=thtaofma-21&linkCode=as2&camp=1634&creative=6738) (2 master cards, 5 for production)
* [1x TP-Link 5-port Ethernet switch](http://www.amazon.co.uk/TP-Link-TL-SF1005D-100Mbps-Unmanaged-Desktop/dp/B000FNFSPY/ref=as_li_tf_tl?ie=UTF8&tag=thtaofma-21&linkCode=as2&camp=1634&creative=6738) and some ancient cables I had lying around (need to build new stripped down ones)
* [5x 6 inch micro USB cables](http://www.amazon.co.uk/gp/product/B003YKX6WM/ref=as_li_tf_tl?ie=UTF8&tag=thtaofma-21&linkCode=as2&camp=1634&creative=6738)
* [1x 5 port USB PSU](http://www.amazon.co.uk/gp/product/B00EKDVGKQ/ref=as_li_tf_tl?ie=UTF8&tag=thtaofma-21&linkCode=as2&camp=1634&creative=6738)
* [0x Ultra-cheap USB Ethernet adapter](http://www.amazon.co.uk/gp/product/B009UOG3NU/ref=as_li_tf_tl?ie=UTF8&tag=thtaofma-21&linkCode=as2&camp=1634&creative=6738) -- NOTE: the ARM driver for this panics the kernel, so I'm back using a Wi-Fi dongle.
* 1x ancient Bondi Blue iMac USB keyboard.
* 1x Custom-printed rack mount base -- I'll eventually draft a "normal" case to hold the boards upright, but for now this does the trick.
* Nx rubber bands (seriously, how else would I manage the cabling)?

## Software

The cluster is now running mostly [Clojure][clj] programs using [Hazelcast][hz] atop JDK 1.8. 

I have also set up [Disco][dp] (and now [Spark][spark]) on it and intend to fiddle with MPI, but am mostly interested in exploring what I can do with an in-memory data grid.

It's a bit ironic to do that on what is essentially 2.5GB of aggregated RAM, but I'm interested in the algorithms themselves and don't plan on doing something silly like tackling the next Netflix Prize with this -- besides, running things on low-end hardware is often the only way to do proper optimization.

### List of packages involved so far:

* [Spark][spark], which has replaced [Disco][dp] for map/reduce jobs.
* [Dash](https://github.com/rcarmo/dash), a real-time status dashboard (rewritten in [Go][golang], available under the `dashboard` folder here, and still being worked on)
* A custom daemon that sends out a JSON-formatted multicast packet with system load, CPU usage and RAM statistics (written in raw C, available in `tools`)
* [ElasticSearch](http://www.elasticsearch.org), which I'm using for playing around with data mining/metrics/etc.
* Oracle [JDK 8](https://jdk8.java.net/download.html)
* [leiningen][lein] (which fetches [Hazelcast][hz] and other dependencies for me, via [this library][clj-hz])
* [Nightcode][nc] as a development environment ([LightTable][lt] doesn't run on ARM, and a lot of my hobby coding these days is actually done on an [ODROID][u2])
* [Moebius Linux][moebius] as a base distro (it's essentially Raspbian without the bloat)
* `distcc` for building binaries slightly faster
* `rpi-update` to get the kernel up to 3.10.x with minimal hassle (need to sort out MPI later)
* `dnsmasq` for DHCP and DNS service on the master to the slaves

Here's what the cluster dashboard looks like:

![First pass at a dashboard](https://raw.github.com/rcarmo/raspi-cluster/master/pics/dash.jpg)


## But isn't the [Raspberry Pi][rpi] slow?

Well spotted, young person. It is, even with the early releases of JDK 1.8. But it's also cheap, and beggars can't be choosers.

But I'm open to [sponsoring][d] so that I can upgrade this to be based on 4 or 5 boards like the [ODROID-U3][u3] (I have an [U2][u2] and love it).

[hz]: http://www.hazelcast.org
[rpi]: http://www.raspberrypi.org
[d]: http://the.taoofmac.com/space/site/Donate
[u2]: http://hardkernel.com/main/products/prdt_info.php?g_code=G135341370451
[u3]: http://hardkernel.com/main/products/prdt_info.php?g_code=G138733896281
[clj]: http://www.clojure.org
[hz]: http://www.hazlecast.org
[nc]: http://nightcode.info
[lt]: http://lighttable.com
[moebius]: http://moebiuslinux.sourceforge.net
[lein]: http://leiningen.org
[clj-hz]: https://github.com/rcarmo/clj-hazelcast
[dp]: http://discoproject.org
[golang]: http://www.golang.org
[spark]: http://spark.apache.org
