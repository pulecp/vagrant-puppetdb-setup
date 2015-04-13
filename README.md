**SOURCE:** [git.vstone.org](http://git.vstone.org/?p=puppet/vagrant/shelled.git)

# Quick Start

## Files

* `puppet/manifests/`:        Default manifests location.
* `puppet/modules/`           Default modules location.
* `puppet/puppet.conf-base`:  This file is copied over as the puppet.conf file
* `puppet/hiera.yaml`:        If present, this file is used as hiera configuration file.
* `hieradata/`:               Default hieradata directory.
* `graphs/`:                  Contains the generated graphs. Useful for debugging dependency issues.

## Puppet Tree

Add your puppet tree in the puppet folder.

## Puppet Master

Adjust the module path for the puppet master configuration:

    vi augeas/puppetmaster.aug


