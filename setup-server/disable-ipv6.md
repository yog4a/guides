# Disabling IPv6 on Debian/Ubuntu

This guide provides step-by-step instructions for disabling IPv6 on Debian/Ubuntu systems.

## Create or edit the IPv6 configuration file

```bash
sudo chmod 666 "/etc/sysctl.d/99-disable-ipv6.conf"
sudo nano "/etc/sysctl.d/99-disable-ipv6.conf"
```

## Add the following lines to the configuration file

```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

## Restore original file permissions

```bash
sudo chmod 644 /etc/sysctl.d/99-disable-ipv6.conf
```

## Apply the changes

```bash
sudo sysctl --system
```

## Verify IPv6 is disabled

```bash
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
```

If it returns `1`, IPv6 is successfully disabled. A value of `0` means IPv6 is still enabled.