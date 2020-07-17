# How to install DaVinci Resolve on CentOS

Here are some notes on how to install  DaVinci Resolve 16 on CentOS Linux. Because software is constantly changing, [these notes are hosted on GitHub Pages](https://github.com/sethgoldin/install-davinci-resolve-centos). If you find something wrong or outdated, please do open a pull request.

These particular notes were originally worked out from installations to an HP Z8 G4 workstations with single GTX 1080 Ti or RTX 2080 Ti cards installed, but the information should be useful for other systems running `x86_64` CPUs and NVIDIA GPUs.

## Choosing a major release of CentOS

Resolve 16.2.4 works great on both CentOS 7.8 and 8.2, but each has its costs and benefits. Carefully consider what you need for your system before deciding which [major](https://access.redhat.com/solutions/401413) release to install.

### CentOS 7.8
Benefits:
- This is the most stable and best-supported platform:
	- Third-party repositories are mature, with an established ecosystem, filled with many packages and good documentation
- [End-of-life is June 30, 2024](https://wiki.centos.org/FAQ/General#What_is_the_support_.27.27end_of_life.27.27_for_each_CentOS_release.3F). This is inherited from the [end of Maintenance Support 2 of RHEL 7](https://access.redhat.com/support/policy/updates/errata).

Costs:
- GNOME 3.28.2 is the default desktop environment, and it's somewhat older
- Many packages can be quite old, so getting more up-to-date packages may depend on [third-party repositories](https://wiki.centos.org/AdditionalResources/Repositories)
- Although kernels are [backported](https://access.redhat.com/security/updates/backporting) with security updates, they are quite old

### CentOS 8.2
Benefits:
- Packages are newer and are distributed via the [AppStream infrastructure](https://www.freedesktop.org/wiki/Distributions/AppStream/)
- GNOME 3.32.2 is the default desktop environment
- Newer kernels provide better performance
- End-of-life is May 2029. This is inherited from the [end of Maintenance Support 2 of RHEL 8](https://access.redhat.com/support/policy/updates/errata). 

Costs:
- CentOS 8 ships with Wayland, but [Wayland does not yet play nice with NVIDIA](https://wiki.gnome.org/Initiatives/Wayland/NVIDIA), so the legacy X environment must be used instead
- Many packages available on 7 have not yet been released or tested on 8

## Instructions
With all these costs and benefits in mind, here are instructions for both major releases of CentOS:
- [CentOS 7.8](centos-7.8.md)
- [CentOS 8.2](centos-8.2.md)

## PostgreSQL
If you'd like to set up your 7.8 workstation as a PostgreSQL server for other workstations, you can adapt [these instructions for setting up an Intel NUC](https://medium.com/@sethgoldin/how-to-set-up-an-intel-nuc-as-a-postgresql-server-for-davinci-resolve-studio-workstations-b36dff0a1872).
