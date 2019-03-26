Working with KIWI
=================

.. hint:: **Abstract**

   The following sections describe the general workflow of building
   appliances with KIWI |version|.

.. toctree::
   :maxdepth: 1

   working_with_kiwi/xml_description
..   working_with_kiwi/overlay
..   working_with_kiwi/
..   working_with_kiwi/terminology


Overview
--------

KIWI builds so-called system images (a fully installed and optionally
configured system in a single file) of a Linux distribution in two steps:

1. *Prepare operation*: generate an *unpacked image tree* of your
   image. The unpacked tree is a directory containing the future file
   system of your image, generated from your image description.

2. *Create operation*: the unpacked tree generated in step 1 is packaged
   into the format required for the final usage (e.g. a ``qcow2`` disk
   image to launch the image with QEMU).

KIWI executes these steps using the following components, which it expects
to find in the *description directory*:

#. ``config.xml`` image description

   This XML file is the main and only mandatory component of an image
   description. It contains the general settings of the final image, like
   the partition table, installed packages, present users, etc.

   The filename :file:`config.xml` is not mandatory, the image description
   file can also have an arbitrary name plus the :file:`*.kiwi`
   extension. KIWI will first look for a :file:`config.xml` file and then
   pick the first :file:`*.kiwi`, if it didn't find the former.

#. ``config.sh`` shell script

   The configuration shell script that runs and the end of the *Prepare
   operation* if present. It can be used to fine tune the unpacked image in
   ways that are not possible via the settings provided in
   :file:`config.xml`.

#. ``images.sh`` shell script

   The configuration shell script that runs at the beginning of the *Create
   operation*. It is expected to be used to handle tasks specific to an
   image type.

#. Overlay tree directory

   The *overlay tree* is a folder (called :file:`root`) or a tarball
   (called :file:`root.tar.gz`) that contains files and directories that
   will be copied into the *unpacked image tree* during the *Prepare
   operation*.
   The copying is executed after all the packages included in
   :file:`config.xml` have been installed. Any already present files are
   overwritten.

#. CD root user data

   For live ISO images and install ISO images an optional cdroot archive
   is supported. This is a tar archive matching the name
   :file:`config-cdroot.tar[.compression_postfix]`.

   If present it will be unpacked as user data on the ISO image. This is
   mostly useful to add e.g license files or user documentation on the
   CD/DVD which can be read directly without booting from the media.

#. Archives included in the :file:`config.xml` file.

   The archives that are included in the `<packages>` using the `<archive>`
   subsection:

   .. code:: xml

      <packages type="image">
          <archive name="custom-archive.tgz"/>
      </packages>

