# qubes-third-party-integration

Documentation regarding issues and possible solutions for integration of
third-party software in Dom0.

This is a WIP, not ready to be discussed. Reviewers are greatly appreciated.

## Table of contents

* [Assumptions](#assumptions)
* [File archive](#file-archive)
* [RPM repository](#rpm-repository)
* [Git repository - bundle](#git-repository---bundle)
* [Git repository - clone over Qrexec](#git-repository---clone-over-qrexec)
* [Git repository - copying directory](#git-repository---copying-directory)
* [QubesOS-contrib package](#qubesos-contrib-package)

## Assumptions

We need to make some assumptions:

1. The package is signed somehow;
2. The user trusts the key they have imported the Dom0;
3. The user trust the vendor providing the package.

Once you have bootstrapped your package to Dom0, future updates are easier and
may not require user interaction. You can also do an update mechanism safer
than the installation mechanism, but if the first installation was
compromised, future updates are also compromised.

## File archive

There are various types of file archives, for simplicity we will deal only
with `tar`, but the same applies to `rpm` files. Assuming the user trusts the
vendor, they trust everything inside that tarball, issues regarding `tar`
vulnerabilities placing files outside of the desired directory are out of
scope, as once the tarball has been verified, assuming the tarball is
malicious contradicts trusting the vendor.

The user needs to copy 2 to 3 files to Dom0:

- The archive and the detached signature of the archive; or
- The archive, the checksum of the archive and the detached signature of the
  checksum.

Benefits:

- Copying these files to Dom0 can easily be done with `qvm-run --pass-io`.

Problems:

- There is no easy update mechanism.

## RPM repository

RPM repositories are similar to file archives.

Benefits:

- Easiest update method.

Problems:

- User has to do more work upfront by editing repository definitions and
  importing the key to the RPM key database;
- Maintainer has to keep a server running, burden of maintenance costs and
  time.

## Git repository - bundle

This option regards to `git-bundle`. User clones your repository, creates a
bundle in DomU and pass the file to Dom0.

Benefits:

- Very easy to do;

Problems:

- There is no way to sanitize the bundle, it can print arbitrary characters to
  your Dom0 terminal when cloning, before you can even do signature
  verification of the repository.
- There is no easy update mechanism.

## Git repository - clone over Qrexec

Cloning over Qrexec is nice! The safest solution is
[woju/qubes-app-split-git](https://github.com/woju/qubes-app-split-git), it
can fetch only signed tags but not commits. Other implementations are bare
bones and don't sanitize data received from the remote.

Benefits:

- Available in the contrib repository, thus easily added to Dom0;
- You can have a trusted repository in Dom0 instead of only release files.

Problems:

- Git SHA1 collision applies;
- Can only fetch tags.

## Git repository - copying directory

Copying a literal directory can be done with `qfile-dom0-unpacker`. Copying a
`git` repository might be interesting.

Benefits:

- Native qubes method sanitizing file names;
- Can verify signatures from commits or tags with git.

Problems:

- Very long command, prone to user making typos;
- Untrusted files in `.git/` can make trusting the repository difficult, it
  can set configuration options, aliases, signature trust level, exclude
  modified files from being tracked, thus making signature verification
  difficult to validate.
- There is no easy update mechanism.

## QubesOS-contrib package

The organization [QubesOS-contrib](https://github.com/qubesos-contrib) is
controlled by the Qubes OS Team. Inclusion of new packages is very low. Qubes
OS Team doesn't want to include unreviewed formulas and as they don't have the
bandwidth to review, they don't include any formula.

Benefits:

- Easy for the user to install your package.

Problems:

- Depends on Qubes OS Team having time and resources to review your solution,
  this can lead to delays of inclusion or never being added at all.
