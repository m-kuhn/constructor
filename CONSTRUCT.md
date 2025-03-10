
<!--
DO NOT EDIT THIS FILE MANUALLY
Edit scripts/make_docs.py and/or constructor/construct.py
and regenerate.
-->

# The `construct.yaml` specification

The `construct.yaml` file is the primary mechanism for controlling
the output of the Constructor package. The file contains a list of
key/value pairs in the standard [YAML](https://yaml.org/) format.
Each configuration option is listed in its own subsection below.

Constructor employs the Selector enhancement of the YAML format
first employed in the
[conda-build](https://docs.conda.io/projects/conda-build/en/latest/)
project. Selectors are specially formatted YAML comments that Constructor
uses to customize the specification for different platforms. The precise
syntax for selectors is described in
[this section](https://docs.conda.io/projects/conda-build/en/latest/resources/define-metadata.html#preprocessing-selectors)
of the `conda-build` documentation. The list of selectors available
for use in Constructor specs is given in the section
[Available selectors](#available-selectors) below.

Finally, `construct.yaml` is parsed as a `jinja2` template and so any valid
`jinja2` templating directives may be used. The current shell environment
is available as the `jinja2` variable `environ`. As an example, setting the
`version` key from an environment variable called `VERSION` would look like:
`version: {{ environ["VERSION"] }}`. Note that the special
environment variables available in `meta.yaml` when running `conda-build`
are not available here.

> Note: This content is also available in the CLI as `constructor --help-construct`

## Available keys
### `name`

_required:_ yes<br/>
_type:_ string<br/>

Name of the installer. Names may be composed of letters, numbers,
underscores, dashes, and periods, but may not begin or end with a
dash or period.

### `version`

_required:_ yes<br/>
_type:_ string<br/>

Version of the installer. Versions may be composed of letters, numbers,
underscores, dashes, and periods, but may not begin or end with a
dash or period.

### `channels`

_required:_ no<br/>
_type:_ list<br/>

The conda channels from which packages are retrieved. At least one channel must
be supplied, either in `channels` or `channels_remap`.

See notes in `channels_remap` for details about local channels.

### `channels_remap`

_required:_ no<br/>
_type:_ list<br/>

A list of `src/dest` channel pairs. When building the installer, conda will
use the `src` channels to solve and fetch the packages. However, the resulting
installation will see the packages as coming from the `dest` equivalent.
This allows an installer to be built against a different set of channels than
will be present when the installer is actually used. Example use:

```yaml
channels_remap:
  - src: file:///tmp/a3/conda-bld              # [unix]
    dest: https://repo.anaconda.com/pkgs/main  # [unix]
  - src: file:///D:/tmp/a3/conda-bld           # [win]
    dest: https://repo.anaconda.com/pkgs/main  # [unix]
```

At least one channel must be supplied, either in `channels` or `channels_remap`.

### `specs`

_required:_ no<br/>
_types:_ list, string<br/>

A list of package specifications; e.g. `python 2.7*`, `pyzmq` or `numpy >=1.8`.
The specifications are identical in form and purpose to those that would be
included in a `conda create --file` command. Packages may also be specified
by an exact URL; e.g.,
`https://repo.anaconda.com/pkgs/main/osx-64/openssl-1.0.2o-h26aff7b_0.tar.bz2`.
This key can also take a `str` pointing to a requirements file with the same syntax.

The specs will be solved with the solver configured for your `base` conda installation,
if any. Starting with conda 22.11, this behavior can be overriden with the
`CONDA_SOLVER` environment variable.

### `user_requested_specs`

_required:_ no<br/>
_types:_ list, string<br/>

A list of package specifications to be recorded as "user-requested" for the
initial environment in conda's history file. This information is used by newer
versions of conda to better filter its package choices on subsequent installs;
for example, if `python=3.6` is included, then conda will always seek versions
of packages compatible with Python 3.6. If this is option is not provided, it
will be set equal to the value of `specs`.

### `exclude`

_required:_ no<br/>
_type:_ list<br/>

A list of package names to be excluded after the `specs` have been resolved.
For example, you can say that `readline` should be excluded, even though it
is contained as a result of resolving the specs for `python 2.7`.

### `menu_packages`

_required:_ no<br/>
_type:_ list<br/>

A list of packages with menu items to be installed. The packages must have
necessary metadata in `Menu/<package name>.json`). Menu items are currently
only supported on Windows. By default, all menu items will be installed;
supplying this list allows a subset to be selected instead.

### `ignore_duplicate_files`

_required:_ no<br/>
_type:_ boolean<br/>

By default, constructor will warn you when adding packages with duplicate
files in them. Setting this option to false will raise an error instead.

### `install_in_dependency_order`

_required:_ no<br/>
_types:_ boolean, string<br/>

_Obsolete_. The current version of constructor relies on the standalone
conda executable for its installation behavior. This option is now
ignored with a warning.

### `environment`

_required:_ no<br/>
_type:_ string<br/>

Name of the environment to construct from. If this option is present, the
`specs` argument will be ignored. Using this option allows the user to
curate the enviromment interactively using standard `conda` commands, and
run constructor with full confidence that the exact environment will be
reproduced.

### `environment_file`

_required:_ no<br/>
_type:_ string<br/>

Path to an environment file (TXT or YAML) to construct from. If this option
is present, the `specs` argument will be ignored. Instead, constructor will
call conda to create a temporary environment, constructor will build and
installer from that, and the temporary environment will be removed.
This ensures that constructor is using the precise local conda configuration
to discover and install the packages. The created environment MUST include
`python`.

Read notes about the solver in the `specs` field.

### `transmute_file_type`

_required:_ no<br/>
_type:_ string<br/>

File type extension for the files to be transmuted into. Currently supports
only '.conda'. See conda-package-handling for supported extension names.
If left empty, no transmuting is done.

### `conda_default_channels`

_required:_ no<br/>
_type:_ list<br/>

If this value is provided as well as `write_condarc`, then the channels
in this list will be included as the value of the `default_channels:`
option in the environment's `.condarc` file. This will have an impact
only if `conda` is included in the environmnent.

### `conda_channel_alias`

_required:_ no<br/>
_type:_ string<br/>

The channel alias that would be assumed for the created installer
(only useful if it includes conda).

### `extra_envs`

_required:_ no<br/>
_type:_ dictionary<br/>

Create more environments in addition to the default `base` provided by `specs`,
`environment` or `environment_file`. This should be a map of `str` (environment
name) to a dictionary of options:
- `specs` (list of str): which packages to install in that environment
- `environment` (str): same as global option, for this env
- `environment_file` (str): same as global option, for this env
- `channels` (list of str): using these channels; if not provided, the global
  value is used. To override inheritance, set it to an empty list.
- `channels_remap` (list of str): same as global option, for this env;
  if not provided, the global value is used. To override inheritance, set it to
  an empty list.
- `user_requested_specs` (list of str): same as the global option, but for this env;
  if not provided, global value is _not_ used

Notes:
- `ignore_duplicate_files` will always be considered `True` if `extra_envs` is in use.
- `conda` needs to be present in the `base` environment (via `specs`)
- support for `menu_packages` is planned, but not possible right now. For now, all packages
  in an `extra_envs` config will be allowed to create their shortcuts.
- If a global `exclude` option is used, it will have an effect on the environments created
  by `extra_envs` too. For example, if the global environment excludes `tk`, none of the
  extra environments will have it either. Unlike the global option, an error will not be
  thrown if the excluded package is not found in the packages required by the extra environment.
  To override the global `exclude` value, use an empty list `[]`.

### `installer_filename`

_required:_ no<br/>
_type:_ string<br/>

The filename of the installer being created. If not supplied, a reasonable
default will determined by the `name`, `version`, platform, and installer type.

### `installer_type`

_required:_ no<br/>
_types:_ string, list<br/>

The type of the installer being created.  Possible values are:
- `sh`: shell-based installer for Linux or macOS;
- `pkg`: macOS GUI installer built with Apple's `pkgbuild`
- `exe`: Windows GUI installer built with NSIS

The default type is `sh` on Linux and macOS, and `exe` on Windows. A special
value of `all` builds _both_ `sh` and `pkg` installers on macOS, as well
as `sh` on Linux and `exe` on Windows.

Notes for silent mode `/S` on Windows EXEs:
- NSIS Silent mode will not print any error message, but will silently abort the installation.
  If needed, [NSIS log-builds][nsis-log] can be used to print to `%PREFIX%\install.log`, which
  can be searched for `::error::` strings. Pre- and post- install scripts will only throw an error
  if the environment variable `NSIS_SCRIPTS_RAISE_ERRORS` is set.
- The `/D` flag can be used to specify the target location. It must be the last argument in
  the command and should NEVER be quoted, even if it contains spaces. For example:
  `CMD.EXE /C START /WAIT myproject.exe /S /D=C:\path with spaces\my project`.

[nsis-log]: https://nsis.sourceforge.io/Special_Builds

### `license_file`

_required:_ no<br/>
_type:_ string<br/>

Path to the license file being displayed by the installer during the install
process. It must be plain text (.txt) for shell-based installers. On PKG,
.txt, .rtf and .html are supported. On Windows, .txt and .rtf are supported.

### `keep_pkgs`

_required:_ no<br/>
_type:_ boolean<br/>

If `False` (default), the package cache in the `pkgs` subdirectory is removed
when the installation process is complete. If `True`, this subdirectory and
its contents are preserved. If `keep_pkgs` is `False`, Unix `.sh` and Windows `.msi`
installers offer a command-line option (`-k` and `/KeepPkgCache`, respectively)
to preserve the package cache.

### `batch_mode`

_required:_ no<br/>
_type:_ boolean<br/>

Only affects ``.sh`` installers. If ``False`` (default), the installer launches
an interactive wizard guiding the user through the available options. If
``True``, the installer runs automatically as if ``-b`` was passed.

### `signing_identity_name`

_required:_ no<br/>
_type:_ string<br/>

By default, the MacOS pkg installer isn't signed. If an identity name is specified
using this option, it will be used to sign the installer with Apple's `productsign`.
Note that you will need to have a certificate (usually an "Installer certificate")
and the corresponding private key, together called an 'identity', in one of your
accessible keychains. Common values for this option follow this format
`Developer ID Installer: Name of the owner (XXXXXX)`.

### `notarization_identity_name`

_required:_ no<br/>
_type:_ string<br/>

If the pkg installer is going to be signed with `signing_identity_name`, you
can also prepare the bundle for notarization. This will use Apple's `codesign`
to sign `conda.exe`. For this, you need an "Application certificate" (different from the
"Installer certificate" mentioned above). Common values for this option follow the format
`Developer ID Application: Name of the owner (XXXXXX)`.

### `signing_certificate`

_required:_ no<br/>
_type:_ string<br/>

On Windows only, set this key to the path of a PFX certificate to be used with `signtool`.
Additional environment variables can be used to configure this step, namely:

- `CONSTRUCTOR_PFX_CERTIFICATE_PASSWORD` (password to unlock the certificate, if needed)
- `CONSTRUCTOR_SIGNTOOL_PATH` (absolute path to `signtool.exe`, in case is not in `PATH`)
- `CONSTRUCTOR_SIGNTOOL_TIMESTAMP_SERVER_URL` (custom RFC 3161 timestamping server, default is
http://timestamp.sectigo.com)

### `attempt_hardlinks`

_required:_ no<br/>
_types:_ boolean, string<br/>

_Obsolete_. The current version of constructor relies on the standalone
conda executable for its installation behavior. This option is now
ignored with a warning.

### `write_condarc`

_required:_ no<br/>
_type:_ boolean<br/>

By default, no `.condarc` file is written. If set, a `.condarc` file is written to
the base environment if there are any channels or conda_default_channels is set.

### `condarc`

_required:_ no<br/>
_types:_ dictionary, string<br/>

If set, a `.condarc` file is written to the base environment containing the contents
of this value. The value can either be a string (likely a multi-line string) or
a dictionary, which will be converted to a YAML string for writing. _Note:_ if this
option is used, then all other options related to the construction of a `.condarc`
file (`write_condarc`, `conda_default_channels`, etc.) are ignored.

### `company`

_required:_ no<br/>
_type:_ string<br/>

Name of the company/entity who is responsible for the installer.

### `reverse_domain_identifier`

_required:_ no<br/>
_type:_ string<br/>

Unique identifier for this package, formatted with reverse domain notation. This is
used internally in the PKG installers to handle future updates and others. If not
provided, it will default to `io.continuum`. (MacOS only)

### `uninstall_name`

_required:_ no<br/>
_type:_ string<br/>

Application name in the Windows "Programs and Features" control panel.
Defaults to `${NAME} ${VERSION} (Python ${PYVERSION} ${ARCH})`.

### `pre_install`

_required:_ no<br/>
_type:_ string<br/>

Path to a pre-install script, run after the package cache has been set, but
before the files are linked to their final locations. As a result, you should
only rely on tools known to be available on most systems (e.g. `bash`, `cmd`,
etc). See `post_install` for information about available environment variables.

### `pre_install_desc`

_required:_ no<br/>
_type:_ string<br/>

A description of the purpose of the supplied `pre_install` script. If this
string is supplied and non-empty, then the Windows and macOS GUI installers
will display it along with checkbox to enable or disable the execution of the
script. If this string is not supplied, it is assumed that the script
is compulsory and the option to disable it will not be offered.

This option has no effect on `SH` installers.

### `post_install`

_required:_ no<br/>
_type:_ string<br/>

Path to a post-install script. Some notes:

- For Unix `.sh` installers, the shebang line is respected if present;
  otherwise, the script is run by the POSIX shell `sh`. Note that the use
  of a shebang can reduce the portability of the installer. The
  installation path is available as `${PREFIX}`. Installer metadata is
  available in the `${INSTALLER_NAME}`, `${INSTALLER_VER}`, `${INSTALLER_PLAT}`
  environment variables. `${INSTALLER_TYPE}` is set to `SH`.
- For PKG installers, the shebang line is respected if present;
  otherwise, `bash` is used. The same variables mentioned for `sh`
  installers are available here. `${INSTALLER_TYPE}` is set to `PKG`.
- For Windows `.exe` installers, the script must be a `.bat` file.
  Installation path is available as `%PREFIX%`. Metadata about
  the installer can be found in the `%INSTALLER_NAME%`, `%INSTALLER_VER%`,
  `%INSTALLER_PLAT%` environment variables. `%INSTALLER_TYPE%` is set to `EXE`.

If necessary, you can activate the installed `base` environment like this:

- Unix: `source "$PREFIX/etc/profile.d/conda.sh" && conda activate "$PREFIX"`
- Windows: `call "%PREFIX%\Scripts\activate.bat"`

### `post_install_desc`

_required:_ no<br/>
_type:_ string<br/>

A description of the purpose of the supplied `post_install` script. If this
string is supplied and non-empty, then the Windows and macOS GUI installers
will display it along with checkbox to enable or disable the execution of the
script. If this string is not supplied, it is assumed that the script
is compulsory and the option to disable it will not be offered.

This option has no effect on `SH` installers.

### `pre_uninstall`

_required:_ no<br/>
_type:_ string<br/>

Path to a pre uninstall script. This is only supported for on Windows,
and must be a `.bat` file. Installation path is available as `%PREFIX%`.
Metadata about the installer can be found in the `%INSTALLER_NAME%`,
`%INSTALLER_VER%`, `%INSTALLER_PLAT%` environment variables.
`%INSTALLER_TYPE%` is set to `EXE`.

### `default_prefix`

_required:_ no<br/>
_type:_ string<br/>

Set default install prefix. On Linux, if not provided, the default prefix
is `${HOME}/${NAME}` (or, if `HOME` is not set, `/opt/${NAME}`). On Windows,
this is used only for "Just Me" installation; for "All Users" installation,
use the `default_prefix_all_users` key. If not provided, the default prefix
is `${USERPROFILE}\${NAME}`.

### `default_prefix_domain_user`

_required:_ no<br/>
_type:_ string<br/>

Set default installation prefix for domain user. If not provided, the
installation prefix for domain user will be `${LOCALAPPDATA}\${NAME}`.
By default, it is different from the `default_prefix` value to avoid installing
the distribution in the roaming profile. Windows only.

### `default_prefix_all_users`

_required:_ no<br/>
_type:_ string<br/>

Set default installation prefix for All Users installation. If not provided,
the installation prefix for all users installation will be
`${ALLUSERSPROFILE}\${NAME}`. Windows only.

### `default_location_pkg`

_required:_ no<br/>
_type:_ string<br/>

Default installation subdirectory in the chosen volume. In PKG installers,
default installation locations are configured differently. The user can choose
between a "Just me" installation (which would result in `~/<PKG_NAME>`) or another
volume (which defaults to `<VOLUME>/<PKG_NAME>`). If you want a different default,
you can add a middle component with this option, let's call it `location`. It would
result in these default values: `~/<LOCATION>/<PKG_NAME>` for "Just me",
`<VOLUME>/<LOCATION>/<PKG_NAME>` for custom volumes. For example, setting this option
to `/Library` in a "Just me" installation will give you `~/Library/<PKG_NAME>`.
Internally, this is passed to `pkgbuild --install-location`.
macOS only.

### `pkg_name`

_required:_ no<br/>
_type:_ string<br/>

Internal identifier for the installer. This is used in the build prefix and will
determine part of the default location path. Combine with `default_location_pkg`
for more flexibility. If not provided, the value of `name` will be used.  (MacOS only)

### `install_path_exists_error_text`

_required:_ no<br/>
_type:_ string<br/>

Error message that will be shown if the installation path already exists.
You cannot use double quotes or newlines. The placeholder `{CHOSEN_PATH}` is
available and set to the destination causing the error. Defaults to:

> '{CHOSEN_PATH}' already exists. Please, relaunch the installer and
> choose another location in the Destination Select step.

(MacOS only)

### `progress_notifications`

_required:_ no<br/>
_type:_ boolean<br/>

Whether to show UI notifications on PKG installers. On large installations,
the progress bar reaches ~90% very quickly and stays there for a long time.
This might look like the installer froze. This option enables UI notifications
so the user receives updates after each command executed by the installer.
(macOS only)

### `welcome_image`

_required:_ no<br/>
_type:_ string<br/>

Path to an image in any common image format (`.png`, `.jpg`, `.tif`, etc.)
to be used as the welcome image for the Windows and PKG installers.
The image is re-sized to 164 x 314 pixels on Windows and 1227 x 600 on Macos.
By default, an image is automatically generated on Windows. On MacOS, Anaconda's
logo is shown if this key is not provided. If you don't want a background on
PKG installers, set this key to `""` (empty string).

### `header_image`

_required:_ no<br/>
_type:_ string<br/>

Like `welcome_image` for Windows, re-sized to 150 x 57 pixels.

### `icon_image`

_required:_ no<br/>
_type:_ string<br/>

Like `welcome_image` for Windows, re-sized to 256 x 256 pixels.

### `default_image_color`

_required:_ no<br/>
_type:_ string<br/>

The color of the default images (when not providing explicit image files)
used on Windows.  Possible values are `red`, `green`, `blue`, `yellow`.
The default is `blue`.

### `welcome_image_text`

_required:_ no<br/>
_type:_ string<br/>

If `welcome_image` is not provided, use this text when generating the image
(Windows and PKG only). Defaults to `name` on Windows.

### `header_image_text`

_required:_ no<br/>
_type:_ string<br/>

If `header_image` is not provided, use this text when generating the image
(Windows only). Defaults to `name`.

### `initialize_conda`

_required:_ no<br/>
_type:_ boolean<br/>

Add an option to the installer so the user can choose whether to run `conda init`
after the install. See also `initialize_by_default`.

### `initialize_by_default`

_required:_ no<br/>
_type:_ boolean<br/>

Whether to add the installation to the PATH environment variable. The default
is true for GUI installers (msi, pkg) and False for shell installers. The user
is able to change the default during interactive installation. NOTE: For Windows,
`AddToPath` is disabled when `InstallationType=AllUsers`.

### `register_python`

_required:_ no<br/>
_type:_ boolean<br/>

Whether to offer the user an option to register the installed Python instance as the
system's default Python. (Windows only)

### `register_python_default`

_required:_ no<br/>
_type:_ boolean<br/>

Default choice for whether to register the installed Python instance as the
system's default Python. The user is still able to change this during
interactive installation. (Windows only).

### `check_path_length`

_required:_ no<br/>
_type:_ boolean<br/>

Check the length of the path where the distribution is installed to ensure nodejs
can be installed.  Raise a message to request shorter path (less than 46 character)
or enable long path on windows > 10 (require admin right). Default is True. (Windows only).

Read notes about the particularities of Windows silent mode `/S` in the
`installer_type` documentation.

### `check_path_spaces`

_required:_ no<br/>
_type:_ boolean<br/>

Check if the path where the distribution is installed contains spaces. Default is True.
To allow installations with spaces, change to False. Note that:

- A recent conda-standalone (>=22.11.1) or equivalent is needed for full support.
- `conda` cannot be present in the `base` environment

Read notes about the particularities of Windows silent mode `/S` in the
`installer_type` documentation.

### `nsis_template`

_required:_ no<br/>
_type:_ string<br/>

If `nsis_template` is not provided, constructor uses its default
NSIS template. For more complete customization for the installation experience,
provide an NSIS template file. (Windows only).

### `welcome_file`

_required:_ no<br/>
_type:_ string<br/>

If `installer_type` is `pkg` on MacOS, this message will be
shown before the license information, right after the introduction.
File can be plain text (.txt), rich text (.rtf) or HTML (.html). If
both `welcome_file` and `welcome_text` are provided, `welcome_file` takes precedence.
(MacOS only).

If the installer is for windows and welcome file type is nsi,
it will use the nsi script to add in extra pages before the installer
begins the installation process.

### `welcome_text`

_required:_ no<br/>
_type:_ string<br/>

If `installer_type` is `pkg` on MacOS, this message will be
shown before the license information, right after the introduction.
If this key is missing, it defaults to a message about Anaconda Cloud.
You can disable it altogether so it defaults to the system message
if you set this key to `""` (empty string).
(MacOS only).

### `readme_file`

_required:_ no<br/>
_type:_ string<br/>

If `installer_type` is `pkg` on MacOS, this message will be
shown before the license information, right after the welcome screen.
File can be plain text (.txt), rich text (.rtf) or HTML (.html). If
both `readme_file` and `readme_text` are provided, `readme_file` takes precedence.
(MacOS only).

### `readme_text`

_required:_ no<br/>
_type:_ string<br/>

If `installer_type` is `pkg` on MacOS, this message will be
shown before the license information, right after the welcome screen.
If this key is missing, it defaults to a message about Anaconda Cloud.
You can disable it altogether if you set this key to `""` (empty string).
(MacOS only).

### `conclusion_file`

_required:_ no<br/>
_type:_ string<br/>

If `installer_type` is `pkg` on MacOS, this message will be
shown at the end of the installer upon success. File can be
plain text (.txt), rich text (.rtf) or HTML (.html). If both
`conclusion_file` and `conclusion_text` are provided,
`conclusion_file` takes precedence. (MacOS only).

If the installer is for windows and conclusion file type is nsi,
it will use the nsi script to add in extra pages and the conclusion file
at the end of the installer.

### `conclusion_text`

_required:_ no<br/>
_type:_ string<br/>

A message that will be shown at the end of the installer upon success.
The behaviour is slightly different across installer types:
- PKG: If this key is missing, it defaults to a message about Anaconda Cloud.
  You can disable it altogether so it defaults to the system message if you set this
  key to `""` (empty string).
- EXE: The first line will be used as a title. The following lines will be used as text.
(macOS PKG and Windows only).

### `extra_files`

_required:_ no<br/>
_type:_ list<br/>

Extra, non-packaged files that should be added to the installer. If provided as relative
paths, they will be considered relative to the directory where `construct.yaml` is.
This setting can be passed as a list of:
- `str`: each found file will be copied to the root prefix
- `Mapping[str, str]`: map of path in disk to path in prefix.

### `temp_extra_files`

_required:_ no<br/>
_type:_ list<br/>

Temporary files that could be referenced in the installation process (i.e. customized
 `welcome_file` and `conclusion_file` (see above)) . Should be a list of
file paths, relative to the directory where `construct.yaml` is. In Windows, these
files will be copied into a temporary folder, the NSIS `$PLUGINSDIR`, during
install process (Windows only).

Supports the same values as `extra_files`.

### `build_outputs`

_required:_ no<br/>
_type:_ list<br/>

Additional artifacts to be produced after building the installer.
It expects either a list of strings or single-key dictionaries:
Allowed keys are:
- `info.json`: The internal `info` object, serialized to JSON. Takes no options.
- `pkgs_list`: The list of packages contained in a given environment. Options:
    - `env` (optional, default=`base`): Name of an environment in `extra_envs` to export.
- `licenses`: Generate a JSON file with the licensing details of all included packages. Options:
    - `include_text` (optional bool, default=`False`): Whether to dump the license text in the JSON.
      If false, only the path will be included.
    - `text_errors` (optional str, default=`None`): How to handle decoding errors when reading the
      license text. Only relevant if include_text is True. Any str accepted by open()'s 'errors'
      argument is valid. See https://docs.python.org/3/library/functions.html#open.


## Available selectors
- `aarch64`
- `arm64`
- `armv7l`
- `linux`
- `linux32`
- `linux64`
- `osx`
- `ppc64le`
- `s390x`
- `unix`
- `win`
- `win32`
- `win64`
- `x86`
- `x86_64`

## Available Platforms
Specify which platform (`CONDA_SUBDIR`) to build for via the `--platform` argument. If provided, this argument must be formated as `<platform>-<architecture>`, e.g.:
- `linux-64`
- `linux-aarch64`
- `linux-ppc64le`
- `linux-s390x`
- `win-64`
- `osx-64`
- `osx-arm64`