**************
VM Autohealing
**************

Besides our own implementation of autohealing there is also idea to use existing
software and teach openstack to use it.

Idea is to modify nova plugin to use cluster management software, which itself
will allow us to add HA mechanisms.
Basic usage case is to simply add/edit nova backend plugin for given hipervisor
and apply its HA mechanisms.

Currently we have this cluster management software to work with:

    * VMWare VCenter

        VMWare has its own cluster management software, namely VCenter, and
        if we would like to add HA implementation it would only mean that we
        have to boot VMS with HA flag on. Least work on our side.

    * Xen Server

        Xen server seems to have some kind of HA mechanism, we need to take a
        closer look at what exactly does that mean

    * KVM

        KVM seems to be biggest problem in our case, as its one of most widely
        used hypervisor, and raw libvirt+kvm/qemu is oblivious to cluster and
        works only in scope of compute node. We would have to add another
        software which works on cluster level.
        We might to write nova plugin from scratch for it. Also, depending how
        this piece of software will implement networking, we might also have to
        write neutron plugin.
        We might also have to write some importer, which will allow to migrate
        both existing cluster to openstack and pure openstack to cluster manged
        by this piece of software.
        So far we have found these options:

            * Proxmox

                * AGPL licence.
                * HA based on DRBD
                * Fencing
