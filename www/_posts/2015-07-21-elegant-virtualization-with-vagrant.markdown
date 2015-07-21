---
layout:     post
title:      "Elegant virtualization with Vagrant"
categories: virtualbox lamp
---
[Vagrant](https://www.vagrantup.com/) is an ingenious tool for standardizing virtual machine configuration. Inventor [Mitchell Hashimoto](http://mitchellh.com/) introduces the tool succinctly in [The Tao of Vagrant](http://mitchellh.com/the-tao-of-vagrant). Let's explore the power of **Vagrant** from the point of view of a developer, including legacy project maintenance.

### Using a virtual machine to emulate an old environment

Before discussing **Vagrant**, consider the utility of a basic [virtual machine](https://en.wikipedia.org/wiki/Virtual_machine) (VM) for dealing with legacy projects. Imagine, for example, trying to get a [Drupal 6](https://www.drupal.org/drupal-6.0) site running on your newish laptop. This hypothetical site [relies on PHP 5.2](https://www.drupal.org/requirements/php#6), whereas the [current stable version of PHP](http://php.net/downloads.php) is 5.6.x.

Attempting to coerce your laptop to act like it's 2008 (when D6 was released) may involve bloating your computer with old software, likely resulting in a setup that is subtly and fatally different from the site's actual live environment (e.g. [Ubuntu 12.04 LTS](http://releases.ubuntu.com/precise/)).

[![Drupal 6 released in 2008](/assets/images/2015-07-16-170327-drupal-6-released-in-2008.png "Drupal 6 released in 2008")](/assets/images/2015-07-16-170327-drupal-6-released-in-2008.png)

Of course, this is where virtualization software like [VirtualBox](https://www.virtualbox.org/) comes in handy. You can emulate **Ubuntu 12.04** by fetching the [appropriate ISO](http://releases.ubuntu.com/precise/ubuntu-12.04.5-server-i386.iso) and [setting up a VM using VirtualBox](http://askubuntu.com/questions/142549/how-to-install-ubuntu-on-virtualbox). Then it's simply a matter of installing and configuring [Apache](https://www.apache.org/), [MySQL](https://www.mysql.com/) and [PHP](http://php.net/) (i.e. a classic [LAMP](https://en.wikipedia.org/wiki/LAMP_%28software_bundle%29) server).

This approach is satisfactory, but in practice requires enough effort that you won't want to do it again. **Vagrant** offers a more reproducible, reliable and shareable methodology.

### Introducing Vagrant

Install **Vagrant** and **VirtualBox**. On [my OSX environment]({% post_url 2015-04-17-my-development-machine %}), I used [Homebrew](http://brew.sh/):

{% highlight bash %}
brew install Caskroom/cask/virtualbox
brew install Caskroom/cask/vagrant
{% endhighlight %}

Otherwise, you can consult the [Vagrant download page](http://www.vagrantup.com/downloads.html) and the [VirtualBox download page](https://www.virtualbox.org/wiki/Downloads) respectively. Once installed, run:

{% highlight bash %}
mkdir ubuntu
cd ubuntu
vagrant init ubuntu/precise64
{% endhighlight %}

You should see output like this:

> A \`Vagrantfile\` has been placed in this directory. You are now
> ready to \`vagrant up\` your first virtual environment! Please read
> the comments in the Vagrantfile as well as documentation on
> \`vagrantup.com\` for more information on using Vagrant.

Excluding comments, the new `Vagrantfile` contains:

{% highlight ruby %}
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/precise64"
end
{% endhighlight %}

Note that `ubuntu/precise64` is a reference to **64-bit Ubuntu 12.04 LTS** (codenamed **Precise Pangolin**), one of the [many available Vagrant boxes](https://atlas.hashicorp.com/boxes/search).

Next, let's launch the VM.

{% highlight bash %}
vagrant up
{% endhighlight %}

On the first execution, **vagrant up** will download the "box" (i.e. disk image) and cache it for future reference in `~/.vagrant.d/boxes`. The box size in this case is just **407M** (a bargain compared to a **670M** ISO image). Once the command completes, the VM is running! You can see this in two ways. First, run:
    
{% highlight bash %}
vagrant status
{% endhighlight %}
    
The output should contain:

> running (virtualbox)
    
Secondly, open VirtualBox and see the status "**Running...**" indicated under the "**ubuntu\_...**" machine.

[![VirtualBox running new Vagrant VM](/assets/images/2015-07-15-181056-virtualbox-running-new-vagrant-vm.png "VirtualBox running new Vagrant VM")](/assets/images/2015-07-15-181056-virtualbox-running-new-vagrant-vm.png)

By the way, the **ubuntu** part of the machine name is derived from the directory name where the `Vagrantfile` resides (which we created above).

### Connecting to the VM

By design, [the VM is bare-bones](https://docs.vagrantup.com/v2/boxes/base.html). A single port on your host machine (usually **2222**) is forwarded to the VM's port **22** to permit communication via [SSH](https://en.wikipedia.org/wiki/Secure_Shell). Try it like this:

{% highlight bash %}
vagrant ssh
{% endhighlight %}

> vagrant@vagrant-ubuntu-precise-64:~$

Now that we are *in the VM*, let's observe a couple of interesting things. First of all, run:

{% highlight bash %}
ls /vagrant
{% endhighlight %}

You will notice that `Vagrantfile` is listed! This is, in fact, the very `Vagrantfile` you created earlier. This happens because `/vagrant` is a [synced folder](http://docs.vagrantup.com/v2/synced-folders/index.html) that reflects the **ubuntu** directory in the host machine.

Next, run:

{% highlight bash %}
ifconfig
{% endhighlight %}

>     eth0      Link encap:Ethernet  HWaddr 08:00:27:18:10:7f
>               inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
>               inet6 addr: fe80::a00:27ff:fe18:107f/64 Scope:Link
>               UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
>               RX packets:469 errors:0 dropped:0 overruns:0 frame:0
>               TX packets:315 errors:0 dropped:0 overruns:0 carrier:0
>               collisions:0 txqueuelen:1000
>               RX bytes:49992 (49.9 KB)  TX bytes:41647 (41.6 KB)

Under **eth0** (the ethernet device), find **inet addr** which indicates VM's IP address (e.g `10.0.2.15`).

Now, exit the VM:

{% highlight bash %}
exit
{% endhighlight %}

Back in the host machine, try reaching the VM via [ping](https://en.wikipedia.org/wiki/Ping_%28networking_utility%29). This pings the VM exactly 4 times:

{% highlight bash %}
ping -c 4 10.0.2.15
{% endhighlight %}

>     PING 10.0.2.15 (10.0.2.15): 56 data bytes
>     Request timeout for icmp_seq 0
>     Request timeout for icmp_seq 1
>     Request timeout for icmp_seq 2
>     Request timeout for icmp_seq 3
>     
>     --- 10.0.2.15 ping statistics ---
>     4 packets transmitted, 0 packets received, 100.0% packet loss

The VM is unreachable because, by default, networking is configured in [Network Address Translation (NAT)](http://www.virtualbox.org/manual/ch06.html#network_nat) mode. This creates a closed network between *VirtualBox* and the VM. Let's change that, because it'll be convenient to access the D6 site directly via a browser on the host machine.

Edit the `Vagrantfile` so that it uses a [private network](http://docs.vagrantup.com/v2/networking/private_network.html) (using [DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol)).

{% highlight ruby %}
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/precise64"
  config.vm.network "private_network", type: "dhcp"
end
{% endhighlight %}

To make these changes take effect, run:

{% highlight bash %}
vagrant reload
{% endhighlight %}

This reboots the machine and re-interprets the `Vagrantfile`. Next, log back in with SSH and re-run **ifconfig**:

{% highlight bash %}
vagrant ssh
ifconfig
{% endhighlight %}

>     eth1      Link encap:Ethernet  HWaddr 08:00:27:00:82:c4
>               inet addr:172.28.128.3  Bcast:172.28.128.255  Mask:255.255.255.0
>               inet6 addr: fe80::a00:27ff:fe00:82c4/64 Scope:Link
>               UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
>               RX packets:4 errors:0 dropped:0 overruns:0 frame:0
>               TX packets:16 errors:0 dropped:0 overruns:0 carrier:0
>               collisions:0 txqueuelen:1000
>               RX bytes:1830 (1.8 KB)  TX bytes:2104 (2.1 KB)

Notice that there is a new virtual network card called **eth1** with a new **inet addr** (e.g. `172.28.128.3`). 

Now, exit the VM:

{% highlight bash %}
exit
{% endhighlight %}

Try to ping the VM again:

{% highlight bash %}
ping -c 4 172.28.128.3
{% endhighlight %}

>     PING 172.28.128.3 (172.28.128.3): 56 data bytes
>     64 bytes from 172.28.128.3: icmp_seq=0 ttl=64 time=0.422 ms
>     64 bytes from 172.28.128.3: icmp_seq=1 ttl=64 time=0.339 ms
>     64 bytes from 172.28.128.3: icmp_seq=2 ttl=64 time=0.385 ms
>     64 bytes from 172.28.128.3: icmp_seq=3 ttl=64 time=0.363 ms
>
>     --- 172.28.128.3 ping statistics ---
>     4 packets transmitted, 4 packets received, 0.0% packet loss 

Success! The VM is reachable. Another way to get the VM's IP addresses, is to run:

{% highlight bash %}
vagrant ssh -c "hostname -I"
{% endhighlight %}

Note that both addresses are returned:

> 10.0.2.15 172.28.128.3

### Maintaining a hosts file

To avoid the inconvenience of referring to a VM by its IP address, we can associate it with a friendly name via a [hosts file](https://en.wikipedia.org/wiki/Hosts_%28file%29). A Vagrant plugin called [hostmanager](https://github.com/smdahlen/vagrant-hostmanager) makes this easy:

{% highlight bash %}
vagrant plugin install vagrant-hostmanager
{% endhighlight %}

In the `Vagrantfile`, set **ubuntu.example.com** as the host name and configure the **hostmanager** plugin. See comments below for reference.

{% highlight ruby %}
Vagrant.configure(2) do |config|
  # Host name of the VM
  config.vm.hostname = "ubuntu.example.com"

  # Base image for VM
  config.vm.box = "ubuntu/precise64"

  # Configure private network by DHCP
  config.vm.network "private_network", type: "dhcp"

  # Update /etc/hosts on all active VMs
  config.hostmanager.enabled = true

  # Update host machine's /etc/hosts
  config.hostmanager.manage_host = true

  # Don't ignore private IPs
  config.hostmanager.ignore_private_ip = false

  # Include offline VMs (rather than just active ones)
  config.hostmanager.include_offline = true

  # Use IP resolver to get DHCP configured address
  config.hostmanager.ip_resolver = proc do |vm, resolving_vm|
    `vagrant ssh -c "hostname -I"`.split.last
  end
end
{% endhighlight %}

To update the **hosts** file, destroy the machine and rebuild it. You will likely be prompted for your password so that **hostmanager** can update your **hosts** file.

{% highlight bash %}
vagrant destroy
vagrant up
{% endhighlight %}

Now examine the tail of the hosts file:

{% highlight bash %}
tail /etc/hosts
{% endhighlight %}

>     ## vagrant-hostmanager-start id: 1d6b1120-e125-4d26-82a8-559208175336
>     172.28.128.9	ubuntu.example.com
>     ## vagrant-hostmanager-end

This shows that **ubuntu.example.com** has been successfully associated with the DHCP-assigned IP address of the VM. You can even ping it.

{% highlight bash %}
ping -c 4 ubuntu.example.com
{% endhighlight %}

### Configuring a LAMP server

Check what happens if you navigate to [ubuntu.example.com](http://ubuntu.example.com/) in your browser.

[![The VM without LAMP](/assets/images/2015-07-17-183919-the-vm-without-lamp.png "The VM without LAMP")](/assets/images/2015-07-17-183919-the-vm-without-lamp.png)

The connection is refused because there is no webserver running. So let's install [Apache](https://www.apache.org/):

{% highlight bash %}
vagrant ssh -c "sudo apt-get -y install apache2"
{% endhighlight %}

Now the browser should show a more promising page:

[![Apache2 installed on VM](/assets/images/2015-07-20-173527-apache2-installed-on-vm.png "Apache2 installed on VM")](/assets/images/2015-07-20-173527-apache2-installed-on-vm.png)

Let's install the other elements of LAMP. Below, the `DEBIAN_FRONTEND` environment variable ensures that you are not prompted for anything during installation, including setting a MySQL root password. **Security warning: With this method, the MySQL root password will be blank. This is handy for local development, but not desirable otherwise.**

{% highlight bash %}
vagrant ssh -c "sudo DEBIAN_FRONTEND=noninteractive \
                apt-get -y install libapache2-mod-php5 \
                php5-mysql mysql-server"
{% endhighlight %}

Create a local `www` directory containing a dummy PHP file.

{% highlight bash %}
mkdir www
echo "<?php echo 'Hello from PHP!';" > www/index.php
{% endhighlight %}

Add the following to the `Vagrantfile` (before the final **end**):

{% highlight ruby %}
# Sync local www directory with /var/www on the VM
config.vm.synced_folder "www", "/var/www", owner: "www-data"
{% endhighlight %}

This simply mounts our local **www** directory where Apache looks for the default website. Note that the owner of the directory (within the context of the VM) will be **www-data**, the Apache user. Run **reload** to effect the changes:

{% highlight bash %}
vagrant reload
{% endhighlight %}

Now in the browser, instead of "It works!", you should see the greeting from PHP.

[![LAMP with basic PHP](/assets/images/2015-07-20-180314-lamp-with-basic-php.png "LAMP with basic PHP")](/assets/images/2015-07-20-180314-lamp-with-basic-php.png)

Next, modify PHP script and observe that the change immediately appears in the browser (upon refresh) because the file is synced to `/var/www` on the VM.

{% highlight bash %}
echo "<?php phpinfo();" > www/index.php
{% endhighlight %}

[![LAMP with phpinfo()](/assets/images/2015-07-20-180647-lamp-with-phpinfo.png "LAMP with phpinfo()")](/assets/images/2015-07-20-180647-lamp-with-phpinfo.png)

Finally, [download the latest 6.x version of Drupal](https://www.drupal.org/node/3060/release?api_version[]=87). Extract into `www`, overwriting the test `index.php` file. The browser should show the standard **Drupal** installation screen.

[![LAMP with Drupal6](/assets/images/2015-07-20-180934-lamp-with-drupal6.png "LAMP with Drupal6")](/assets/images/2015-07-20-180934-lamp-with-drupal6.png)

If you want to proceed with the installation, you'll need to create a database:

{% highlight bash %}
MY_SQL_COMMAND="CREATE DATABASE drupal; \
                GRANT ALL ON drupal.* TO drupal@localhost \
                IDENTIFIED BY 'password';"
vagrant ssh -c "mysql -u root -e '$MY_SQL_COMMAND'"
{% endhighlight %}

This creates a MySQL user called `drupal` and grants it all permissions on database called `drupal`, authenticated with an **insecure** password `password`. If you want to, you can proceed with the instructions in the browser to complete the setup of Drupal.

[![LAMP with Drupal6 - Installed](/assets/images/2015-07-20-182356-lamp-with-drupal6-installed.png "LAMP with Drupal6 - Installed")](/assets/images/2015-07-20-182356-lamp-with-drupal6-installed.png)

### Refactoring into a provisioning script

In the same directory as the `Vagrantfile`, create a new file called **provision**, containing the below:

{% highlight bash %}
#! /bin/bash

# Exit with failure immediately if any command fails
set -e

# Install Apache
sudo apt-get -y install apache2

# Install PHP and MySQL with Apache bindings
sudo DEBIAN_FRONTEND=noninteractive \
     apt-get -y install libapache2-mod-php5 \
     php5-mysql mysql-server

# Create a dummy Drupal database
MY_SQL_COMMAND="CREATE DATABASE drupal; \
                GRANT ALL ON drupal.* TO drupal@localhost \
                IDENTIFIED BY 'password';"
mysql -u root -e '$MY_SQL_COMMAND'

# Restart Apache
/etc/init.d/apache2 restart
{% endhighlight %}

Add the following line to the `Vagrantfile`, to instruct **Vagrant** to execute the **provision** script on the VM during provisioning.

{% highlight ruby %}
# Use shell script for provisioning
config.vm.provision "shell", inline: File.read('provision'), privileged: false
{% endhighlight %}

(Note that **file:** can be used instead of **inline:**, as per the [shell provisioning documentation](https://docs.vagrantup.com/v2/provisioning/shell.html), but the latter makes debugging easier, since it forces an update with every `vagrant reload --provision`.)

Now, *for the moment of truth*, destroy and recreate **your entire VM** with just two commands:

{% highlight bash %}
vagrant destroy
vagrant up
{% endhighlight %}

For reference, this is the final `Vagrantfile`:

{% highlight ruby %}
# vi: set ft=ruby :
Vagrant.configure(2) do |config|
  # Host name of the VM
  config.vm.hostname = "ubuntu.example.com"

  # Base image for VM
  config.vm.box = "ubuntu/precise64"

  # Configure private network by DHCP
  config.vm.network "private_network", type: "dhcp"

  # Update /etc/hosts on all active VMs
  config.hostmanager.enabled = true

  # Update host machine's /etc/hosts
  config.hostmanager.manage_host = true

  # Don't ignore private IPs
  config.hostmanager.ignore_private_ip = false

  # Include offline VMs (rather than just active ones)
  config.hostmanager.include_offline = true

  # Use IP resolver to get DHCP configured address
  config.hostmanager.ip_resolver = proc do |vm, resolving_vm|
    `vagrant ssh -c "hostname -I"`.split.last
  end

  # Sync local www directory with /var/www on the VM
  config.vm.synced_folder "www", "/var/www", owner: "www-data"

  # Use shell script for provisioning
  config.vm.provision "shell", inline: File.read('provision'), privileged: false
end
{% endhighlight %}

### Next steps

There's a wealth of other [boxes listed on Vagrantbox.es]( http://www.vagrantbox.es/). For example, you can use **Vagrant** to fire up a [Windows 8.1](https://en.wikipedia.org/wiki/Windows_8.1) box [supplied by Microsoft](http://dev.modern.ie/).

{% highlight ruby %}
# vi: set ft=ruby :
Vagrant.configure('2') do |config|
  # Supply name and URL for box
  config.vm.box = 'win81'
  config.vm.box_url = 'http://aka.ms/vagrant-win81-ie11'

  # Enable GUI
  config.vm.provider "virtualbox" do |vm|
    vm.gui = true
  end
end
{% endhighlight %}

Suffice to say that since [Mitchell's first Vagrant commmit](https://github.com/mitchellh/vagrant/commit/050bfd9c686b06c292a9614662b0ab1bbf652db3) in 2010, the project has very quickly become an indispensable tool in any developer's tool belt. The project's activity remains hot, so we can expect more great things to come.

[![Vagrant project activity on Github](/assets/images/2015-07-14-184104-vagrant-project-activity-on-github.png "Vagrant project activity on Github")](/assets/images/2015-07-14-184104-vagrant-project-activity-on-github.png)
