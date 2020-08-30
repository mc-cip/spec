# Implementing Installation Information

This document exists as a guide for launchers and other automation tools that may work with installation information. This is not a comprehensive manual and it may not fit for every launcher. However, it's recommended to follow this guide as accurately as possible in order to have a smooth experience.

## Implementing `placeInDirectory`

The `directory` field in this value should be relative to Minecraft installation that the project is being installed to. This field is subject to path rules defined in **installation.md - Specifying Paths**. Launchers should check for rule violations in any field where a directory is passed in order to ensure format-compliant information.

Files should be placed with the same filename and file extension as when downloaded.

## Implementing `versionJsonInstall`

Launchers will likely already have the mechanisms in place to parse Minecraft version JSON files. To install a Version with the `versionJsonInstall` method, first find a file with the `rel` value set to `versionJson`. If a file is not present, an error should be presented to the user. If multiple files are found, the first one should be used.

Launchers can then use the file with the `versionJson` relation and automatically install libraries from URLs specified in the JSON file.

## Implementing `classpathLibrary`

This is a library that needs to be on the Minecraft classpath. Launchers can choose to place the file marked with `primary` anywhere. An optional `versionJson` field describes how the classpath would be specified in a version JSON file.

## Implementing `runInstaller`

This is a very vague method and there's no correct way to implement it. Launchers can run the file marked with the platform's current installer, `installer-(win, macos, or linux)`, or `generic-installer` if a platform-native one is not specified.

Implementation of `runInstaller` mostly lies on the launcher. Launchers may add special exemptions for certain modloaders, for example.

## Implementing `instanceInstall`

Launchers should follow these steps in order to correctly install these projects:

- Create a new directory specifically for this instance.
- Install the required dependencies to the directory according to their installation info
- If a file with `"rel": "compressedOverrides"` exists, download it and extract the contents to the instance directory

## Implementing `jarMod`

The File marked with `primary` should be downloaded to a temporary location. The file should then be extracted. The contents of this file should be copied over to the Minecraft Game JAR (also extracted). Launchers may choose to automatically delete META-INF.

## Implementing `other`

If an `other` installation method is found, an error should be presented to the user.
