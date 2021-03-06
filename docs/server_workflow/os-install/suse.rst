OpenSuse Installation
=======================

RackHD SUSE installation support multiple versions. Please refer to :ref:`os-installation-workflows-label` to see which versions are supported. We'll take openSUSE 42.1 as the example below. If you want to install another version's SUSE, please replace with corresponding version's image, mirror, payload, etc.

Setup Mirror
------------

A mirror should be setup firstly before installation.

* **Local ISO mirror**: Download SUSE ISO image, mount ISO image in a local server as the repository, http service for this repository is provided so that a node could access without proxy.
* **Local sync mirror**: Sync public site's mirror repository to local, http service for this repository is provided so that a node could access without proxy.
* **Public mirror**: The node could access a public or remote site's mirror repository with proxy.

.. include:: mirror-notes.rst

.. tabs::

    .. tab:: Local ISO Mirror

        .. code-block:: shell

            mkdir ~/iso && cd !/iso

            # Download iso file
            http://mirror.clarkson.edu/opensuse/distribution/openSUSE-current/iso/openSUSE-Leap-42.3-DVD-x86_64.iso

            # Create mirror folder
            mkdir -p /var/mirrors/suse

            # Replace {on-http-dir} with your own
            mkdir -p {on-http-dir}/static/http/mirrors

            # Mount iso
            sudo mount openSUSE-Leap-42.3-DVD-x86_64.iso /var/mirrors/suse

            # Replace {on-http-dir} with your own
            sudo ln -s /var/mirrors/suse {on-http-dir}/static/http/mirrors/


    .. tab:: Local Sync Mirror

        For SUSE local mirror, The mirrors are easily made by syncing public SUSE mirror site, on any recent distribution of SUSE:

        .. code-block:: shell

            # Replace xx.x with your own version

            sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/leap/xx.x/repo/oss/ /var/mirrors/suse/distribution/xx.x

            sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/leap/xx.x /var/mirrors/suse/update/leap/xx.x

            sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/leap/xx.x /var/mirrors/suse/update/leap/xx.x

    .. tab:: Public Mirror

        Add following block into httpProxies in ``/opt/monorail/config.json``, and restart on-http service.

        .. code::

            {
              "localPath": "/suse",
              "server": "http://mirror.clarkson.edu/",
              "remotePath": "/opensuse/distribution/leap/42.3/repo/"
            }


Call API to Install OS
-----------------------

After the mirror is setup, We could download payload and call workflow API to install OS.

Get payload example.

.. code-block:: shell

    wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_suse_payload_minimal.json

Call OS installation workflow API to install OS. ``127.0.0.1:9090`` is according to the configuration ``address`` and ``port`` of ``httpEndPoints`` -> ``northbound-api-router`` in ``/opt/monorail/config.json``

.. code-block:: shell

    curl -X POST -H 'Content-Type: application/json' -d @install_suse_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallSUSE | jq '.'

Please record the API's returned result, it's this workflow's Id (like ``342cce19-7385-43a0-b2ad-16afde072715``), it will be used to check result later.

.. include:: install-notes.rst

.. include:: check-result.rst

