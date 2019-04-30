# Ansible Nominatim
Based on official install documentation: wiki.openstreetmap.org/wiki/Nominatim/Installation
This role installs a fully functionnal Nominatim instance on Debian OS

WARNING : this is configured to import planet-wide data, so it is recommended to use a tmux / screen session to launch this role.
          The longest task is the planet database import: about 3 days on a 2x NVME SSD, 12 cores E5-1650 machine.

## Prerequisite
- Ansible 2.2
- Debian Stretch
- python2


## Playbook example
```
---

- hosts: "{{cluster_name}}"
  vars_files:
    - "vars/{{stage}}/osm/{{cluster_name}}.yml"
  roles:
    - nominatim
```

You could override the default values in the vars file.
Mainly for 
  - nominatim_version
  - web_hostname (vhost fqdn) for the service
  - apache2_trusted_proxies range


## Time for importing data
With these Server's specifications :
- 2 cores, 12 threads, Intel(R) Xeon(R) CPU E5-1650 v4 @ 3.60GHz
- 64GB of memory
- 2 nvme 2TB ; software raid1

planet import : approx 2.5 days
tiger2018 import : 2h

### Import processing

For our setup, tuning PostgreSQL as recommended in the wiki doesn't change import speed so much
```
fsync = off
full_page_writes = off
synchronous_commit = off

```
So we choose to NOT change theses values for planet import and let the default PostgreSQL configuration.

During imports, you can follow the output in the log files, for examples
```
/var/log/nominatim/planet_import.log
/var/log/nominatim/tiger_import.log
```

### Troubleshooting
http://nominatim.org/release-docs/latest/admin/Faq/#can-a-stoppedkilled-import-process-be-resumed

To see if there is a blocked SQL operation in PG
```
psql -d nominatim
nominatim=# select pid, usename, pg_blocking_pids(pid) as blocked_by, query as blocked_query from pg_stat_activity where cardinality(pg_blocking_pids(pid)) > 0;?
```

Check the import log
Check the PostgreSQL logs

The index creation could be restarted manually
```
./utils/setup.php --index --create-search-indices --create-country-names
```
(see `CONST_BasePath.'/utils/setup.php` for the full import process)

### Benchmarks
see official wiki page https://wiki.openstreetmap.org/wiki/Osm2pgsql/benchmarks

https://wiki.openstreetmap.org/wiki/Osm2pgsql
http://www.geofabrik.de/media/2012-09-08-osm2pgsql-performance.pdf

