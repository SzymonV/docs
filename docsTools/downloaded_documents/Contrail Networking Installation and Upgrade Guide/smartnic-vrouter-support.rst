Using Netronome SmartNIC vRouter with Contrail Networking
=========================================================

 **Note**

The Netronome SmartNIC vRouter technology covered in this document is
available for evaluation purposes only. It is not intended for
deployment in production networks.

Contrail supports Netronome Agilio CX SmartNICs for Contrail Networking
deployment with Red Hat OpenStack Platform Director (RHOSPd) 13
environment.

This feature will enable service providers to improve the forwarding
performance which includes packets per second (PPS) of vRouter. This
will optimize server CPU usage and you can deploy more Virtual network
functions (VNFs) per server.

Benefits:

-  Increased PPS capacity of Contrail vRouter datapath allowing
   applications to reach their full processing capacity.

-  Reclaimed CPU cores from Contrail vRouter off-loading allowing more
   VMs and VNFs to be deployed per server.

The goal of this topic is to provide a procedure for deploying
accelerated vRouter compute nodes.

Before you begin:

-  Equip compute nodes with Netronome Agilio CX SmartNIC.

   For details, refer to `Agilio CX
   SmartNICs <https://www.netronome.com/products/agilio-cx/>`__.

-  Retrieve *Agilio heat-template plugin*.

   Register on Netronome support site at https://help.netronome.com and
   provide Docker Hub credentials.

   Netronome will provide the TripleO templates for SmartNIC vRouter
   deployment on compute nodes. Also, Netronome will authorize Docker
   Hub registry access.

   For details, refer to `Netronome Agilio vRouter 19\ ``xx`` deployment
   guide <https://github.com/netronome-support/Agilio-vRouter-19xx/wiki/Agilio-vRouter-19xx-deployment-guide-%5BRHEL-7.6%5D%5BRHOSP-13%5D>`__.

-  Note the following version tags:

   ``AGILIO_TAG="2.38-rhel-queensFORWARDER_TAG="2.38-rhel-queens``

Procedure:

**Note**

If you have multiple undercloud nodes deployed, you must perform the
following procedure on the same node.

1. Configure *Agilio* plugin.

   For details, refer to `Netronome agilio-ovs-openstack-plugin GitHub
   Repository <https://github.com/Netronome/agilio-ovs-openstack-plugin>`__.

   1. Extract the *Agilio* plugin archive and copy the ``agilio-plugin``
      folder into the ``tripleo-heat-templates`` directory.

      ``[stack@queensa ~]$ tar -xzvf rhosp-contrail-agilio-heat-plugin-5-34.tgzagilio-plugin/agilio-plugin/agilio-vrouter.yamlagilio-plugin/agilio_upgrade.shagilio-plugin/deploy_rhosp.shagilio-plugin/nfp_udev.shagilio-plugin/agilio-env.yamlagilio-plugin/versionagilio-plugin/README.md[stack@queensa ~]$ cp -r agilio-plugin/ tripleo-heat-templates/``

   2. Navigate to the ``agilio-plugin`` directory on the undercloud
      node.

      ``[tripleo-heat-templates]$ cd agilio-plugin/``

   3. Modify ``agilio-env.yaml`` file as per your
      environment.\ **Note**\ 

      Reserve at least 1375*2 MB hugepages for ``virtio-forwarder``.

      .. raw:: html

         <div id="jd0e143" class="sample" dir="ltr">

      Sample ``agilio-env.yaml`` file:

      .. raw:: html

         <div class="output" dir="ltr">

      ::

         resource_registry:
           OS::TripleO::NodeExtraConfigPost: agilio-vrouter.yaml

         parameter_defaults:
           # Hugepages
           ContrailVrouterHugepages2MB: "8192"
           # IOMMU
           ComputeParameters:
             KernelArgs: "intel_iommu=on iommu=pt isolcpus=1,2 " 

           ComputeCount: 3

           # Aditional config
           ControlPlaneDefaultRoute: 10.0.x.1
           EC2MetadataIp: 10.0.x.1  # Generally the IP of the Undercloud
           DnsServers: ["8.8.8.8","192.168.3.3"]
           NtpServer: ntp.is.co.za
           ContrailRegistryInsecure: true
           DockerInsecureRegistryAddress: 172.x.x.150:6666,10.0.x.1:8787
           ContrailRegistry: 172.x.x.150:6666
           ContrailImageTag: <container_tag>-rhel-queens

         # Fix DB Diskspace too low issue
           ContrailAnalyticsDBMinDiskGB: 40

      .. raw:: html

         </div>

      .. raw:: html

         </div>

   4. Add Docker Hub credentials to
      ``tripleo-heat-templates/agilio-plugin/agililo_upgrade.sh`` file
      to retrieve containers from
      ``AGILIO_REPO="docker.io/netronomesystems/"`` repository.

      ``#GENERAL DOCKER CONFIGDOCKER_USR=****** #ENTER_DOCKER_USERNAME_HEREDOCKER_PASS=****** #ENTER_DOCKER_PASSWORD_HERE``

      ``[root@overcloud-novacompute-2 heat-admin]# docker ps -a | grep virtio_for7d5af8a2591d        docker.io/netronomesystems/virtio-forwarder:2.38-rhel-queens           "./entrypoint.sh"        30 seconds ago      Up 15 seconds virtio_forwarder``

      ``[root@overcloud-novacompute-2 heat-admin]# docker ps -a | grep agilioc7c611b5168b        docker.io/netronomesystems/agilio-vrouter:2.38-rhel-queens             "./entrypoint.sh"        46 seconds ago      Up 38 seconds agilio_vrouter``

2. Prepare the Contrail Networking cluster for deployment.

   Refer to the following topics for deployment:

   -  `Understanding Red Hat OpenStack Platform
      Director <../../topic-map/setting-up-contrail-rhosp-introduction.html>`__

   -  `Setting Up the
      Infrastructure <../../topic-map/setting-up-contrail-rhosp-infrastructure.html>`__

   -  `Setting Up the
      Undercloud <../../topic-map/setting-up-contrail-rhosp-undercloud.html>`__

   -  `Setting Up the
      Overcloud <../../topic-map/setting-up-contrail-rhosp-overcloud.html>`__

      **Note**

      Do not perform steps for `Installing
      Overcloud <../../topic-map/setting-up-contrail-rhosp-overcloud.html#id-installing-overcloud>`__.

3. Deploy the cluster by one of the following ways:

   -  Add ``agilio-env.yaml`` to installing overcloud step as mentioned
      in `Installing
      Overcloud <../../topic-map/setting-up-contrail-rhosp-overcloud.html#id-installing-overcloud>`__
      topic.

      ``openstack overcloud deploy --templates ~/tripleo-heat-templates-e ~/overcloud_images.yaml-e ~/tripleo-heat-templates/environments/network-isolation.yaml-e ~/tripleo-heat-templates/environments/contrail/contrail-plugins.yaml-e ~/tripleo-heat-templates/environments/contrail/contrail-services.yaml-e ~/tripleo-heat-templates/environments/contrail/contrail-net.yaml-e ~/tripleo-heat-templates/agilio-plugin/agilio-env.yaml--roles-file ~/tripleo-heat-templates/roles_data_contrail_aio.yaml``

      Or

   -  Run the following command:

      ``deploy_rhosp.sh``

      ``-e ~/tripleo-heat-templates/agilio-plugin/agilio-env.yaml``

On completing above steps successfully, refer to `Netronome
agilio-ovs-openstack-plugin GitHub
Repository <https://github.com/Netronome/agilio-ovs-openstack-plugin>`__
on how to spin up the accelerated VMs.

 
