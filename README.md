# debian-workstation

## TODO

* Debian
  - Get [nfs mount](https://developer.hashicorp.com/vagrant/docs/synced-folders/nfs) working 
  - libvirt uses "pause" state on `vagrant suspend` rather than a "saved" state.
    * VM seems to corrupt over night.  CPU is stopped, but memory and other resources are still allocated.

## Debian

```bash
cp debian/Vagrantfile ./
cp debian/workstation ~/bin

# copy/paste line from debian/crontab.halt
crontab -e
```
