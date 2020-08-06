# Automating Installation

The ability to automate installation of mods is important. However, there are many complex variables that come with automated installs.

Per my draft spec ([PR viewable here](https://github.com/mc-cip/spec/pull/6)), both Versions and Files can have `installation` objects, which describe how projects can be installed. This separation allows automated installs to require multiple files, while retaining the ability to have separate methods per file.

## Installation Methods

Files and versions can contain the `installation` object, which informs how to install the file. It contains the `method` field, which has the following possible values:

- `placeInDirectory` - Place this file in a specified directory, relative to the path of the Minecraft game. Specify the directory in a field named `directory` installed the `installation` object
- `jarMod` - This file is a jar mod, and must be added to minecraft.jar
- `versionJsonInstall` - This version must be installed according to the file with the `versionJson` relation.
- `runForgeInstaller` - This version should be installed using a Forge installer. Launchers MAY ignore this.
- `other` - Unknown installation method

For framework values such as `versionJsonInstall` and `runForgeInstaller`, launchers may ignore them if they do not provide enough information or they do not want to implement them. For example, a launcher may not want to run the Forge installer and instead perform the installation themselves.
