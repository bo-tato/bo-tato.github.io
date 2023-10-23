# LXD for security research

Doing security research we are constantly setting up local installations of
software we are testing, and running many scripts and utilities. To avoid
risking or polluting our computer with this, we do most things isolated in
virtual machines or containers. I've found LXD to be quite convenient for this.

## Unified interface for containers and virtual machines

Containers are nice because they start almost instantly and use much less memory
compared to VMs. But sometimes we need VMs in order to run windows, or because
we need the stronger security and isolation they provide. In LXD configuration
of networking, transferring files, and basically everything else is identical
for dealing with a container or a VM, so it makes it trivial to set up a mixed
lab network of containers and VMs in the same private network.

## vs Docker (for containers)

Most stuff you can do with LXD containers you can do with docker also, but by
default I think LXD is nicer for this use case where we basically are using
containers as lighter weight virtual machines, rather than as a way to bundle and
deploy a single application which is more the use case docker targets.

### Security

For containers LXD provides better security by default as root and other users
inside a container map to users with no permissions in the host system. In
docker by default, uid 0 (root) in a container is uid 0 on the host, just with a
restricted view of the filesystem and other restrictions. This can be
[configured in docker.](https://tbhaxor.com/prevent-container-breakout-privilege-escalation-via-userns-remap/)

But when we do something like run a malware sample to analyze it's behavior, a
container is not safe enough. Sometimes we need the stronger isolation provided
by a virtual machine with a separate kernel. LXD makes this easy. Whatever
commands we were using to set up and configure the container, we just add the
`--vm` switch to `lxc launch` and we'll get a VM instead.

### Fast snapshots

In LXD the default storage pool uses btrfs, which allows for very fast snapshot
and clones. This is great when we want to try out some tools without permanently
installing them, we just try them out in a clone we'll delete after. Docker can
also be configured to use btrfs for storage but it's default overlayfs is far
slower at this.

### Init system

By default LXD will run a normal systemd init system inside a container, while
docker by default will run your application as the init process. This makes LXD
containers easier to use as a more lightweight VM alternative.

## Wifi tools in containers

We sometimes want to run tools like wifite, airodump, etc that require a
wireless device in monitor mode. As in LXD the root user in the container is a
user with no permissions outside, even if we put the wireless device in the
container, root in the container won't have permission to set it to monitor
mode. This is simple enough to do outside the container:
```sh
# create a profile with the wireless device
lxc profile create antenna
lxc profile device add antenna antenna nic nictype=physical parent=wlan0 name=mon0

# set the device on the host to monitor mode
ip link set wlan0 down
iwconfig wlan0 mode monitor
```

Now we can launch a kali container and attach this profile to it:
```sh
lxc launch images:kali kali
lxc profile add kali antenna
```

The kali container now has a mon0 device in monitor mode and tools like wifite
will work fine in the container despite not having real root access.

When the container no longer needs wifi monitor mode we can remove the profile:
```sh
lxc profile remove kali antenna
```

## Xorg in containers

Often we want to run GUI applications in containers and have them seamless in
our desktop as just another program. Just sharing our Xorg socket into the
container would be horribly insecure as any program in X can take screenshots,
access the clipboard, and even record and inject keystrokes to other
applications.

### X11 Security Extension

You can use xauth to generate cookies to allow programs to connect to X with
less permissions. They won't be able to record or inject keystrokes of your
normal programs, or take screenshots. They can however still read your clipboard
although they can't write to it. But the biggest problem is it doesn't provide
isolation between your different containers. All programs running in X with
"untrusted" cookies can inject or record keystrokes of each other, just not of
the "trusted" windows. I also had problems with flickering as it seems with the
security extension, untrusted programs didn't have access to the double buffer
extension.

### Wayland

It's often stated that Wayland is more secure that Xorg and it's kind of true in
that pure Wayland doesn't provide access to clipboard, screenshots, and
keystrokes. But almost everything in Wayland is implemented by compositors like
wlroots that do give access to the
[clipboard](https://github.com/swaywm/sway/issues/4511),
[screen](https://github.com/swaywm/sway/issues/5118), and
[keyboard](https://github.com/swaywm/sway/issues/5555). There is a proposal for
wayland to allow different "security contexts" so that containerized programs
could use wayland in a secure way without that access, but it is just in the
proposal stage.

### Xpra

This is the nicest tool I found, allowing to give each container an isolated X
display, and still have all the programs seamless in your desktop environment.
If you allow xpra to share the clipboard it will flash a notification whenever
the container accesses the host clipboard. I hacked together [a
script](/scripts/xcontainer) that will start a new xpra process running as a
nobody user, set a cookie in the Xauthority of the container allowing it to
connect to xpra, and start a shell in the container with DISPLAY set so GUI
programs just work. For example to get a shell as root in a kali container where
we can run GUI programs:
```sh
lxc launch images:kali kali
xcontainer kali root
```

## [Incus](https://github.com/lxc/incus)

Recently the main developers of LXD split off from canonical and started a fork
called Incus that is probably the future of LXD development. I'm still on LXD
just because my distro has LXD packaged and not Incus.

## Conclusion

While it's not as common as docker or VMware, I've found LXD quite nice for this
use case of dealing with a mix of disposable and persistent containers and
virtual machines. If you have any questions or advice, or are using some totally
different approach like Nix or Guix to create disposable containers and VMs, I'd
love to [hear from you](mailto:b0tato@proton.me).
