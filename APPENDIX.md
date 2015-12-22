# Appendix: OpenStack Swift でS3互換APIを使う

---
## ■はじめに
### ■■このドキュメントの目的
* vagrantで構築したOpenStack SwiftでS3互換APIを利用可能にする

### ■■前提条件
* vagrant upしてsiwft起動できること

### ■■想定読者
* OpenStack SwiftでS3互換APIを使いたい人

### ■■想定知識
* OpenStack Swiftの概要が分かること

---
## ■手順
### ■■まずはswift APIの動作確認
* クライアントから。。

``` bash
curl -v -H 'X-Storage-User: test:tester' -H 'X-Storage-Pass: testing'  http://192.168.8.80:8080/auth/v1.0

curl -v -H 'X-Auth-Token: AUTH_tkc6669fb15e80470483a709fca55f6d1b' http://192.168.8.80:8080/v1/AUTH_test
curl -v -H 'X-Auth-Token: AUTH_tkc6669fb15e80470483a709fca55f6d1b' http://192.168.8.80:8080/v1/AUTH_test/test1 -X PUT
echo ABC > test1.txt
curl -v -H 'X-Auth-Token: AUTH_tkc6669fb15e80470483a709fca55f6d1b' http://192.168.8.80:8080/v1/AUTH_test/test1/test.txt -T test1.txt

curl -v -H 'X-Auth-Token: AUTH_tkc6669fb15e80470483a709fca55f6d1b' http://192.168.8.80:8080/v1/AUTH_test/test1/
```

### ■■swift3インストール
* インストール

```bash
git clone https://github.com/stackforge/swift3.git
cd swift3
sudo python setup.py install
```

* 20_settings.conf修正

```bash
vi 20_settings.conf

vagrant@saio:/etc/swift/proxy-server/proxy-server.conf.d$ diff 20_settings.conf 20_settings.conf.ORG
5c5
< pipeline = catch_errors healthcheck proxy-logging cache swift3 container_sync bulk tempurl tempauth slo dlo staticweb proxy-logging proxy-server
---
> pipeline = catch_errors healthcheck proxy-logging cache container_sync bulk tempurl tempauth slo dlo staticweb proxy-logging proxy-server
13,15d12
<
< [filter:swift3]
< use = egg:swift3#swift3
```

* swift proxyの再起動

```bash
sudo swift-init proxy restart
```

### ■■便利グッズインストール
* xmlindent

```bash
sudo apt-get xmlindent
```

* unzip

```bash
sudo apt-get install unzip
```

* s3-curl

```bash
wget http://s3.amazonaws.com/doc/s3-example-code/s3-curl.zip
unzip s3-curl.zip
cd s3-curl/
chmod +x s3curl.pl

vi s3curl.pl

vagrant@saio:~/s3-curl$ diff s3curl.pl s3curl.pl.ORG
30,39c30,37
< #my @endpoints = ( 's3.amazonaws.com',
< #                  's3-us-west-1.amazonaws.com',
< #                  's3-us-west-2.amazonaws.com',
< #                  's3-us-gov-west-1.amazonaws.com',
< #                  's3-eu-west-1.amazonaws.com',
< #                  's3-ap-southeast-1.amazonaws.com',
< #                  's3-ap-northeast-1.amazonaws.com',
< #                  's3-sa-east-1.amazonaws.com', );
<
< my @endpoints = ( 'localhost' );
---
> my @endpoints = ( 's3.amazonaws.com',
>                   's3-us-west-1.amazonaws.com',
>                   's3-us-west-2.amazonaws.com',
>                   's3-us-gov-west-1.amazonaws.com',
>                   's3-eu-west-1.amazonaws.com',
>                   's3-ap-southeast-1.amazonaws.com',
>                   's3-ap-northeast-1.amazonaws.com',
>                   's3-sa-east-1.amazonaws.com', );
127,128c125,126
< #    printCmdlineSecretWarning();
< #    sleep 5;
---
>     printCmdlineSecretWarning();
>     sleep 5;

```

### ■■s3-curlで動確

```bash
./s3curl.pl --id test:tester --key testing -- -s -v http://localhost:8080/ -k | xmlindent

./s3curl.pl --id test:tester --key testing --put /dev/null -- -s -v http://localhost:8080/test2 -k | xmlindent
./s3curl.pl --id test:tester --key testing -- -s -v http://localhost:8080/ -k | xmlindent

./s3curl.pl --id test:tester --key testing -- -s -v http://localhost:8080/test1 -k | xmlindent



./s3curl.pl --id test:tester --key testing --put ./README -- -s -v http://localhost:8080/test2/AAAA -k | xmlindent
./s3curl.pl --id test:tester --key testing -- -s -v http://localhost:8080/test2/AAAA -k | xmlindent
```

---
## ■おまけ: vagrant upからサービス起動までのコンソールログ

``` bash
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Setting the name of the VM: vagrant-saio-20151222-164110
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
    default: Adapter 2: hostonly
==> default: Forwarding ports...
    default: 22 => 2222 (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Connection timeout. Retrying...
==> default: Machine booted and ready!
==> default: Configuring proxy for Apt...
==> default: Configuring proxy for Chef provisioners...
==> default: Configuring proxy environment variables...
GuestAdditions 4.3.28 running --- OK.
==> default: Checking for guest additions in VM...
==> default: Setting hostname...
==> default: Configuring and enabling network interfaces...
==> default: Mounting shared folders...
    default: /vagrant => D:/vagrant/vagrant-swift-all-in-one
==> default: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> default: to force provisioning. Provisioners marked to run always will still run.

hoge@poyo /d/vagrant/vagrant-swift-all-in-one (feature_proxy)
$ vagrant ssh
Welcome to Ubuntu 14.04.3 LTS (GNU/Linux 3.13.0-71-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Tue Dec 22 07:41:34 UTC 2015

  System load:  0.07              Processes:           93
  Usage of /:   5.4% of 39.34GB   Users logged in:     0
  Memory usage: 16%               IP address for eth0: 10.0.2.15
  Swap usage:   0%                IP address for eth1: 192.168.8.80

  Graph this data and manage this system at:
    https://landscape.canonical.com/

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud


Last login: Tue Dec 22 07:33:48 2015 from 10.0.2.2
vagrant@saio:~$
vagrant@saio:~$
vagrant@saio:~$ sudo swift-init status
Usage: swift-init <server>[.<config>] [<server>[.<config>] ...] <command> [options]

where:
    <server>  is the name of a swift service e.g. proxy-server.
              The '-server' part of the name may be omitted.
    <config>  is an explicit configuration filename without the
              .conf extension. If <config> is specified then <server> should
              refer to a directory containing the configuration file, e.g.:

                  swift-init object.1 start

              will start an object-server using the configuration file
              /etc/swift/object-server/1.conf
    <command> is a command from the list below.

Commands:
    force-reload: alias for reload
            kill: stop a server (no error if not running)
       no-daemon: start a server interactively
         no-wait: spawn server and return immediately
            once: start server and run one pass on supporting daemons
          reload: graceful shutdown then restart on supporting servers
         restart: stops then restarts server
        shutdown: allow current requests to finish on supporting servers
           start: starts a server
          status: display status of tracked pids for server
            stop: stops a server

Options:
  -h, --help            show this help message and exit
  -v, --verbose         display verbose output
  -w, --no-wait         won't wait for server to start before returning
  -o, --once            only run one pass of daemon
  -n, --no-daemon       start server interactively
  -g, --graceful        send SIGHUP to supporting servers
  -c N, --config-num=N  send command to the Nth server only
  -k N, --kill-wait=N   wait N seconds for processes to die (default 15)
  -r RUN_DIR, --run-dir=RUN_DIR
                        alternative directory to store running pid files
                        default: /var/run/swift
  --strict              Return non-zero status code if some config is missing.
                        Default mode if all servers are explicitly named.
  --non-strict          Return zero status code even if some config is
                        missing. Default mode if any server is a glob or one
                        of aliases `all`, `main` or `rest`.
ERROR: specify server(s) and command
vagrant@saio:~$ sudo swift-init all status
No container-updater running
No account-auditor running
No object-replicator running
No container-sync running
No container-replicator running
No object-auditor running
No object-expirer running
No container-auditor running
No container-server running
No object-reconstructor running
No object-server running
No account-reaper running
No proxy-server running
No account-replicator running
No object-updater running
No container-reconciler running
No account-server running
vagrant@saio:~$ sudo swift-init all start
Starting container-updater...(/etc/swift/container-server/1.conf.d)
Starting container-updater...(/etc/swift/container-server/2.conf.d)
Starting container-updater...(/etc/swift/container-server/3.conf.d)
Starting container-updater...(/etc/swift/container-server/4.conf.d)
Starting account-auditor...(/etc/swift/account-server/1.conf.d)
Starting account-auditor...(/etc/swift/account-server/2.conf.d)
Starting account-auditor...(/etc/swift/account-server/3.conf.d)
Starting account-auditor...(/etc/swift/account-server/4.conf.d)
Starting object-replicator...(/etc/swift/object-server/1.conf.d)
Starting object-replicator...(/etc/swift/object-server/2.conf.d)
Starting object-replicator...(/etc/swift/object-server/3.conf.d)
Starting object-replicator...(/etc/swift/object-server/4.conf.d)
Starting container-sync...(/etc/swift/container-server/1.conf.d)
Starting container-sync...(/etc/swift/container-server/2.conf.d)
Starting container-sync...(/etc/swift/container-server/3.conf.d)
Starting container-sync...(/etc/swift/container-server/4.conf.d)
Starting container-replicator...(/etc/swift/container-server/1.conf.d)
Starting container-replicator...(/etc/swift/container-server/2.conf.d)
Starting container-replicator...(/etc/swift/container-server/3.conf.d)
Starting container-replicator...(/etc/swift/container-server/4.conf.d)
Starting object-auditor...(/etc/swift/object-server/1.conf.d)
Starting object-auditor...(/etc/swift/object-server/2.conf.d)
Starting object-auditor...(/etc/swift/object-server/3.conf.d)
Starting object-auditor...(/etc/swift/object-server/4.conf.d)
Starting object-expirer...(/etc/swift/object-expirer.conf.d)
Starting container-auditor...(/etc/swift/container-server/1.conf.d)
Starting container-auditor...(/etc/swift/container-server/2.conf.d)
Starting container-auditor...(/etc/swift/container-server/3.conf.d)
Starting container-auditor...(/etc/swift/container-server/4.conf.d)
Starting container-server...(/etc/swift/container-server/1.conf.d)
Starting container-server...(/etc/swift/container-server/2.conf.d)
Starting container-server...(/etc/swift/container-server/3.conf.d)
Starting container-server...(/etc/swift/container-server/4.conf.d)
Starting object-reconstructor...(/etc/swift/object-server/1.conf.d)
Starting object-reconstructor...(/etc/swift/object-server/2.conf.d)
Starting object-reconstructor...(/etc/swift/object-server/3.conf.d)
Starting object-reconstructor...(/etc/swift/object-server/4.conf.d)
Starting object-server...(/etc/swift/object-server/1.conf.d)
Starting object-server...(/etc/swift/object-server/2.conf.d)
Starting object-server...(/etc/swift/object-server/3.conf.d)
Starting object-server...(/etc/swift/object-server/4.conf.d)
Starting account-reaper...(/etc/swift/account-server/1.conf.d)
Starting account-reaper...(/etc/swift/account-server/2.conf.d)
Starting account-reaper...(/etc/swift/account-server/3.conf.d)
Starting account-reaper...(/etc/swift/account-server/4.conf.d)
Starting proxy-server...(/etc/swift/proxy-server/proxy-noauth.conf.d)
Starting proxy-server...(/etc/swift/proxy-server/proxy-server.conf.d)
Starting account-replicator...(/etc/swift/account-server/1.conf.d)
Starting account-replicator...(/etc/swift/account-server/2.conf.d)
Starting account-replicator...(/etc/swift/account-server/3.conf.d)
Starting account-replicator...(/etc/swift/account-server/4.conf.d)
Starting object-updater...(/etc/swift/object-server/1.conf.d)
Starting object-updater...(/etc/swift/object-server/2.conf.d)
Starting object-updater...(/etc/swift/object-server/3.conf.d)
Starting object-updater...(/etc/swift/object-server/4.conf.d)
Starting container-reconciler...(/etc/swift/container-reconciler.conf.d)
Starting account-server...(/etc/swift/account-server/1.conf.d)
Starting account-server...(/etc/swift/account-server/2.conf.d)
Starting account-server...(/etc/swift/account-server/3.conf.d)
Starting account-server...(/etc/swift/account-server/4.conf.d)
Traceback (most recent call last):
  File "/usr/local/bin/swift-object-replicator", line 6, in <module>
    exec(compile(open(__file__).read(), __file__, 'exec'))
  File "/vagrant/swift/bin/swift-object-replicator", line 17, in <module>
    from swift.obj.replicator import ObjectReplicator
  File "/vagrant/swift/swift/obj/replicator.py", line 41, in <module>
    from swift.obj.diskfile import DiskFileManager, get_data_dir, get_tmp_dir
  File "/vagrant/swift/swift/obj/diskfile.py", line 60, in <module>
    from swift.common.splice import splice, tee
  File "/vagrant/swift/swift/common/splice.py", line 104, in <module>
    tee = Tee()
  File "/vagrant/swift/swift/common/splice.py", line 41, in __init__
    libc = ctypes.CDLL(ctypes.util.find_library('c'), use_errno=True)
  File "/usr/lib/python2.7/ctypes/util.py", line 253, in find_library
    return _findSoname_ldconfig(name) or _get_soname(_findLib_gcc(name))
  File "/usr/lib/python2.7/ctypes/util.py", line 242, in _findSoname_ldconfig
    f = os.popen('/sbin/ldconfig -p 2>/dev/null')
OSError: [Errno 12] Cannot allocate memory

Traceback (most recent call last):
  File "/usr/local/bin/swift-object-replicator", line 6, in <module>
    exec(compile(open(__file__).read(), __file__, 'exec'))
  File "/vagrant/swift/bin/swift-object-replicator", line 17, in <module>
    from swift.obj.replicator import ObjectReplicator
  File "/vagrant/swift/swift/obj/replicator.py", line 41, in <module>
    from swift.obj.diskfile import DiskFileManager, get_data_dir, get_tmp_dir
  File "/vagrant/swift/swift/obj/diskfile.py", line 60, in <module>
    from swift.common.splice import splice, tee
  File "/vagrant/swift/swift/common/splice.py", line 104, in <module>
    tee = Tee()
  File "/vagrant/swift/swift/common/splice.py", line 41, in __init__
    libc = ctypes.CDLL(ctypes.util.find_library('c'), use_errno=True)
  File "/usr/lib/python2.7/ctypes/util.py", line 253, in find_library
    return _findSoname_ldconfig(name) or _get_soname(_findLib_gcc(name))
  File "/usr/lib/python2.7/ctypes/util.py", line 242, in _findSoname_ldconfig
    f = os.popen('/sbin/ldconfig -p 2>/dev/null')
OSError: [Errno 12] Cannot allocate memory

Traceback (most recent call last):
  File "/usr/local/bin/swift-object-replicator", line 6, in <module>
    exec(compile(open(__file__).read(), __file__, 'exec'))
  File "/vagrant/swift/bin/swift-object-replicator", line 17, in <module>
    from swift.obj.replicator import ObjectReplicator
  File "/vagrant/swift/swift/obj/replicator.py", line 41, in <module>
    from swift.obj.diskfile import DiskFileManager, get_data_dir, get_tmp_dir
  File "/vagrant/swift/swift/obj/diskfile.py", line 60, in <module>
    from swift.common.splice import splice, tee
  File "/vagrant/swift/swift/common/splice.py", line 104, in <module>
    tee = Tee()
  File "/vagrant/swift/swift/common/splice.py", line 41, in __init__
    libc = ctypes.CDLL(ctypes.util.find_library('c'), use_errno=True)
  File "/usr/lib/python2.7/ctypes/util.py", line 253, in find_library
    return _findSoname_ldconfig(name) or _get_soname(_findLib_gcc(name))
  File "/usr/lib/python2.7/ctypes/util.py", line 242, in _findSoname_ldconfig
    f = os.popen('/sbin/ldconfig -p 2>/dev/null')
OSError: [Errno 12] Cannot allocate memory

vagrant@saio:~$
vagrant@saio:~$ sudo swift-init all status
container-updater running (2170 - /etc/swift/container-server/1.conf.d)
container-updater running (2171 - /etc/swift/container-server/2.conf.d)
container-updater running (2172 - /etc/swift/container-server/3.conf.d)
container-updater running (2173 - /etc/swift/container-server/4.conf.d)
account-auditor running (2176 - /etc/swift/account-server/3.conf.d)
account-auditor running (2177 - /etc/swift/account-server/4.conf.d)
account-auditor running (2174 - /etc/swift/account-server/1.conf.d)
account-auditor running (2175 - /etc/swift/account-server/2.conf.d)
No object-replicator running
container-sync running (2184 - /etc/swift/container-server/3.conf.d)
container-sync running (2182 - /etc/swift/container-server/1.conf.d)
container-sync running (2183 - /etc/swift/container-server/2.conf.d)
container-replicator running (2187 - /etc/swift/container-server/2.conf.d)
container-replicator running (2189 - /etc/swift/container-server/4.conf.d)
object-auditor running (2192 - /etc/swift/object-server/3.conf.d)
object-auditor running (2193 - /etc/swift/object-server/4.conf.d)
object-auditor running (2190 - /etc/swift/object-server/1.conf.d)
object-auditor running (2191 - /etc/swift/object-server/2.conf.d)
object-expirer running (2194 - /etc/swift/object-expirer.conf.d)
container-auditor running (2195 - /etc/swift/container-server/1.conf.d)
container-auditor running (2196 - /etc/swift/container-server/2.conf.d)
container-auditor running (2197 - /etc/swift/container-server/3.conf.d)
container-auditor running (2198 - /etc/swift/container-server/4.conf.d)
container-server running (2200 - /etc/swift/container-server/2.conf.d)
container-server running (2201 - /etc/swift/container-server/3.conf.d)
container-server running (2202 - /etc/swift/container-server/4.conf.d)
container-server running (2199 - /etc/swift/container-server/1.conf.d)
object-reconstructor running (2203 - /etc/swift/object-server/1.conf.d)
object-reconstructor running (2204 - /etc/swift/object-server/2.conf.d)
object-reconstructor running (2205 - /etc/swift/object-server/3.conf.d)
object-reconstructor running (2206 - /etc/swift/object-server/4.conf.d)
object-server running (2209 - /etc/swift/object-server/3.conf.d)
object-server running (2210 - /etc/swift/object-server/4.conf.d)
account-reaper running (2211 - /etc/swift/account-server/1.conf.d)
account-reaper running (2212 - /etc/swift/account-server/2.conf.d)
account-reaper running (2213 - /etc/swift/account-server/3.conf.d)
account-reaper running (2214 - /etc/swift/account-server/4.conf.d)
proxy-server running (2216 - /etc/swift/proxy-server/proxy-server.conf.d)
proxy-server running (2215 - /etc/swift/proxy-server/proxy-noauth.conf.d)
account-replicator running (2217 - /etc/swift/account-server/1.conf.d)
account-replicator running (2219 - /etc/swift/account-server/3.conf.d)
account-replicator running (2220 - /etc/swift/account-server/4.conf.d)
object-updater running (2224 - /etc/swift/object-server/4.conf.d)
object-updater running (2221 - /etc/swift/object-server/1.conf.d)
object-updater running (2223 - /etc/swift/object-server/3.conf.d)
container-reconciler running (2225 - /etc/swift/container-reconciler.conf.d)
account-server running (2226 - /etc/swift/account-server/1.conf.d)
account-server running (2227 - /etc/swift/account-server/2.conf.d)
account-server running (2228 - /etc/swift/account-server/3.conf.d)
account-server running (2229 - /etc/swift/account-server/4.conf.d)
```
