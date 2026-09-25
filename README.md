# Ansible Role: Redis

Installs [Redis](http://redis.io/) on Linux.

## Role Variables

### Role vars (global)

| Variable | Default | Description |
| --- | --- | --- |
| `redis_enabled` | `true` | Whether instance services are started/enabled. |
| `redis_port` | `6379` | Default port used when `redis_instances` is empty. |
| `redis_bind_interface` | `127.0.0.1` | Bind address (set `0.0.0.0` to listen on all interfaces). |
| `redis_unixsocket` | `''` | Optional Unix socket path. |
| `redis_timeout` | `300` | Client idle timeout (`0` disables). |
| `redis_loglevel` | `notice` | Log level. |
| `redis_logfile` | `/var/log/redis/redis-server.log` | Base logfile path (per-instance logfile is generated). |
| `redis_databases` | `16` | Number of Redis DBs. |
| `redis_save` | `[900 1, 300 10, 60 10000]` | RDB snapshot rules. |
| `redis_rdbcompression` | `yes` | RDB compression toggle. |
| `redis_dbfilename` | `dump.rdb` | RDB filename. |
| `redis_dbdir` | `/var/lib/redis` | Base data dir (per-instance data dir is generated). |
| `redis_maxmemory` | `0` | Default max memory (`0` = unlimited). |
| `redis_maxmemory_policy` | `noeviction` | Default eviction policy. |
| `redis_maxmemory_samples` | `5` | Samples for LRU/LFU approximation. |
| `redis_appendonly` | `no` | AOF persistence toggle. |
| `redis_appendfsync` | `everysec` | AOF fsync mode (`always`, `everysec`, `no`). |
| `redis_includes` | `[]` | Extra include files in `redis.conf`. |
| `redis_requirepass` | `""` | Default password when not set per instance. |
| `redis_replicaof_host` | `""` | Default primary host; empty keeps instances writable primaries. |
| `redis_masterauth` | `""` | One credential every replica authenticates to its primary with. Overrides the per-instance derivation below. |
| `redis_replica_read_only` | `yes` | Whether replicas reject writes. |
| `redis_disabled_commands` | `[]` | Commands to disable via `rename-command ... ""`. |
| `redis_extra_config` | `""` | Extra raw Redis config appended at end of file. |
| `redis_instances` | `[]` | List of instances (see table below). |
| `redis_conf_dir` | `/etc/redis` | Directory for generated instance configs. |
| `redis_runtime_dir` | `/run/redis` | PID/runtime directory. |
| `redis_log_dir` | `/var/log/redis` | Directory for per-instance logs. |
| `redis_data_dir` | `/var/lib/redis` | Directory for per-instance data folders. |
| `redis_supervised` | `systemd` | Value for `supervised` in config. |
| `redis_daemonize` | `no` | Value for `daemonize` in config. |
| `redis_disable_default_service` | `true` | Disable packaged default `redis-server` service. |
| `redis_healthcheck_retries` | `12` | Number of authenticated readiness/replication checks. |
| `redis_healthcheck_delay` | `5` | Seconds between readiness/replication checks. |
| `redis_healthcheck_host` | Derived from `redis_bind_interface` | Address the readiness/replication checks connect to. Loopback for a wildcard bind, the bound address otherwise. |
| `redis_enablerepo` | `epel` | RHEL-family repo override (not used on Debian). |
| `redis_package` | OS-specific | Package name (`redis-server` on Debian). |

### Instance vars (`redis_instances[]`)

| Field | Required | Default / Fallback | Description |
| --- | --- | --- | --- |
| `name` | Yes | - | Instance name. Used for config/service/data naming. |
| `port` | Yes | - | Redis port for this instance. |
| `password` | No | `redis_requirepass` | Password for this instance. |
| `maxmemory` | No | `redis_maxmemory` | Per-instance max memory override. |
| `maxmemory_policy` | No | `redis_maxmemory_policy` | Per-instance eviction policy override. |
| `replicaof_host` | No | `redis_replicaof_host` | Primary host for this instance; empty makes it a primary. |
| `replicaof_port` | No | Instance `port` | Port of the corresponding primary instance. |
| `masterauth` | No | `redis_masterauth`, then this instance's effective password | Credential used for primary-to-replica synchronization. |

## Dependencies

None.

Instance units stop Redis with `SIGTERM`, which follows Redis's graceful
shutdown path without embedding an authenticated `redis-cli` invocation in the
unit or exposing a password in the process list.

The role restarts the instances whose configuration or unit file changed
itself, just before the readiness checks, rather than notifying a handler:
handlers only run at the end of the play, and `meta: flush_handlers` is
play-scoped — it would also fire whatever the roles before this one had
queued. The readiness and replication checks are skipped under `--check`,
where there is nothing listening to check yet.

An instance without a password is only reachable over loopback, whatever
`redis_bind_interface` says: Redis protected mode refuses every other
connection to an instance whose default user has no password. Give any
instance that has to be reached from another host — a replica synchronizing
from it included — a `password`.

## Example Playbook

```yaml
- hosts: all
  become: true
  roles:
    - role: ansible-role-redis
      vars:
        redis_bind_interface: "0.0.0.0"
        redis_instances:
          - name: "{{ immich_redis_db }}"
            port: "{{ immich_redis_port }}"
            password: "{{ immich_redis_password }}"
          - name: "{{ nextcloud_redis_db }}"
            port: "{{ nextcloud_redis_port }}"
            password: "{{ nextcloud_redis_password }}"
          - name: "{{ paperless_redis_db }}"
            port: "{{ paperless_redis_port }}"
            password: "{{ paperless_redis_password }}"
```

## License

MIT / BSD

## Author Information

This role was created in 2014 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).

It has since been modified and is maintained by Jari.
