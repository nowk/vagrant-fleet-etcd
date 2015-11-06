# vagrant-fleet-etcd

Fleet cluster backed by etcd on Vagrant *(VirtualBox only)*

*All coreos, all the time.*


---

    user-data.etcd

This the `cloud-config` for the etcd boxes.

    user-data.fleet

The `cound-config` for the Fleet nodes.

    Vagrantfile

Used to spin up both the etcd cluster and the fleet cluster and their respective
provisions.

    ticker@.service

Sample unit file to play around with.

*(This does not get provisioned into any boxes, just for reference.)*


## Basic Usage

    vagrant up

To launch the vagrant boxes. By default you will get a 3 box etcd cluster and a
4 box fleet cluster.


    vagrant ssh fleet-00

and you will be on the first node box.

_(`$` is within a fleet-* box.)_


    $ fleetctl list-machines
    MACHINE         IP              METADATA
    4f5cc79f...     192.168.10.112  -
    a58776a4...     192.168.10.113  -
    ba55e211...     192.168.10.111  -
    f0bf93df...     192.168.10.110  -

If everything is configured properly you should see something similar to this. 
*pretty....*


    $ fleetctl submit ticker\@.service

Add your service.

*(Using the sample `ticker@.service` in this repo.)*

More info @ [Deploying a Service Using fleet][0]

  [0]: https://github.com/coreos/fleet/blob/master/Documentation/examples/example-deployment.md#service-files


    $ fleetctl start ticker@{1..4}.service
    Unit ticker@4.service launched
    Unit ticker@2.service launched on 4f5cc79f.../192.168.10.112
    Unit ticker@1.service launched on f0bf93df.../192.168.10.110
    Unit ticker@3.service launched on a58776a4.../192.168.10.113

Start your ticker units.


    $ fleetctl list-units
    UNIT                    MACHINE                         ACTIVE  SUB
    ticker@1.service        f0bf93df.../192.168.10.110      active  running
    ticker@2.service        4f5cc79f.../192.168.10.112      active  running
    ticker@3.service        a58776a4.../192.168.10.113      active  running
    ticker@4.service        ba55e211.../192.168.10.111      active  running

Each unit is spread across the cluster due to the `Conflicts` configuration 
defined in the unit file's `[X-Fleet]`.

---

Lets play...

    vagrant halt fleet-01

Drop a node, bye-bye.


    $ fleetctl list-machines
    MACHINE         IP              METADATA
    4f5cc79f...     192.168.10.112  -
    a58776a4...     192.168.10.113  -
    f0bf93df...     192.168.10.110  -

    $ fleetctl list-units
    UNIT                    MACHINE                         ACTIVE  SUB
    ticker@1.service        f0bf93df.../192.168.10.110      active  running
    ticker@2.service        4f5cc79f.../192.168.10.112      active  running
    ticker@3.service        a58776a4.../192.168.10.113      active  running

*(Note: the constraint will not bring the unit in the dropped back up in 
another node)*


    vagrant up fleet-01

Bring the node back online.


    $ fleetctl list-machines
    MACHINE         IP              METADATA
    4f5cc79f...     192.168.10.112  -
    a58776a4...     192.168.10.113  -
    ba55e211...     192.168.10.111  -
    f0bf93df...     192.168.10.110  -

    $ fleetctl list-units
    UNIT                    MACHINE                         ACTIVE  SUB
    ticker@1.service        f0bf93df.../192.168.10.110      active  running
    ticker@2.service        4f5cc79f.../192.168.10.112      active  running
    ticker@3.service        a58776a4.../192.168.10.113      active  running
    ticker@4.service        ba55e211.../192.168.10.111      active  running

And so is our service.

