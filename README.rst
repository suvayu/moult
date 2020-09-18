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

- It also creates a pattern that matches all *orphaned packages*,
  where none of the versions of a package are included in an
  environment.

- These patterns are then used to delete the orphaned packages (*like
  dead skin from a real :snake:*).


Using the script
----------------

By default the script runs in dry run mode, it will only try to list
the packages that would be deleted.  Since the globbing pattern is
formed as a negative match, you will see errors if there are no older
versions for a particular package.  For the rest, the older versions
are listed.  This way you can confirm what would be deleted if you
were to run the script.

To run the script you need to pass the ``--run`` option, this repeats
the same exercise, except this time instead of listing the old
versions of the packages, it deletes them recursively.  The delete
command is hardcoded to be verbose (``rm -r -v`` ), so that you know
exactly what was done in case something goes wrong.  It is advised to
log both *stderr* and *stdout* to a log file. ::

  $ moult /opt/conda --run |& tee clean.log
