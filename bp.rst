**************
VM Autohealing
**************


Problem
-------

Openstack in its current release doesn't provide any mechanism for autohealing.
There are 3rd party software which supplies such usability, like Cloudify, but
we would like to see something like that in openstack core.

Autohealing ability is one of key features of enterprise-ready cloud.


VMR
---

VMR is an idea created in Intel and Red Hat. It would work along with ceilometer
and nova (? nova also gives us host-down info ?). We would create entity called
contract which will essentialy be <trigger> -> <action> mechanism. For example
Ceilometer alarm will trigger action defined in contract, like vm evacuate.


Possible actions
----------------

    * Live migrate VM
        Mve out vms from a host. Usable for scheduled maintanance. Not
        usable for random host failure or network down. Also shared storage is
        required between hosts.

    * Boot from snapshot
        If we lose contact to master node we may boot up slave node from
        snapshot. It may be useful for servers without important stateful
        storage, like edge servers or http servers with external DB.
        Also it may be useful for stateful vms if storage is placed on volume,
        then we can attach volume after boot and therefore get storage back.

    * Evacuate
        Much like above, only vm is booted from existing vm file on shared
        storage. New password for vm is generated.

    * Active-standby + floating IP
        We could also create keepalived-like functionality. We may boot up 2
        instances of given type and in case of host-down we may migrate floating
        ip from one to another. Will require neutron setup.

    * Active-standby + LBaaS
        Much like example above only we perform migration on LB rather than
        floatings.

    * Suspend/Resume
        Its not autohealing per se, we could free up some resources for other
        instances this way.

    * Rebuild
        Not much use for host failure, but may be useful for VM OS healing.

    * Custom action
        Mistral?

    Note: We might also have to apply STONITH or other fencing technique
    on these scenerios (only LM is safe in that matter) to ensure that for
    example split-brain will not cause cronjobs on former master nodes to mess
    up data.


Our place in OS
---------------

Problem we are facing is where exactly we find ourselves in openstack environ.
There are few options:

    * Nova
        Obvious first choice, since we will be working closely with hipervisor.
        
        Pros:
            * Core openstack project
            * Strong hipervisor integration
            * Every openstack installation has nova enabled
        Cons:
            * Nova guys don't want us in
            * We are dependant on non-essential projects like ceilometer
            * Not really nova usage case (although imho vnc isn't one as well)

    * Glance
        A bit far fetched idea, but idea nonetheless
        Pros:
            * Core openstack project
            * We will be woring with images, snapshots etc
            * We could use graffiti for it (for example HA flavor)
            * We have strong connectons with Glance guys
        Cons:
            * This is really far fetched idea
            * Not a glance use case

    * Ceilometer
        Since we will be dependant on it, we might as well be part of it
        Pros:
            * We could connect to alarm api istelf and optimize performance
            * We have access to all the raw data we need
            * Core Openstack Project
            * PTL is from Red Hat
        Cons:
            * Ceilometer is only for metering, so its not its use case
            * Ceilometer guys don't want us in

    * Heat
        Orchiestration is related to autohealing
        Pros:
            * Core Openstack project
            * PTL is from Red Hat
            * Dependant on ceilometer (as are we)
            * Heat use case
        Cons:
            * Legacy code, deployments without heat will not have our tools

        Note: We still have to talk with Heat guys. Also legacy problem might
        be avoidable if we would use some sort of heat impoter. I've found this:
        http://dev.cloudwatt.com/en/blog/introducing-flame-automatic-heat-template-generation.html
        might be worth looking at.

    * Mistral
        We'll use mistral and it might be good place as well
        Pros:
            * Young project, we might have strong impact on it
            * Easily appendable to existing deployments
        Cons:
            * Not a openstack core project
            * Not completely our use case

    * Rally
        It already has scenerios we could use and vm setup mechanisms
        Pros:
            * It has some abilities we could use
        Cons:
            * Not an openstack core project
            * Not really our use case

    * Our own project
        We have always option to start from scratch
        Pros:
            * We do what we want with it without asking anyone for anything
        Cons:
            * 2 years of incubation at best

