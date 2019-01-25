# rotcheck

A simple shell script to recursively generate checksums for files you care
about and verify them. It's **useful for detecting bit rot**.

It's written in POSIX shell, but requires GNU coreutils, BusyBox or some other
collection that includes similar checksum tools.

## Advantages compared with other tools

Simple and should work forever (because it's POSIX shell). It's also trivial
to edit the code for your own needs.

## Disadvantages compared with other tools

Verifying checksums is just as fast as any other tool, but generating
or updating checksums is slower (because it's POSIX shell).

Maybe use a faster tool if you have millions of files and regularly update your
checksums.

# Installation

Download [this
script](https://raw.githubusercontent.com/jamielinux/rotcheck/master/rotcheck)
to somewhere in your `$PATH`. For example:

```
curl -O https://raw.githubusercontent.com/jamielinux/rotcheck/master/rotcheck
sudo cp rotcheck /usr/local/bin/rotcheck
sudo chmod +x /usr/local/bin/rotcheck
```

# Usage

```
Usage: rotcheck MODE [OPTIONS]
   or: rotcheck MODE [OPTIONS] -- [DIRECTORY]... [ARBITRARY FIND OPTION]...
Recursively generate, update and verify checksums.

MODES:
 -a           APPEND-ONLY mode: Record checksums for any files without
              a checksum already. Never modify existing checksums.
 -c           CHECK-ONLY mode: Check that files checksums are the same.
 -d           DELETE mode: Remove checksums for files that don't exist.
 -u           APPEND-AND-UPDATE mode: Like append-only mode, but also
              update checksums for files with a modification date newer
              than the checksum file. (NB: Also see `-M`.)

OPTIONS:
 -b COMMAND   Checksum command to use. Default: sha512sum
 -f FILE      File to store checksums. Default: ./.rotcheck
 -h           Display this help.
 -n           Don't follow symlinks. The default is to follow symlinks.
 -q           Suppress "OK" messages when checking checksums.
 -v           Be more verbose when adding, deleting or changing checksums.
 -w           Warn about improperly formatted checksum lines.
 -M           Use with `-u` to update checksums regardless of modification
              time. This is *much* slower so avoid if possible; try using
              `touch` instead to bump the modification time of the file.
              WARNING: The checksums might have changed due to bit rot so
              use this option with care!

 (specific to GNU coreutils)
 -i           Ignore missing files when verifying checksums.


Supported commands:
  GNU coreutils:
    md5sum, sha1sum, sha224sum, sha256sum, sha384sum, sha512sum, b2sum

  BusyBox (applets must be symlinked):
    md5sum, sha1sum, sha256sum, sha512sum, sha3sum

  BSD & macOS (install GNU coreutils):
    gmd5sum, gsha1sum, gsha224sum, gsha256sum, gsha384sum, gsha512sum, gb2sum


Examples:
  # Generate checksums for the first time, storing in ./.rotcheck file.
  # If ./.rotcheck exists, append checksums for files with no checksum yet.
  rotcheck -a

  # Verify checksums in the ./.rotcheck file:
  rotcheck -c

  # Regenerate checksums with b2sum (BLAKE2) instead of sha512sum:
  rm ./.rotcheck
  rotcheck -b b2sum -a
  rotcheck -b b2sum -c

  # Search other directories instead of the current directory.
  # WARNING: checksums might get duplicated if mixing relative and absolute
  # paths, or if you change the way you specify directory paths!
  rotcheck -a -- /mnt/archive-2018/ /mnt/archive-2019/

  # Exclude .git folders (these arguments are passed directly to find):
  rotcheck -a -- ! -path '*/.git/*'

```
