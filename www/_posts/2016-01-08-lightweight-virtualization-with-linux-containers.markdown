---
layout:     post
title:      "Lightweight virtualization with Linux containers"
categories: lxc docker
---
[Containerization](http://www.webopedia.com/TERM/C/containerization.html) is a kind of [virtualization](https://en.wikipedia.org/wiki/Virtualization) that's surprisingly good at crowding Linux virtual machines onto hosts, including humble home servers. In this post, we'll look at [Linux containers](https://linuxcontainers.org/) (LXC), a powerful virtualization technology leveraged by [Docker](https://www.docker.com/), the [billion-dollar company](http://www.bloomberg.com/news/articles/2015-04-14/docker-said-to-join-1-billion-valuation-club-with-new-funding) that made containerization famous.

## Overview

Containerization can be understood in contrast to [hardware virtualization](https://en.wikipedia.org/wiki/Hardware_virtualization), in which a whole machine is emulated, complete with CPU, memory, networking and a hard disk. The latter approach, used by [VirtualBox](https://www.virtualbox.org/) and [Amazon's EC2](https://aws.amazon.com/ec2/), conveniently runs any operating system on the virtual hardware, regardless of the host operating system.

Unfortunately, the alchemy of hardware virtualization can [consume 10-30%](http://serverfault.com/questions/261974/how-much-overhead-does-x86-x64-virtualization-have) of the host machine's raw computational power, depending on the task at hand. In terms of density (i.e. [maximum virtual machines on given hardware](http://www.zdnet.com/article/virtual-machine-density-can-be-a-virtualization-deal-breaker/)), hardware virtualization is severely limited by the extent to which you want to guarantee minimum performance on each instance. In practice, I've found that running more than a few **VirtualBox** machines on a consumer-grade computer is untenable.

Containers, on the other hand, can virtualize *orders of magnitude* more machines by deliberately reusing elements of the host operating system. Specifically, **LXC** ([pronounced **lex-cee**](http://www.ubuntu.com/cloud/lxd)) reuses the host's [kernel](https://en.wikipedia.org/wiki/Linux_kernel), the central core of the operating system, as stated in the [introduction to LXC](https://linuxcontainers.org/lxc/introduction/): 

> The goal of LXC is to create an environment as close as possible to a standard Linux installation but without the need for a separate kernel.

The main catch is that you can only virtualize host-kernel-compatible Linux machines with this technique. The other catch is that security (i.e. preventing container escape) is a bit [harder to get right](http://andrea.corbellini.name/2015/02/20/are-lxc-and-docker-secure/).

## Creating a networked container

In setting up **LXC** on a [Ubuntu 14.04.3 LTS (Trusty Tahr)](http://releases.ubuntu.com/trusty/) server, I'm referring to the [Ubuntu LXC server guide](https://help.ubuntu.com/lts/serverguide/lxc.html). It's worth noting that **Ubuntu** is *officially* the best OS for the job, as stated in **LXC**'s [Getting Started](https://linuxcontainers.org/lxc/getting-started/) reference:

> Ubuntu is also one of the few (if not only) Linux distributions to come by default with everything that's needed for safe, unprivileged LXC containers.

The first step is to install **LXC**:

{% highlight bash %}
sudo apt-get install lxc
{% endhighlight %}

You *could* now simply run **sudo lxc-create** to create a *privileged* container, in other words, a container owned by the root user. However, this incurs the unnecessary risk of compromising the superuser account, should an attacker escape the container. As discussed in [LXC security](https://linuxcontainers.org/lxc/security/): 

> [Privileged containers are] not safe at all and should only be used in environments where unprivileged containers aren't available and where you would trust your container's user with root access to the host.

Creation of a so-called *unprivileged* container (i.e. owned by a regular user), is enabled by [subordinate users](http://man7.org/linux/man-pages/man5/subuid.5.html) and [subordinate groups](http://man7.org/linux/man-pages/man5/subgid.5.html). On a newish Ubuntu installation, every user has been assigned a reserved range of subordinate user and group ids:

{% highlight bash %}
cat /etc/subuid
cat /etc/subgid
{% endhighlight %}

To discover the ranges for the current user:

{% highlight bash %}
MY_SUBUID_RANGE=`grep "^$USER:" /etc/subuid | sed 's/\w\+://' | sed 's/:/ /'`
MY_SUBGID_RANGE=`grep "^$USER:" /etc/subgid | sed 's/\w\+://' | sed 's/:/ /'`
echo "My (user=$USER) sub-user range is $MY_SUBUID_RANGE and my sub-group range is $MY_SUBGID_RANGE"
{% endhighlight %}

You should see output like:

> My (user=ubuntu) sub-user range is 100000 65536 and my sub-group range is 100000 65536

This means that the user **ubuntu** reserves the right to use 65,536 ids beginning with `100000`. I think this means the actual range is 100000-165535, but in any case, it's a lot of ids. **LXC** needs a new subordinate user to each new container, as we'll see later.

You can now write the necessary **LXC** config file:

{% highlight bash %}
mkdir -p ~/.config/lxc

cat << EOF > ~/.config/lxc/default.conf
# Map root uid (0) gid (0) to subordinate ranges
lxc.id_map = u 0 $MY_SUBUID_RANGE
lxc.id_map = g 0 $MY_SUBGID_RANGE

# Network configuration
lxc.network.type = veth
lxc.network.link = lxcbr0
EOF

cat ~/.config/lxc/default.conf
{% endhighlight %}

The above maps **root** (which has a uid `0` and a gid of `0`) to the subordinate user and group ranges discovered above. It also instructs **LXC** to create a virtual network interface using [bridged networking](https://en.wikipedia.org/wiki/Bridging_%28networking%29).

Next, we need to allow the user to create new virtual network interfaces:

{% highlight bash %}
echo "$USER veth lxcbr0 10" | sudo tee -a /etc/lxc/lxc-usernet
{% endhighlight %}

Finally, you can create a container:

{% highlight bash %}
lxc-create -n ubuntu_example -t download -- --dist ubuntu --release trusty --arch amd64
{% endhighlight %}

The `-n` parameter specifies an arbitrary name for the container. The `-t` parameter indicates the container template, which refers to one of the scripts found in  `/usr/share/lxc/templates`. At the moment, the choices are:

* alpine
* altlinux
* archlinux
* busybox
* centos
* cirros
* debian
* **download**
* fedora
* gentoo
* openmandriva
* opensuse
* oracle
* plamo
* sshd
* **ubuntu**
* ubuntu-cloud

By the way, these are *non-trivial shell scripts*. For example, the **ubuntu** template, which lives at `/usr/share/lxc/templates/lxc-ubuntu`, is almost 800 lines long. It actually does a from-scratch installation (using [debootstrap](https://wiki.debian.org/Debootstrap)), while the **download** template script (about 600 lines long) grabs and extracts a pre-built disk image from **linuxcontainers.com**. 

Parameters after `--` are passed to the template script. In the command above, **download** is being instructed to get 64-bit Trusty Tahr from a URL that (today) resolves to `http://images.linuxcontainers.org/images/ubuntu/trusty/amd64/default/20160105_03:49/`.

![Latest trusty root image](/assets/images/2016-01-05-181517-latest-trusty-root-image.png "Latest trusty root image")

Incidentally, **download** is the only template that currently works for an unprivileged user. You can experiment with the others, but you'll likely get an error like:

> This template can't be used for unprivileged containers.
> You may want to try the "download" template instead.

## Managing containers

Once a container is created, you can recall its name and status by running:

{% highlight bash %}
lxc-ls --fancy
{% endhighlight %}

The root file system for the container is located at `~/.local/share/lxc/ubuntu_example/rootfs`:

{% highlight bash %}
du -sh ~/.local/share/lxc/ubuntu_example/rootfs
{% endhighlight %}

This shows that the ubuntu installation is **389M**.

Now, start the container by running:

{% highlight bash %}
lxc-start -n ubuntu_example -d
{% endhighlight %}

The `-d` parameter runs it as a daemon (i.e. in the background). If you don't do that, your terminal will get taken over by the booting container.

If you get an error like the following, the user probably does not have the correct [cgroups](https://en.wikipedia.org/wiki/Cgroups).

> lxc_container: lxc_start.c: main: 341 The container failed to start.

To correct the issue, just reboot and try again.

{% highlight bash %}
sudo shutdown -r now
{% endhighlight %}

Once you've successfully started the container, run **lxc-ls** again:

{% highlight bash %}
lxc-ls --fancy
{% endhighlight %}

The output should include a status like this:

> ubuntu_example  RUNNING  10.0.3.233

Login to the container by running:

{% highlight bash %}
lxc-attach -n ubuntu_example
{% endhighlight %}

> root@ubuntu_example:/#

From here, you can install software with `apt-get` or set a password with `passwd` and so forth. For example, we can install a web server like [nginx](https://www.nginx.com/resources/wiki/).

{% highlight bash %}
apt-get install nginx
{% endhighlight %}

Then return to the host machine:

{% highlight bash %}
exit
{% endhighlight %}

Visit to the IP address above with a browser or use `curl`:

{% highlight bash %}
curl 10.0.3.233
{% endhighlight %}

If everything is working correctly, the result will include:

> Welcome to nginx!

As mentioned earlier, **LXC** is using subordinate users and groups to keep containers isolated. To see this in action, run **top** on the host computer (substituting `100000` for your first subordinate uid).

{% highlight bash %}
top -U 100000
{% endhighlight %}

You should see all the processes running on the container (note the newly installed **nginx**);

![Running top for container processes](/assets/images/2016-01-08-062211-running-top-for-container-processes.png "Running top for container processes")

It's worth emphasizing that these processes are transparently *running on the host*, using the host's kernel, which is why they're visible on the host's **top**. This is different from the typical hardware-virtualized machine scenario, in which the processes are *invisible to the host*. Press `q` to quit **top**.

To stop the container, run:

{% highlight bash %}
lxc-stop -n ubuntu_example
{% endhighlight %}

To destroy the container and **delete all associated data**, run:

{% highlight bash %}
lxc-destroy -n ubuntu_example
{% endhighlight %}

## Next steps

The idea of containers, while [not exactly new](www.itworld.com/virtualization/416864/containers-bring-skinny-new-world-virtualization-linux) (apparently around since 2000), has skyrocketed in popularity over the past two years due to [the success of Docker](http://www.zdnet.com/article/what-is-docker-and-why-is-it-so-darn-popular/) and fuelled by the maturation of containerization technology like **LXC**, which reached a [major milestone](https://www.stgraber.org/2013/12/20/lxc-1-0-blog-post-series/) (version 1.0) [in early 2014](https://linuxcontainers.org/lxc/news/#lxc-100-release-announcement-20th-of-february-2014).

The [Google Trend](https://www.google.com/trends/explore#q=%2Fm%2F0wkcjgj%2C%20%2Fm%2F0crds9p%2C%20%2Fm%2F02t3gg%2C%20%2Fm%2F0272hgj&cmpt=q&tz=Etc%2FGMT%2B5) for **Docker** (see the blue line) compared to a couple other virtualization solutions ([Xen](http://www.xenproject.org/) and [KVM](http://www.linux-kvm.org/page/Main_Page)), shows rising popularity of the former since 2014. **LXC** seems to fly a bit under the radar.


![Docker popularity skyrocketing since 2013](/assets/images/2016-01-07-183540-docker-popularity-skyrocketing-since-2013.png "Docker popularity skyrocketing since 2013")

**Docker**, a brilliant platform that makes it convenient to deploy software via containers, is worth a detailed exploration another time. Meanwhile, here's [a good summary of containerization and Docker](https://jaxenter.com/containerization-vs-virtualization-docker-introduction-120562.html) and a [great StackOverflow answer about how containers are different from "normal" virtual machines](http://stackoverflow.com/questions/16047306/how-is-docker-different-from-a-normal-virtual-machine).

Finally, if you're concerned about container security, here's some additional reading:

* [LXC security on Ubuntu LTS Server Guide](https://help.ubuntu.com/lts/serverguide/lxc.html#lxc-security)
* [Ubuntu's wiki article on LXC security](https://wiki.ubuntu.com/LxcSecurity)
* [Container security in the Docker context](https://blog.docker.com/2013/08/containers-docker-how-secure-are-they/)
* [Docker best practices for container security](http://linux-audit.com/docker-security-best-practices-for-your-vessel-and-containers/)
