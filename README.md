# How to install DaVinci Resolve on CentOS

Here are some notes on how to install  DaVinci Resolve 16 on CentOS Linux. Because software is constantly changing, [these notes are hosted on GitHub Pages](https://github.com/sethgoldin/install-davinci-resolve-centos). If you find something wrong or outdated, please do open a pull request.

These particular notes were originally worked out from an installation to an HP Z8 G4 workstation with a single GTX 1080 Ti card installed, but the information should be useful for other systems running `x86_64` CPUs and NVIDIA GPUs.

## Choosing a major release of CentOS

Resolve 16.1.2 works great on both CentOS 7.7 and 8.1, but each has its costs and benefits.

### 7.7
Benefits:
- This is the most stable and best-supported platform:
	- Third-party repositories are mature, with an established ecosystem with many packages and good documentation
	- [End-of-life is June 30, 2024](https://wiki.centos.org/FAQ/General#What_is_the_support_.27.27end_of_life.27.27_for_each_CentOS_release.3F). This is inherited from the [end of Maintenance Support 2 of RHEL 7](https://access.redhat.com/support/policy/updates/errata).
Costs:
- GNOME 2 is the default desktop environment
- Packages can be quite old, so getting more up-to-date packages may depend on [third-party repositories](https://wiki.centos.org/AdditionalResources/Repositories)
- Although kernels are [backported](https://access.redhat.com/security/updates/backporting) with security updates, they are quite old
### 8.1
Benefits:
- Packages are newer and are distributed via the [AppStream infrastructure](https://www.freedesktop.org/wiki/Distributions/AppStream/)
- GNOME 3 is the default desktop environment
- Newer kernels
Costs:
- [Desktop Video 11.4 does not work on 8.1](https://github.com/sethgoldin/install-davinci-resolve-centos/issues/25)
	- You could, in theory go source 8.0.1905, on which Desktop Video 11.4 did indeed work, but this is _unsupported_ and inadvisable.
