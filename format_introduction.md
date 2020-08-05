# Introduction to the MCIP Format

The MCIP Format is an open format for Minecraft Projects. A Minecraft Project is a Mod, Resource/Texture Pack, World, or Modpack.
The Format stores metadata about Projects such as their description, authors, download links, and more.

## Goals

- Create an open standard that can be used by Hosts like Diluv and Modrinth.
- Encourage open discussion and debate about the properties and metadata stored in the format.
- Make Minecraft Project distribution open and allow easy interoperability between Hosts.

---

## Users of the Format

The MCIP format is designed to be used by anyone who works with Minecraft assets. Examples include:

- Hosts, such as Diluv, Modrinth, and CurseForge
- Launchers, such as MultiMC and GDLauncher
- Mod Developers, who can provide metadata about their mods

---

## Format Documentation

Documentation on the Format specification is provided in **format_spec.md**.

---

## Implementing the Format

- Hosts that provide an API are encouraged to allow API users to request data be returned in the MCIP Format.
- Launcher developers are encouraged to allow browsing and installing Projects that are served by Hosts providing MCIP Format-compliant data.

---

## MCIP Standardized API

The MCIP Standardized API is a work-in-progress API that allows hosts to synchronize their databases between each other. It also makes implementing new hosts easier for launcher developers, and it allows launchers to download content from hosts that have not been directly implemented into the launcher.

The MCIP Standardized API is currently incomplete. [You can view a Pull Request here](https://github.com/mc-cip/spec/pull/3).

The API is NOT required in order for hosts to use the MCIP format, however it is HIGHLY ENCOURAGED to implement it.

---

### MCIP Standardized API for Launcher Developers

Launchers will have to read the `src` field of dependencies in order to see where to look for a dependency. This field MAY be a URL that links to a MCIP Standardized API-compliant service, however it also MAY be a String representing the name of a host.

If the value of the field is a URL, then launchers should use the MCIP Standardized API, with the value of the field as the base URL.

If the value of the field is a name of a host, then launchers will have to use special implementations. Launchers will need to add special cases for different hosts. This is why it's recommended for hosts to utilize the MCIP Standardized API.

If a launcher encounters a host that it does not know of, or a host set to `unknown`, it should look for the same ID in the hosts that it does know of. If it is unable to find the Project, an error should be shown to the user, explaining that it was unable to find the Project with the specified host.

If a launcher encounters a host with the value `manual`, then launchers SHOULD NOT look for a Project with the same ID in known hosts. Instead, the launcher should tell the user that it manually needs to find a Project with the specified ID.

---

### Implementing Frameworks for Launchers

Frameworks also exist as MCIP Format Projects. It's important for launchers to understand the Project metadata and install frameworks as needed.

Framework IDs MUST be prefixed with `framework-`, so launchers can tell if a project is a Framework.
For frameworks with the installation method set to `versionJsonInstall`, launchers should read the version JSON metadata specified in a file with `rel` set to `versionJson`. An example of a modloader using `versionJsonInstall` would be Fabric.

Launchers may deviate from or completely ignore the value set in the installation method for the version.

The `runForgeInstaller` installation method exists to tell launchers that this contains a Forge installer. Launchers may choose to use their own method of installation if they would like.
