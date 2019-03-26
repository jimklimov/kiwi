The Image Description
=====================

The image description file is a XML file that defines the basic aspects of
the appliance that will be build by KIWI, for example:

- image type (e.g. QEMU disk image, PXE bootable image, Vagrant box, etc.)
- partition layout
- packages to be installed on the system
- users to be added

The following sections will walk you through the major elements and
attributes of the XML schema. A complete description of the schema can be
found in :ref:`schema-docs`.


The `image` Element
-------------------

The image description lives under the top-level `image` element, all
additional settings will thus be children of it:

.. code-block:: xml

   <?xml version="1.0" encoding="utf-8"?>

   <image schemaversion="6.9" name="LimeJeOS-Leap-15.0">
       <!-- all setting belong here -->
   </image>

The `image` element requires the following two attributes (as shown in the
above example):

- `name`: A name for this image that must not contain spaces or `/`.

- `schemaversion`: The version of the XML schema that should be used. If in
  doubt, use the latest available version.

The `name` attribute will be used to create the bootloader entry, however
it can be inconvenient to use as it must be POSIX-safe. You can therefore
provide an alternative name that will be displayed in the bootloader via
the attribute `displayName`, which doesn't have the same strict rules as
`name`:

.. code-block:: xml

   <?xml version="1.0" encoding="utf-8"?>

   <image schemaversion="6.9" name="LimeJeOS-Leap-15.0" displayName="LimeJeOS Leap 15.0">
       <!-- all setting belong here -->
   </image>


The `description` section
-------------------------

The first element inside the `image` element is by convention the
`description` element, which contains some high level information about
your image:

.. code-block:: xml

   <?xml version="1.0" encoding="utf-8"?>

   <image schemaversion="6.9" name="LimeJeOS-Leap-15.0">

     <description type="system">
       <author>Jane Doe</author>
       <contact>jane@myemaildomain.xyz</contact>
       <specification>
         Leap 15.0 JeOS, a small image, that is just enough of an OS
       </specification>
       <license>GPLv3</license>
     </description>

     <!-- snip -->
   </image>


The `description` element must always contain a `type` attribute. This
attribute accepts the values `system` or `boot`. The value `boot` is used
by the KIWI developers and is not relevant for the end user, thus `type`
should be always set to `system`.

`description` allows the following optional sub-elements to be present:

- `author`: The name of the author of this image.

- `contact`: Some means how to contact the author of the image (e.g. an
  email address, an IM nickname and network, etc.)

- `specification`: A detailed description of this image, e.g. its use case.

- `license`: If applicable, you can specify a license for the image in this
  field.


The `preferences` section
-------------------------

The mandatory `preferences` section contains the definition of the various
enabled image types (so-called build types). Each of these build types can
be supplied with attributes specific to that image type, which we described
section :ref:`xml-description-build-types`.

The elements that are not image type specific are presented afterwards in
section :ref:`xml-description-preferences-common-elements`.


.. _xml-description-build-types:

Build types
^^^^^^^^^^^

A build type defines the type of appliance that is produced by KIWI, for
instance a live ISO image or a virtual machine disk. For the

For example a live ISO image is specified as follows:

.. code-block:: xml

   <?xml version="1.0" encoding="utf-8"?>

   <image schemaversion="6.9" name="LimeJeOS-Leap-15.0">
     <preferences>
       <type image="iso" primary="true" flags="overlay" hybridpersistent_filesystem="ext4" hybridpersistent="true"/>
       <!-- additional preferences -->
     </preferences>

     <!-- additional image settings -->
   </image>


A build type is defined via a single `type` element whose only required
attribute is `image`, which defines which image will be created. All other
attributes are optional and can be used to customize a image further. In
the above example we created an ISO image, with the an ext4
hybrid-persistent filesystem [#f1]_.

It is possible to provide **multiple** `type` elements with different
`image` attributes inside the preferences section. For instance the
following XML snippet can be used to create a live image, an OEM
installation image and a virtual machine version of the same appliance:

.. code-block:: xml

   <?xml version="1.0" encoding="utf-8"?>

   <image schemaversion="6.9" name="LimeJeOS-Leap-15.0">
     <preferences>
       <!-- Live ISO -->
       <type image="iso" primary="true" flags="overlay" hybridpersistent_filesystem="ext4" hybridpersistent="true"/>

       <!-- Virtual machine -->
       <type image="vmx" filesystem="ext4" bootloader="grub2" kernelcmdline="splash" firmware="efi"/>

       <!-- OEM installation image -->
       <type image="oem" filesystem="ext4" initrd_system="dracut" installiso="true" bootloader="grub2" kernelcmdline="splash" firmware="efi">
         <oemconfig>
           <oem-systemsize>2048</oem-systemsize>
           <oem-swap>true</oem-swap>
           <oem-device-filter>/dev/ram</oem-device-filter>
           <oem-multipath-scan>false</oem-multipath-scan>
         </oemconfig>
         <machine memory="512" guestOS="suse" HWversion="4">
         </machine>
       </type>
       <!-- additional preferences -->
     </preferences>

     <!-- additional image settings -->
   </image>


KIWI supports the following values for the `image` attribute, further
relevant attributes to `type` are documented inside the referenced
sections:

- `iso`: a live ISO image, see :ref:`hybrid_iso`
- `vmx`: build a virtual machine image, see: :ref:`vmx`
- `oem`: results in an expandable image that can be deployed via a bootable
  installation medium, e.g. a USB drive or a CD. See :ref:`oem`
- `pxe`: creates an image that can be booted via PXE (network boot), see
  :ref:`build_pxe`

- `docker`, `oci`: container images, see :ref:`building-docker-build`

- `btrfs`, `ext2`, `ext3`, `ext4`, `xfs`: KIWI will convert the
  image into a mountable filesystem of the specified type.

- `squashfs`, `clicfs`: creates the image as a filesystem that can be used
  on live systems

- `tbz`, `cpio`: the unpacked source tree will be compressed into a `XZ
  <https://en.wikipedia.org/wiki/Xz>`_ or `CPIO
  <https://en.wikipedia.org/wiki/Cpio>`_ archive.


The `type` element furthermore supports the following subelements (as shown
above, `oemconfig` is a subelement of `<type image="oem" ...>`):

- `containerconfig`: contains settings specific for the creation of
  container images, see :ref:`building-docker-build`

- `oemconfig`: configurations relevant for building OEM images, see:
  :ref:`oem`

- `pxedeploy`: settings for PXE booting, see :ref:`build_pxe`

- `vagrantconfig`: instructs KIWI to build a Vagrant box instead of a
  standard virtual machine image, see :ref:`setup_vagrant`

Please consult the :ref:`schema-docs` for concerning the documentation of
the remaining child-elements: `machine`, `size` and `systemdisk`.


.. _xml-description-preferences-common-elements:

Common elements
^^^^^^^^^^^^^^^


.. _xml-description-image-profiles:

Image profiles
--------------


Adding users
------------

User accounts can be added or modified via the `users` element, which
supports a list of multiple `user` sub-elements:

.. code-block:: xml

   <?xml version="1.0" encoding="utf-8"?>

   <image schemaversion="6.9" name="JeOS-Tumbleweed">
     <users>
       <user password="this_is_soo_secure" home="/home/me" name="me" groups="users" pwdformat="plain" />
       <user password="$1$wYJUgpM5$RXMMeASDc035eX.NbYWFl0" home="/root" name="root" groups="root"/>
     </users>

   </image>

Each `user` element represents a specific user that should be added or
modified. The following attributes are mandatory:

- `name`: the UNIX username

- `home`: the path to the user's home directory

Additionally the following optional attributes can be specified:

- `groups`: A comma separated list of groups. The first element of the list
  is used for the user's primary group. The remaining elements are appended
  to the user's supplementary groups. When no groups are assigned then the
  system's default primary group will be used [#f2]_

- `id`: The user-id of this account

- `pwdformat`: The format in which `password` is provided, either `plain`
  or `encrypted` (the latter is the default)

- `password`: The password for this user account. It can be provided either
  in cleartext form (`pwdformat="plain"`) or in `crypt`'ed form
  (`pwdformat="encrypted"`). Plain passwords are discouraged, as everyone
  with access to the image description would know the password. Thus please
  generate a hash of your password, e.g. with the `mkpasswd` tool
  (available in most Linux distributions via the `whois` package):

  .. code:: bash

     $ mkpasswd -m sha-512 -S $(date +%N) -s <<< INSERT_YOUR_PASSWORD_HERE


Defining repositories and packages
----------------------------------


.. [#f1] This option results in the creation of a copy-on-write file on the
         filesystem to keep data persistent over a reboot.

.. FIXME: @schaefi: is this correct?
.. [#f2] Note that the default primary group is taken from the system on
         which KIWI is run, not from the system that KIWI is building.
