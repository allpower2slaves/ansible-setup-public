# allpower2slaves's ansible-setup
My Ansible thing-collection for my personal use (I set up my desktop systems
with it) . Feel free to reuse different tasks or entire roles from this
repository. Currently most stuff here supports only Fedora, however there is
some OpenSUSE-related stuff. Use at your own risk. Some parts were made as a
result of kitchen-table effort, expect some jank (like tasks tagged with
`btrfs` as well as clever solutions (like the ones found in
`roles/fedora-personal/tasks/gnome-setup.yaml`). Feedback is appreciated :).

There are two repositories. If you are reading this, you are probably looking
at the public version of my Ansible *thing* :). There is a private version of this
repository that I directly edit and regularly use.

# Dependencies
I had these RPM packages installed when I was making and testing parts of this repository:
+ `ansible-collection-community-general-8.4.0-1.fc40.noarch`
+ `ansible-core-2.16.12-1.fc40.noarch`
+ `ansible-9.11.0-1.fc40.noarch`
+ `ansible-srpm-macros-1-16.fc40.noarch`

# Playbooks
There is a reference `play-reference.yaml` playbook. This playbook was modeled after
playbooks that I write and execute on my systems. It's structure was made
deliberately to support `--connection localhost` execution, however it can also
be executed on remote systems as well. 

# Notes
About reboots -- in the reusable `tasks/reboot.yaml` task there set rules for
when to and when not to reboot the target system. The target system **will not**
reboot if:
+ Ansible uses `localhost` connection;
+ `sshd` is not enabled.

# Global variables
Found in `group_vars/all.yaml`:
+ `reboot_timeout` -- timer for Ansible reboot connection timeout. (defaults to 300)

# Handlers
When writing playbooks, i just import a reusable `handlers/main.yaml` list.
Currently, there are two handlers:
+ `reboot` -- for restarting the target system.
+ `relogin` -- for printing out a message suggesting to re-login into the user
  session on target system. (For example, after adding regular users to new groups).

# Roles and their variables

## Personalized roles
Roles that are complex and have very personalized configuration. Usually roles
in this category are changing a lot.

### fedora-personal
This role stores almost all my personal configuration -- from software I
install on my computers to subdirectories in my `$HOME`.
*Some tasks in this role require btrfs filesystem, these tasks have been tagged with `btrfs` tag, so it is easy to skip them.*

#### Variables
+ `main_user` -- first user after successfully installing Fedora Workstation.
  In this case that would be me. (defaults to `{{ ansible_default_id }}`)
+ `apply_nifty_hacks` -- install hardware-related fixes and tweaks, GNOME dconf overrides and other stuff. (defaults to `true`)
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
+ `setup_steam` -- install Valve Software's Steam as a Flatpak application and
  set up `/opt/steam/` btrfs subvolume. (defaults to `false`) 
  *WARNING: due to my laziness, subvolume creation will currently only happen
  with `apply_nifty_hacks` also set to `true`*

## Generic roles
Roles for basic tasks like upgrading packages, rebooting, installing drivers.
Their functionality does not change often and the roles themselves have sane
defaults -- feel free to use them without setting any variables, they will
behave as you would expect.

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
Regular mode of operation also will check for reboot necessity and notify a
handler named `reboot`.

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
    + `suppress_reboot` -- do not notify `reboot` handler;
    + `offline-upgrade` -- do a `dnf offline-upgrade` operation.
    **WARNING:** HIGHLY experimental feature, might not work at all. Also, please set
    `reboot_timeout` variable to have adequate amount of seconds for upgrade
    process to finish.

### flathub-setup
Role for adding Flathub remote (repository) to system installation. *NOTE: this role
will also install `flatpak` package on your sytsem`

#### Variables
+ `flathub_enable` -- enable or **disable** `flathub` remote system-wide. If
  set to `false`, will try to disable flathub remote.
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
+ `gnome_suspend_disable` -- if set to `true`, will install a dconf defaults
  override for disabling suspend on AC power (not running from a battery
  source). Made this one because GDM made my computer suspend all while running
  heavy tasks in tty and over ssh :).
  (defaults to `false`) *WARNING: the same functionality can be found in `fedora-personal` role*

### opensuse-postinstall
Role intented for execution after installing OpenSUSE. Originally it was
intended and tested for use right after installing OpenSUSE Leap 15.6 with "Server" System Role in default(-ish) configuration. It was not tested for use with desktop systems and Tumbleweed (yet).

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

| **tag**      | **desc**                                      |
|--------------|-----------------------------------------------|
| postinstall  | fresh OS installation activities              |
| rpmfusion    | rpmfusion setup                               |
| git          | git stuff :)                                  |
| gpu          | codecs/drivers                                |
| dnf          | dnf package manager                           |
| zypper       | zypper package manager                        |
| ssh          | ssh-related stuff                             |
| flatpak      | flatpak operations (personal)                 |
| flathub      | flathub setup                                 |
| upgrade      | upgrades (package upgrades, etc)              |
| security     | security or privacy oriented changes          |
| reboot       | operations with power, like rebooting etc etc |
| home         | user configuration changes                    |
| podman       | podman stuff                                  |
| nonfree      | non-free software (currently unused)          |
| personal     | custom setup :)                               |
| latex        | latex setup (will download 5+ GB)             |
| remove       | remove files/uninstall packages               |
| btrfs        | btrfs-related stuff (subvolumes, etc)         |
| broken       | tag for stuff that did work but now doesnt    |
| experimental | experimental stuff, use it at your own risk   |

### Unused tags

| **tag**     | **desc**                  |
|-------------|---------------------------|
| interactive | interactive tasks         |
| nonfree     | non-free software         |
| heavy       | bandwith-heavy downloads  |

**WARNING**: some RPM packages are sourced from RPMFusion repostitory and tasks
that install them are not tagged with `rpmfusion` (yet?). Tasks that install
Flatpak applications are tagged with `flathub` because ~~all of Flatpak
applications were sourced from Flathub~~ im lazy and could not bother at the
moment.

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
codecs. ~~And because im lazy to parse rpm -qa output for codec packages.~~

Some tasks with `experimental` tag could be coupled with `never` tag -- to
execute them, explicitely include the tag `experimental`.

To skip adding RPMFusion, Flathub and installing GPU-related stuff, one can
skip `postinstall` tag.
