## Older instances (EXAMPLE)

For older instances under Micro, check the settings in `/opt/bitnami/php/etc/bitnami/common.conf`. These pretty much never crash:
```
pm.max_children=5
pm.start_servers=1
pm.min_spare_servers=1
pm.max_spare_servers=3
pm.max_requests=5000
pm=dynamic
```

Extra setting in `/opt/bitnami/php/etc/php.ini`. Can be reduced if emergency?
```
memory_limit = 128M
```

## Newer instances

Newer instances use much bigger settings `/opt/bitnami/php/etc/memory.conf` (IS SYMLINK).
To calculate max_children, use `top -o %MEM` (ie. "5%") and work out how many of these will fit (5% * 20 = 100%, so 20 children). Or just use the settings below:

### 1. Edit existing config file(s).

Run `ls -l` to see which file `/opt/bitnami/php/etc/memory.conf` points to:

```
ls -l

lrwxrwxrwx 1 root    root   24 Apr 20 14:55 memory.conf -> memory/memory-large.conf
```

Edit the corresponding file (In this case, `memory/memory-large.conf`) and change it to the corresponding config:

```
sudo nano /opt/bitnami/php/etc/memory/<NAME_OF_MEMORY_FILE>
```

#### Micro
```
pm.max_children=5
pm.start_servers=1
pm.min_spare_servers=1
pm.max_spare_servers=3
pm.max_requests=200
pm.process_idle_timeout=10s
```

#### Small
```
pm.max_children=10
pm.start_servers=2
pm.min_spare_servers=2
pm.max_spare_servers=5
pm.max_requests=200
pm.process_idle_timeout=10s
```

#### Medium
```
pm.max_children=25
pm.start_servers=4
pm.min_spare_servers=4
pm.max_spare_servers=10
pm.max_requests=200
pm.process_idle_timeout=10s
```

#### Large
```
pm.max_children=50
pm.start_servers=5
pm.min_spare_servers=5
pm.max_spare_servers=30
pm.max_requests=200
pm.process_idle_timeout=10s
```

### 2. Restart php-fpm after changing the settings:
```
sudo /opt/bitnami/ctlscript.sh restart php-fpm
```

This will hopefully stop the server from using up so much memory. If it continues to crash, reduce `max_children`, or reduce `max_requests` to around 100.

### Notes

Extra setting in `/opt/bitnami/php/etc/php.ini-production`. Can be reduced if emergency?
```
memory_limit = 128M
```

https://tideways.com/profiler/blog/an-introduction-to-php-fpm-tuning
