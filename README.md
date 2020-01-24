# What is...?

There are a lot of IT professionals out there who think they make simple concepts sound obvious, but they don't. I can't tell you how many times I wish someone had made a simple, catch-all, easy-to-understand definition of some hot new product out there.  Well, here's my attempt.  Note: I will generalize a lot, because I want people to know CONCEPTS before they worry about exceptions and details.

## What is Linux?

Linux is a kernel.  Think of it like an engine.  By itself, not very useful.  But add a handlebar, wheels, gas tank, and a seat, and you got yourself a vehicle.  

## What is a Linux distribution?

When you buy a motorcycle from a dealership, it's already got essential accessories like handlebars, wheels, gas tank, seat, and everything else.  Pay for it, put gas in it, drive off.  You don't have to worry about afterparts, unless you want to.  The dealership is the company like Red Hat, Canonical, Debian, Slackware, and so on.  How they package the accessories, like drivers (modules), a shell (bash, zsh), package managers (deb, rpm), a desktop enviroment (GNOME, KDE), and so on is called "the distribution."  Most of these companies give away the distributions for free, and sell you support packages.  You don't have to pay for it unless you want them to do everything for you.  Most of us Linux folks do things for ourselves.  Some of us make distributions that don't even sell support.

* Linux kernel = engine
* Everything else = accessories
* Operating system = distribution

That was it for a while. A motorcycle usually only takes one passenger: the driver. 

## What is docker?

Suppose you want to take a second passenger?  You could build a whole second motorcycle and attach it, but that's kind of stupid. For a second passenger, all you needs it a place for them to sit: a sidecar.  The sidecar has a windshield, an extra wheel for stability, and a seat.  But no engine, handlebars, or gas tank.  It doesn't need one.  It just needs to be attached to the engine.

Docker is a sidecar, a container for the extra passenger

In fact, the argument could be made, "well, just let them hop on the back of my already existing seat."  This is also true.  Then all they need is a helmet and a good grip.  But for now, think of Docker as a side car: all that's needed for one extra person and none of what's not needed.

Do you know about chroot?  That's all Docker is.  In fact, it's behind LXC and OpenVZ, too.  Now, when you use chroot in FTP, you have a restricted "jail."  You might have thought that the jail just prevents you from going back a few directories, and it does, but it can be used for so much more.

Imagine a Linux distribution that only has the libraries and configs for a single application.  A normal distribution will have drivers, libraries, shells, and everything else a normal human would need.  But if you just want to run nginx web server with one config point to an application, you don't need anything else, really. Why waste a whole second server for that?

*"Okay, well, why not just run nginx on the host? Why add a docker layer?"*

Now imagine you have a developer, maybe several developers, working on a web application.  It needs nginx, python, maybe a wee small database.  Developer 1 has Ubuntu 18.04.  Dev 2 has CentOS 7. Dev 3 has a Mac. Dev 4 has Ubuntu 16.04.  They can "get the application to work just fine on my system," but it's not very portable. What runs on a Mac is very different than what runs on Ubuntu with file locations, memory, libraries, and so on.  It would be better if there was a standard little box that ran on everyone's system.  But instead of dealing with a virtual OS, like Virtualbox, VMware, or whatever, all they need is a container with nginx, python 3.7, and redis.

Docker is a "container" with ONLY what the application needs, and nothing superfluous that it doesn't.  Just like a sidecar doesn't need its own engine, gas tank, or handlebars.  It just needs a motorcycle to attach to.  You can also have different versions of nginx and code running at the same time without overwriting the config files of the others.

*"Well, how does this differ from just some virtual machine I can pass around?"*

Docker containers are very small, very portable (if the system can run docker, it can run the app), and run by being a chrooted environment piggybacking onto the Linux distribution you already have going.  But chrooting everything by yourself is complicated, so docker takes care of all of that for you. Docker just uses the same kernel you're using, makes a chrooted environment and then adds its own libraries, config files, programs, shells, and so on... *but only enough to run what you need for that container.*   Same with LXC, OpenVZ, and so on.  A virtual machine can do this, too, but it has a ton of extra stuff you probably don't need at a cost of memory, power, and complexity.

* Just need the app to run on any old kernel?  Container.
* Need a specific or different kernel than the host?  Virtual machine.

Docker is versioned, too. So if they make a change to the nginx config, and suddenly gets a 504 error to redis, you can "roll back" to the last version and fire that back up. You can store containers in version on a docker hub, like github.  Suddenly, containers are as flexible as software.

Another advantage is that you can have more than one docker container running on the same host.  In most cases, one container would run the nginx server with the python code, and another container would run redis. It can be as complex or as granular as you like, but for purposes of sanity, having each as a separate container is best (again, for rolling back).  So any changes Dev 2 and 3 make with redis, if it fails, we roll back to the previous redis setup.  When dev 1 and 4 change nginx or some python code that acts funny, you know what parts changed.

And a BIG advantage of multiple containers on the same host means that the host starts to become irrelevant. Some companies sell you "container holding services" like AWS has EKS, Google has GKS, and Azure has AKS. The K stands for Kubernetes, which we'll get to in a moment.

Now, suppose you have the "app" like before: a few small pieces that run together. Suppose you run a ticketing service that uses the app.  People log in, buy tickets, select seats, select VIP options, and get a QCode that can be scanned at the gate.

Your app sits idle 99% of its life. Most of the time, it's just waiting for people to log in.  On a normal day, it gets 100-200 logins an hour, mostly between the hours of 9am and 5pm. But occasionally, some new tickets are in huge demand, and the service gets flooded, like 100-200 logins a minute. Your app is designed to have many in place at the same time.  So when the single app gets overwhelmed, you can turn on more.  Maybe 4 more, or 10 more.  You'd need a load balancer, too, so the login will know which container is free or has low load.  Then you need to monitor that load, add the new container to the load balancer, and what a pain.  Especially since you don't know when this flood will be.  It could be 3am on a Sunday.

You could just have 10 containers running all the time.  You guess "our max load is 10 containers worth, so make sure that's always up."  But even 1 container is idle 99% of the time.  You're paying for those apps to be running.  Now you're paying more because once in a while, it gets busy, but you don't know when.

What if there was a way to AUTOMATICALLY say, "when the container starts running a lot, before it times out, fire up a new instance in response and add it to the load balancer.  If the container is not running a lot, shut it down, and remove it from the load balancer."

You need an orchestrator, like a conductor of an orchestra combined with a traffic cop.

## What is Kubernetes/k8s?

Kubernetes is an orchestrator.

Kubernetes, called "k8s" for short because there are 8 letters between the "k" and "s," (I don't get it either) is a system that can do all of this and more.  But let's look at a simple example.

You have a k8s setup with one app, one load balancer, and a few other tools. You tell k8s "I want two containers running MINIMUM."  That's in case one crashes, or you're updating one while the other continues to run. "If both containers get 50% busy, fire up a new container, and add it to the pool."  Okay, but what about after some huge event?  "If a container has been idle for 2 or more minutes, shut it down.  And keep shutting idle containers down until there's the 2 minimum left." Yes, you can also set maximums.  Or scheduled increases and decreases.  You can define "busy," like network connections, response time, CPU utilization, and so on.  Sky's the limit, man.

Kubernetes have different pieces: nodes (hosts), pods (containers on hosts), services (your application on the fleet of containers), and networks (the network bundle that controls the ins an outs to the services).

So now you have a police dispatcher (k8s) who knows when police motorcycles (nodes) and sidecars (pods) need to go to a situation (services) in a city (network).

Comparing it to the old way of doing things, this is like having a data center that hooks up computers, network equipment, and software configurations on the fly as needed, and then turns them off when they are not needed. And docker means that they have a bunch of old computers and configurations lying about in case they need to use those, too. Imagine what that service would have cost back in the day!

So in summary:
- Linux kernel is just an engine
- Linux distribution is a whole motorcycle: an engine with accessories.
- Docker is a container: or a motorcycle sidecar that only has what an extra passenger needs to use the motorcycle
- Kubernetes is an orchestrator: or a police dispatch that sends a fleet of police motorcycles and sidecars that are used only when needed.
