# hello world
My personal ~~Fedora set up~~ Ansible-related repository. 

This is a open and stripped version (fork?, variant?) of my private Ansible-related
repository. Do not expect working playbooks and comprehensible commit history :).

Feel free to reuse parts of this repo. Any feedback would be welcome. :)

# Dependencies
I had these packages installed on my host:
+ `ansible-collection-community-general-8.4.0-1.fc40.noarch`
+ `ansible-core-2.16.12-1.fc40.noarch`
+ `ansible-9.11.0-1.fc40.noarch`
+ `ansible-srpm-macros-1-16.fc40.noarch`

# Variables
You can edit these global varibles:
+ `reboot_timeout` -- timer for Ansible reconnection after a reboot

# Roles and their variables

## Complex roles
Roles that are complex and have personalized configurations.

### fedora-personal
This is a reference version of my personal role to set up Fedora Workstation
desktop systems. The variant I use is private, however I will try to update
this one as well. Role was made with default Fedora Workstation (the one with GNOME) setup with one user already being created and logged into GNOME.

#### Variables
+ `main_user` -- first user after successfully installing Fedora Workstation.
  In this case that would be me. (defaults to `{{ ansible_default_id }}`)
+ `apply_nifty_hacks` -- install stuff udev rules, system-level GNOME defaults
  and many more configs. (defaults to `true`)
+ `install_packages` -- install and remove packages with `dnf` package manager.
  (defaults to `true`)
+ `install_flatpaks` -- install and remove flatpak applications. (defaults to
  `true`)
+ `skip_dependencies` -- if `true`, dont execute required roles like `fedora-postinstall`
  and `flathub-setup`. (defaults to `false`)
+ `setup_home` -- set up additional users/groups and manage `$HOME` for users.
  (defaults to `true`)
+ `gnome_setup` -- apply GNOME/dconf settings for `main_user`.
  (defaults to `true`)

### fedora-workstation-setup
This role is deprecated and its functionality was split between multiple roles.
Might not work correctly.

## Generic roles
Roles for basic tasks like upgrading packages, rebooting, installing drivers.

### zypper-up
Upgrade your system with `zypper` package manager.

#### Variables
+ `allow_vendor_change` -- allow vendor change when upgrading packages.
  (defaults to `true`)
+ `zypper_up` -- if set to `false`, the role will only check for reboot necessity. 
  (defaults to `true`)
+ `no_recommends` -- if `true`, dont install recommended packages. (defaults to `false`)
+ `reboot_force` -- if reboot is necessary, flush all handlers  (defaults to `false`)
+ `dist_upgrade` -- if `true` run `zypper dup` and not `zypper up`. *NOTE:
  OpenSUSE Tumbleweed cant be upgraded with `zypper up`* (defaults to `true`)


### dnf-upgrade
Role for stuff that is related to `dnf` package manager system upgrade process.  

#### Variables
+ `suppress-reboot` -- if `true`, do not notify `reboot` handler no matter what.
  (defaults to `false`)
+ `mode` -- sets how you would update your system. there are three modes:
    + `upgrade` -- just upgrade your system and then check for reboot necessity
      and notify `reboot`. **default variable for `mode`**;
    + `security` -- same thing as `upgrade`, but fetch and install only
      security-related upgrades;
    + `check-reboot` -- dont upgrade your system, just check for reboot
      necessity and notify `reboot`;
    + `offline-upgrade` -- do a `dnf offline-upgrade` operation.
    **WARNING:** experimental feature, please set `reboot_timeout` variable to
    have adequate amount of seconds for upgrade process to finish.

### flathub-setup
Role for adding Flathub remote (repository) to system installation. *NOTE: this role
will also install `flatpak` package on your sytsem`

#### Variables
+ `flathub_enable` -- enable or disable `flathub` remote system-wide.
  (defaults to `true`)

### fedora-postinstall
This role sets up `RPMFusion` repository and installs NVIDIA drivers (if
needed) and other hardware-related software for playing hardware-accelerated
media.

#### Variables
+ `hardware-setup` -- install GPU-related software (drivers, codecs). *comes
  with NVIDIA proprietary drivers and appications* (defaults to `true`)
+ `force-nvidia` -- install NVIDIA-related packages even when not needed.
+ `kefir` -- install additional packages like `python3-psutil` and other
  libraries (defaults to `true`)

### opensuse-postinstall
Role intented for execution after installing OpenSUSE. Originally it was
intended and tested for use right after installing OpenSUSE Leap 15.6 with "Server" System
Role in default(-ish) configuration. 

This role will pull `zypper-up` role as a dependency -- after testing it became
apparent that installing and setting up SELinux-related software without
upgrading and changing package vendors (in case with Leap 15.6) would break the
system.

**WARNING:** If you set `selinux_setup` to `true`, please ensure that your
`reboot_timeout` value in `reboot` handler is set to adequate value, because
SELinux relabeling can take a lot of time.

#### Variables
+ `sudo_wheel` -- set up `sudo`. (like in other Linux distributions) (defaults to `true`)
+ `main_user` -- user that will be added in `wheel` group. (defaults to `{{ ansible_user_id }}`)
+ `disable_root` -- disable `root` account login. (defaults to `true`)
+ `selinux_setup` -- setup SELinux. It will enforce `targeted` policy.
+ `skip_dependencies` -- if `true`, dont execute required roles like
  `zypper_up`. (defaults to `false`)

## Tags

| tag          | desc                                           |
| ------------ | ---------------------------------------------- |
| postinstall  | fresh OS installation activities               |
|--------------|------------------------------------------------|
| rpmfusion    | rpmfusion setup                                |
| git          | git stuff :)                                   |
| gpu          | codecs/drivers                                 |
| dnf          | dnf package manager                            |
| zypper       | zypper package manager                         |
| ssh          | ssh-related stuff                              |
| flatpak      | flatpak operations (personal)                  |
| flathub      | flathub setup                                  |
| upgrade      | upgrades (package upgrades, etc)               |
| security     | security or privacy oriented changes           |
| reboot       | operations with power, like rebooting etc etc  |
| home         | user configuration changes                     |
| podman       | podman stuff                                   |
| nonfree      | non-free software (currently unused)           |
| personal     | custom setup :)                                |
| latex        | latex setup (will download 5+ GB)              |
| remove       | remove files/uninstall packages                |
| btrfs        | btrfs-related stuff (subvolumes, etc)          |
| broken       | tag for stuff that did work but now doesnt     |
| ------------ | ---------------------------------------------- |
| interactive  | interactive tasks                              |
| experimental | experimental stuff, use it at your own risk    |
| nonfree      | not currently in use                           |
| heavy        | bandwith-heavy downloads, not currently in use |

**WARNING**: some RPM packages are sourced from RPMFusion repostitory and tasks
that install them are not tagged with `rpmfusion` (yet?). Tasks that install
Flatpak applications are tagged with `flathub` because all of Flatpak
applications were sourced from Flathub.

For example, one might skip `upgrade` to forbid upgrading any software on the
target system. To save network bandwich, one might skip `flatpak` tag to forbid
downloading Flatpak applications (`flathub` tag is about setting up Flathub
repository and there is no reason to skip it unless you don't want Flathub
repository to be present and active on your system).

Keep in mind that even if you skip `upgrade` tag -- not skipping `gpu`-tagged
tasks (codecs and drivers installation) (also might be tagged with
`postinstall` and `dnf`) might result in running an upgrade on `@multimedia`
group. This was a deliberate choice due not to tag it with `upgrade` to
possibility that a system could have `ffmpeg` installed but not the required
codecs. *And because im lazy to parse rpm -qa output for codec packages.*

Some tasks with `experimental` tag could be coupled with `never` tag -- to
execute them, explicitely include the tag `experimental`.

To skip adding RPMFusion, Flathub and installing GPU-related stuff, one can
skip `postinstall` tag.

## Notes
Role will only execute reboots if two of these things are true:
+ Target is not a host machine (`--connection local`)
+ `sshd` system unit has `enabled` status (so that Ansible will not lose connection)
