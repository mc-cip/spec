# Automating Installation

The ability to automate installation of mods is important. However, there are many complex variables that come with automated installs.

Per my draft spec ([PR viewable here](https://github.com/mc-cip/spec/pull/6)), both Versions and Files can have `installation` objects, which describe how projects can be installed. This separation allows automated installs to require multiple files, while retaining the ability to have separate methods per file.

## A notice about launchers

It's important to remember that launchers may choose to ignore provided information or deviate from the spec. This is true of all formats and standards, but it's especially true for something as complex as installation. If you're providing installation information in the MCIP format, it's important to keep in mind that not all launchers will use data the same way.

## Storing Installation Information

Installation information is stored in the `installation` object, present in either a Version Object or File Object. For more information on the Version and File Objects, please view the [Spec Draft PR](https://github.com/mc-cip/spec/pull/6).

The `installation` object has a very basic structure. By default, it only requires one field, `method`. This describes the method that launchers and other automation tools should use in order to install and properly set up this project. Any extra information is stored in optional fields also stored inside the `installation` object. These fields are specific to the `method` specified.

## Installation Methods

The following is a comprehensive summary of every available installation method in the MCIP format. Information on how launchers should implement handling of these methods is described in **implementing_installation.md**.

##### Version Notice

If a method is used in a Version Object instead of a File Object, the default implementation is to apply the installation behavior to allow of the files marked `"rel": "primary"`. If this differs for a method, it will be noted.

### `placeInDirectory`

Place the file in the directory specified in the extra `directory` field.

The `directory` field is subject to the rules defined in **Specifying Paths**.

### `jarMod`

This is a jar mod, and the contents of it must be added to the Minecraft jar file. See the Implementing Installation document. The file specified MUST be in the .zip format.
`TODO: should other formats be supported?`

### `versionJsonInstall`

This should be installed according to the file with `"rel": "versionJson"`. This should be only used for Projects which NEED a version JSON file, like modloaders.

This MUST be used for Versions that contain a file with `"rel": "versionJson"`. If a file with the `versionJson` relation is not present, this method is invalid. It is RECOMMENDED to place this method inside the Version Object instead of a File Object.

## `classpathLibrary`

This project is a library that must be present on the Minecraft classpath.

This has an optional `versionJson` field. The field is an object and it contains the information that would be stored about the project if it were specified in a version JSON file, such as `name` and `url`.

Launchers should use the value of the `versionJson` field to correlate libraries defined in a version JSON file and a library in MCIP format.

### `runInstaller`

The file with an installer relation value should be executed. Note that this field is very vague and it's recommended to use something with more information such as `versionJsonInstall`.

### `other`

An unknown installation method.

`TODO: what other methods should be added? what about classpath dependencies that are not specified in version.json files?`

---

## Specifying Paths

Some fields, such as `placeInDirectory`'s `directory` field, requires a path to be supplied. This path by default is relative to the root of the Minecraft Instance that this project is being installed to. If the Project is not able to be installed to a Minecraft Instance for whatever reason, it is up to launchers to supply a default relative path.

Paths MUST use forward slashes to indicate nesting. Backslashes (even on Windows systems) CANNOT be used.

Paths MAY start with `./`, however it is not needed. For example, `./mods` is a valid directory path, however `mods` also works and does not have the unnecessary `./`.

Absolute paths MAY be specified by prefixing the path with an absolute path prefix. This prefix has two valid options:

- `/` character for Unix-like systems
- Windows device prefix path (e.g. `C:/`)

Absolute prefixes are HIGHLY DISCOURAGED and should only be used when absolutely necessary. For example, a mod that installs itself to the `mods` folder SHOULD NOT use an absolute path, and instead should specify `mods`.

The following are rules on paths and filenames:

- Paths that contain Windows restricted filenames and characters are not allowed
  - Restricted characters are `<`, `>`, `:`, `"`, `|`, `?`, `*`
- Empty strings are not allowed
  - If you need to specify the current relative path, without using a subdirectory, use `.`
- Filenames CANNOT contain a slash or backslash
- Filenames that end in `.` CANNOT be used
- Filenames that end in a space CANNOT be used
- New line and carriage return characters CANNOT be used
- Most Unicode characters are allowed, but using normal ASCII characters (A-Z, a-z, 0-9) is recommended

  - Restricted Unicode characters include NUL and CR

`TODO: more information on restricted characters`
