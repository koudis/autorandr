# autorandr

Automatically select a display configuration based on connected devices

## Branch information

This is a compatible Python rewrite of
[wertarbyte/autorandr](https://github.com/wertarbyte/autorandr). Contributions
for bash-completion, fd.o/XDG autostart, Nitrogen, pm-utils, and systemd can be
found under [contrib](contrib/).

The original [wertarbyte/autorandr](https://github.com/wertarbyte/autorandr)
tree is unmaintained, with lots of open pull requests and issues. I forked it
and merged what I thought were the most important changes. If you are searching
for that version, see the [`legacy` branch](https://github.com/phillipberndt/autorandr/tree/legacy).
Note that the Python version is better suited for non-standard configurations,
like if you use `--transform` or `--reflect`. If you use `auto-disper`, you
have to use the bash version, as there is no disper support in the Python
version (yet). Both versions use a compatible configuration file format, so
you can, to some extent, switch between them.  I will maintain the `legacy`
branch until @wertarbyte finds the time to maintain his branch again.

If you are interested in why there are two versions around, see
[#7](https://github.com/phillipberndt/autorandr/issues/7),
[#8](https://github.com/phillipberndt/autorandr/issues/8) and
especially
[#12](https://github.com/phillipberndt/autorandr/issues/12)
if you are unhappy with this version and would like to contribute to the bash
version.

## License information and authors

autorandr is available under the terms of the GNU General Public License
(version 3).

Contributors to this version of autorandr are:

* Adrián López
* andersonjacob
* Alexander Wirt
* Brice Waegeneire
* Chris Dunder
* Christoph Gysin
* Christophe-Marie Duquesne
* Daniel Hahler
* Maciej Sitarz
* Mathias Svensson
* Matthew R Johnson
* Nazar Mokrynskyi
* Phillip Berndt
* Rasmus Wriedt Larsen
* Simon Wydooghe
* Stefan Tomanek
* stormc
* tachylatus
* Timo Bingmann
* Timo Kaufmann
* Tomasz Bogdal
* Victor Häggqvist
* Jan-Oliver Kaiser
* Alexandre Viau

## Installation/removal

You can use the `autorandr.py` script as a stand-alone binary. If you'd like to
install it as a system-wide application, there is a Makefile included that also
places some configuration files in appropriate directories such that autorandr
is invoked automatically when a monitor is connected or removed, the system
wakes up from suspend, or a user logs into an X11 session. Run `make install`
as root to install it.

If you prefer to have a system wide install managed by your package manager,
you can

* Use the [official Arch package](https://www.archlinux.org/packages/community/any/autorandr/).
* Use the [official Debian package](https://packages.debian.org/sid/x11/autorandr) on sid
* Use the [FreeBSD Ports Collection](https://www.freshports.org/x11/autorandr/) on FreeBSD.
* Use the [ebuild from zugaina](https://gpo.zugaina.org/x11-misc/autorandr) on Gentoo.
* Use the
  [nix package](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/services/misc/autorandr.nix)
  on NixOS.
* Use the
  [guix package](https://git.savannah.gnu.org/cgit/guix.git/log/gnu/packages/xdisorg.scm?qt=grep&q=autorandr)
  on Guix.
* Use the [SlackBuild](https://slackbuilds.org/repository/14.2/desktop/autorandr/) on Slackware.
* Use the automated nightlies generated by the
  [openSUSE build service](https://build.opensuse.org/package/show/home:phillipberndt/autorandr)
  for various distributions (RPM and DEB based).
* Use the [X binary package system](https://wiki.voidlinux.eu/XBPS)' on Void Linux 
* Build a .deb-file from the source tree using `make deb`.
* Build a .rpm-file from the source tree using `make rpm`.

We appreciate packaging scripts for other distributions, please file a pull
request if you write one.

If you prefer `pip` over your package manager, you can install autorandr with:

    sudo pip install "git+http://github.com/phillipberndt/autorandr#egg=autorandr"

or simply

    sudo pip install autorandr

if you prefer to use a stable version.

## How to use

Save your current display configuration and setup with:

    autorandr --save mobile

Connect an additional display, configure your setup and save it:

    autorandr --save docked

Now autorandr can detect which hardware setup is active:

    $ autorandr
      mobile
      docked (detected)

To automatically reload your setup:

    $ autorandr --change

To manually load a profile:

    $ autorandr --load <profile>

or simply:

    $ autorandr <profile>

autorandr tries to avoid reloading an identical configuration. To force the
(re)configuration:

    $ autorandr --load <profile> --force

To prevent a profile from being loaded, place a script call _block_ in its
directory. The script is evaluated before the screen setup is inspected, and
in case of it returning a value of 0 the profile is skipped. This can be used
to query the status of a docking station you are about to leave.

If no suitable profile can be identified, the current configuration is kept.
To change this behaviour and switch to a fallback configuration, specify
`--default <profile>`. The system-wide installation of autorandr by default
calls autorandr with a parameter `--default default`. There are three special,
virtual configurations called `horizontal`, `vertical` and `common`. They
automatically generate a configuration that incorporates all screens
connected to the computer. You can symlink `default` to one of these
names in your configuration directory to have autorandr use any of them
as the default configuration without you having to change the system-wide
configuration.

You can store default values for any option in an INI-file located at
`~/.config/autorandr/settings.ini`. In a `config` section, you may place any
default values in the form `option-name=option-argument`.

A common and effective use of this is to specify default `skip-options`, for
instance skipping the `gamma` setting if using
[`redshift`](https://github.com/jonls/redshift) as a daemon.  To implement
the equivalent of `--skip-options gamma`, your `settings.ini` file should look
like this:

```
[config]
skip-options=gamma
```

## Advanced usage

### Hook scripts

Three more scripts can be placed in the configuration directory
(as defined by the [XDG spec](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html),
usually `~/.config/autorandr` or `~/.autorandr` if you have an old installation
for user configuration and `/etc/xdg/autorandr` for system wide configuration):

- `postswitch` is executed *after* a mode switch has taken place. This can be
  used to notify window managers or other applications about the switch.
- `preswitch` is executed *before* a mode switch takes place.
- `postsave` is executed after a profile was stored or altered.
- `predetect` is executed before autorandr attempts to run xrandr. 

These scripts must be executable and can be placed directly in the configuration
directory, where they will always be executed, or in the profile subdirectories,
where they will only be executed on changes regarding that specific profile.

Instead (or in addition) to these scripts, you can also place as many executable
files as you like in subdirectories called `script_name.d` (e.g. `postswitch.d`).

If a script with the same name occurs multiple times, user configuration
takes precedence over system configuration (as specified by the
[XDG spec](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html))
and profile configuration over general configuration.

As a concrete example, suppose you have the files

- `/etc/xdg/autorandr/postswitch`
- `~/.config/autorandr/postswitch`
- `~/.config/autorandr/postswitch.d/notify-herbstluftwm`
- `~/.config/autorandr/docked/postswitch`

and switch from `mobile` to `docked`. Then
`~/.config/autorandr/docked/postswitch` is executed, since the profile specific
configuration takes precedence, and
`~/.config/autorandr/postswitch.d/notify-herbstluftwm` is executed, since
it has a unique name.

If you switch back from `docked` to `mobile`, `~/.config/autorandr/postswitch`
is executed instead of the `docked` specific `postswitch`.

If you experience issues with xrandr being executed too early after connecting
a new monitor, then you can use a `predetect` script to delay the execution.
Write e.g. `sleep 1` into that file to make autorandr wait a second before
running `xrandr`.

#### Variables

Some of autorandr's state is exposed as environment variables
prefixed with `AUTORANDR_`, such as:
- `AUTORANDR_CURRENT_PROFILE`
- `AUTORANDR_CURRENT_PROFILES`
- `AUTORANDR_PROFILE_FOLDER`
- `AUTORANDR_MONITORS`

with the intention that they can be used within the hook scripts.

For instance, you might display which profile has just been activated by
including the following in a `postswitch` script:
```sh
notify-send -i display "Display profile" "$AUTORANDR_CURRENT_PROFILE"
```

The one kink is that during `preswitch`, `AUTORANDR_CURRENT_PROFILE` is
reporting the *upcoming* profile rather than the *current* one.

### Wildcard EDID matching

The EDID strings in the `~/.config/autorandr/*/setup` files may contain an
asterisk to enable wildcard matching: Such EDIDs are matched against connected
monitors using the usual file name globbing rules. This can be used to create
profiles matching multiple (or any) monitors.

### udev triggers with NVidia cards

In order for `udev` to detect `drm` events from the native NVidia driver, the
kernel parameter `nvidia-drm.modeset` must be set to 1. For example, add a file
`/etc/modprobe.d/nvidia-drm-modeset.conf`:

```
options nvidia_drm modeset=1
```

## Changelog

**autorandr 1.12.1**
* *2021-12-22* Fix `--match-edid` (see #273)

**autorandr 1.12**
* *2021-12-16* Switch default interpreter to Python 3
* *2021-12-16* Add `--list` to list all profiles
* *2021-12-16* Add `--cycle` to cycle all detected profiles
* *2021-12-16* Store display properties (see #204)

**autorandr 1.11**
* *2020-05-23* Handle empty sys.executable
* *2020-06-08* Fix Python 2 compatibility
* *2020-10-06* Set group membership of users in batch mode

**autorandr 1.10.1**
* *2020-05-04* Revert making the launcher the default (fixes #195)

**autorandr 1.10**
* *2020-04-23* Fix hook script execution order to match description from readme
* *2020-04-11* Handle negative gamma values (fixes #188)
* *2020-04-11* Sort approximate matches in detected profiles by quality of match
* *2020-01-31* Handle non-ASCII environment variables (fixes #180)
* *2019-12-31* Fix output positioning if the top-left output is not the first
* *2019-12-31* Accept negative gamma values (and interpret them as 0)
* *2019-12-31* Prefer the X11 launcher over systemd/udev configuration

**autorandr 1.9**

* *2019-11-10* Count closed lids as disconnected outputs
* *2019-10-05* Do not overwrite existing configurations without `--force`
* *2019-08-16* Accept modes that don't match the WWWxHHH pattern
* *2019-03-22* Improve bash autocompletion
* *2019-03-21* Store CRTC values in configurations
* *2019-03-24* Fix handling of recently disconnected outputs (See #128 and #143)

**autorandr 1.8.1**

* *2019-03-18* Removed mandb call from Makefile

**autorandr 1.8**

* *2019-02-17* Add an X11 daemon that runs autorandr when a display connects (by @rliou92, #127)
* *2019-02-17* Replace width=0 check with disconnected to detect disconnected monitors (by @joseph-jones, #139)
* *2019-02-17* Fix handling of empty padding (by @jschwab, #138)
* *2019-02-17* Add a man page (by @somers-all-the-time, #133)

**autorandr 1.7**

* *2018-09-25* Fix FB size computation with rotated screens (by @Janno, #117)

**autorandr 1.6**

* *2018-04-19* Bugfix: Do not load default profile unless --change is set
* *2018-04-30* Added a `AUTORANDR_MONITORS` variable to hooks (by @bricewge, #106)
* *2018-06-29* Fix detection of current configuration if extra monitors are active
* *2018-07-11* Bugfix in the latest change: Correctly handle "off" minitors when comparing
* *2018-07-19* Do not kill spawned user processes from systemd unit
* *2018-07-20* Correctly handle "off" monitors when comparing -- fixup for another bug.

**autorandr 1.5**

* *2018-01-03* Add --version
* *2018-01-04* Fixed vertical/horizontal/clone-largest virtual profiles
* *2018-03-07* Output all non-error messages to stdout instead of stderr
* *2018-03-25* Add --detected and --current to filter the profile list output
* *2018-03-25* Allow wildcard matching in EDIDs

**autorandr 1.4**

* *2017-12-22* Fixed broken virtual profile support
* *2017-12-14* Added support for a settings file
* *2017-12-14* Added a virtual profile `off`, which disables all screens

**autorandr 1.3**

* *2017-11-13* Add a short form for `--load`
* *2017-11-21* Fix environment stealing in `--batch` mode (See #87)

**autorandr 1.2**

* *2017-07-16* Skip `--panning` unless it is required (See #72)
* *2017-10-13* Add `clone-largest` virtual profile

**autorandr 1.1**

* *2017-06-07* Call systemctl with `--no-block` from udev rule (See #61)
* *2017-01-20* New script hook, `predetect`
* *2017-01-18* Accept comments (lines starting with `#`) in config/setup files

**autorandr 1.0**

* *2016-12-07* Tag the current code as version 1.0.0; see github issue #54
* *2016-10-03* Install a desktop file to `/etc/xdg/autostart` by default
