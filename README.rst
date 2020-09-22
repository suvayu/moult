.. image:: https://live.staticflickr.com/1278/979979379_935bdb7037_n_d.jpg

Moult
=====

``conda`` is a common way to manage Python environments.
Unfortunately it's pretty terrible at being space efficient.  One
issue being, it does not clean-up old packages.  As a user updates
their environment, new versions of packages are downloaded and made
available in their environment.  However, the old package versions are
not removed, as another environment could be using it.  This means
with time a conda install gets filled up with these orphaned packages;
almost like *conda is moulting* :wink:.

This script uses the following logic to identify orphaned packages,
and removes them.

- List all packages used by all environments managed by ``conda``.  As
  input it expects the conda install directory. ::

    $ tree -L 1 /opt/conda/
    /opt/conda/
    ├── envs
    └── pkgs

  For the above install, it expects ``/opt/conda`` as argument.
  Typically the install directory is in your ``$HOME``, something like
  ``~/.conda``.

- The script then creates a globbing pattern for each package that
  matches the package versions that are not in use by an environment.

  Say if ``env1`` uses ``pkg-1.0``, and ``env2`` uses ``pkg-1.2``, but
  the package versions present are ``pkg-0.9.``, ``pkg-1.0``, and
  ``pkg-1.2``, then ``pkg-0.9`` will be removed, and ``pkg-1.0`` and
  ``pkg-1.2`` will be retained.

- It also creates a pattern that matches all packages, where none of
  the versions of a package are included in an environment.

  Say ``env1`` used to include ``another-pkg-1.1``, but it was
  removed, and no other environment includes it, then it will be
  removed.

- These patterns are then used to delete the orphaned packages (*like
  dead skin from a real :snake:*).


Using the script
----------------

By default the script runs in dry run mode, it will only try to list
the packages that would be deleted.  Since the globbing pattern is
formed as a negative match, you will see errors if there are no
orphaned versions for a particular package.  For the rest, the
orphaned versions are listed.  This way you can confirm what would be
deleted if you were to run the script.

To delete the orphaned packages, you need to pass the ``--run``
option, except this time instead of listing the orphaned versions of
the packages, it deletes them recursively.  The delete command is
hardcoded to be verbose (``rm -r -v`` ) so that you know exactly what
was done in case something goes wrong.  Again, since the pattern is a
negative match, if there are no orphaned version for a package, it
will generate an error from the ``rm`` command, which can be safely
ignored.  I have chosen to not silence these messages as they give
visibility to what is going on, and is useful to debug in case
something goes wrong.

It is advised to log both *stderr* and *stdout* to a log file. ::

  $ moult /opt/conda --run |& tee dead-skin.log


*The social media preview is thanks to Scott Ableman and can be found
on* Flickr_.

.. _Flickr: https://www.flickr.com/photos/ableman/979979379
