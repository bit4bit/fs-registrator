# fs-registrator

*FreeSWITCH Sofia-SIP Registry Bridge (Sync to Key/Value Store)*

[![Build Status](https://travis-ci.org/CpuID/fs-registrator.svg?branch=master)](https://travis-ci.org/CpuID/fs-registrator) [![Coverage Status](https://coveralls.io/repos/github/CpuID/fs-registrator/badge.svg?branch=master)](https://coveralls.io/github/CpuID/fs-registrator?branch=master)

# Summary

When run and pointed at a FreeSWITCH instance, will take note of all Sofia-SIP registrations, and propagate them to a (replicated) K/V store for lookups.

Useful for discovering which SIP registrations reside on which server/s.

We use ESL events + a semi-regular sync for reconciliation (to gracefully handle restarts and/or missed events).

# Supported K/V Stores

Currently the focus is on [etcd](https://github.com/coreos/etcd), with the intention to support others in future. [Consul](https://github.com/hashicorp/consul) and [redis](https://github.com/antirez/redis) would be the most likely next targets (both support prefix-based wildcard lookups and TTLs for the most part).

New K/V store backends can be added, see [kv_etcd.go](https://github.com/CpuID/fs-registrator/blob/master/kv_etcd.go) for an example implementation. As long as you satisfy the [KvBackend](https://github.com/CpuID/fs-registrator/blob/master/kv.go#L10-L13) interface and [register the backend](https://github.com/CpuID/fs-registrator/blob/master/kv.go#L18), it will be available.

# Configuration

Configuration is performed via CLI arguments, and self documenting using `--help`:

```
NAME:
   fs-registrator - FreeSWITCH Sofia-SIP Registry Bridge (Sync to Key/Value Store)

USAGE:
   fs-registrator [global options] command [command options] [arguments...]

VERSION:
   0.1.0

COMMANDS:
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --fshost value           FreeSWITCH ESL Hostname/IP (default: "localhost")
   --fsport value           FreeSWITCH ESL Port (default: 8021)
   --fspassword value       FreeSWITCH ESL Password (default: "ClueCon")
   --fsprofiles value       List of Sofia Profiles to watch (comma separated list) (default: "internal")
   --fsadvertiseip value    SIP Destination IP to store in K/V Store for FreeSWITCH
   --fsadvertiseport value  SIP Destination Port to store in K/V Store for FreeSWITCH
   --kvbackend value        Key/Value Backend (one of: etcd) (default: "etcd")
   --kvhost value           Key/Value Store Hostname/IP (default: "etcd")
   --kvport value           Key/Value Store Port (default: 2379)
   --kvprefix value         Key Space Prefix in K/V Store to store Registrations (default: "fs_registrations")
   --syncinterval value     Interval (in seconds) between full sync. A full sync is performed on initial startup also. (default: 3600)
   --help, -h               show help
   --version, -v            print the version
```

# Building

`go get -d && go build` should produce a single executable. Binary releases are also available [here](https://github.com/CpuID/ec2-sg-mangler/releases)

# Running Tests

You'll need to add `sip.testserver.tld` to your `/etc/hosts` file, to workaround sipsak wanting a resolveable hostname for some tests. For background, see this [mailing list post](http://lists.sip-router.org/pipermail/sr-users/2005-September/051903.html). Something like the below is fine in `/etc/hosts`:

```
127.0.0.1   sip.testserver.tld
```

Also, some of the tests rely on [Docker](https://github.com/docker/docker) containers to provide a real FreeSWITCH/etcd instances. Ensure you have Docker + Docker Compose installed, then run:

```
docker-compose pull
go test
```

Tests requiring Docker use [libcompose](https://github.com/docker/libcompose) in [main_test.go](https://github.com/CpuID/fs-registrator/blob/master/main_test.go)
