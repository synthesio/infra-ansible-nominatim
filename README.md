# Ansible Nominatim
Based on official install documentation: wiki.openstreetmap.org/wiki/Nominatim/Installation
This role installs a fully functionnal Nominatim instance on Debian OS

WARNING : this is configured to import planet-wide data, so it is recommended to use a tmux / screen session to launch this role.
          The longest task is the planet database import: about 4 days on a 4x SSD, 8 cores Xeon-D machine.

## Prerequisite
- Ansible 2.2
- Debian Jessie

