# MCIP Format Specification

This document provides detailed information on the MCIP Format.

The MCIP format uses JSON to store information. If providing an API that returns format-compliant information, the `application/json` Content-Type header SHOULD be provided. If stored on disk, the `.json` extension SHOULD be used.

Metadata MAY be modularized in order to save bandwidth or storage space. Information on what fields may be modularized and the proper method of doing it will be at the end of this document.

An format-compliant example is provided in `format_example.json`

## Fields

Fields are stored inside the JSON metadata of a Project. Fields that are optional will be marked as such. All fields are stored as a String value unless stated otherwise. Below is a complete list of all fields available for the MCIP Format, grouped into categories.

## General Project Information

### `schemaVersion`

This field MUST be stored as an integer. It contains the version of the Format being used, to help with backwards compatibility.

---

### `id`

The ID of the Project. There is no standard or required format on how to store an ID, other than it CANNOT include spaces. It may be a slug, UUID, random number, or other method of storing IDs. It is RECOMMENDED that IDs are not tied to the name of a project.

---

### `name`

The name of a Project. This name MUST be a human readable name and SHOULD not contain unnecessary information. For instance, the name field should not specify where the project is being hosted, or the authors of a project.

---

### `summary` (optional)

A short, one sentence summary of the Project. Any form of rich text such as HTML SHOULD NOT be used.

---

## Detailed Project Information

### `description` (optional)

A long description of the project. HTML MAY be used. It is not recommended to use Markdown as there may be rendering inconsistencies. Any form of code execution such as JavaScript or WebAssembly SHOULD NOT be used. HTML forms, buttons, or other form of interactive elements SHOULD NOT be used. iframes to external domains MAY be used, but it is recommended to embed iframes only with a trustworthy website.

---

### `media` (optional)

This field MUST be stored as an array. This contains Media Item objects, which have the following fields.

##### `rel` (optional)

The relation of this file. Currently, the only valid option is `icon`. If the Media Item is not the Project's icon, this field SHOULD NOT be present.

##### `type`

The type of this Media Item. Valid options are `video` and `image`.

##### `embed`

The `embed` field is valid for both `video` and `image`. It contains a URI to a webpage which SHOULD be embedded when displaying this Media Item. It is recommended for hosts to only allow trustworthy domains to be embedded.

##### `src`

The `src` field is valid for both `video` and `image`. It contains a direct URI to either a playable video file or an image file. It is RECOMMENDED for images to be in the WebP or PNG formats, and it is RECOMMENDED for videos to be in the WebM format. Recommended video codecs include VP9 and AV1. Base64 encoded images, which are typically prefixed with `data:base64

`embed` and `src` are conflicting fields and both fields CANNOT be present in the same Gallery Item.

##### `caption` (optional)

An optional string which can be displayed as a caption for the current Gallery Item. Any form of rich text such as HTML CANNOT be used. `caption` CANNOT be present when `rel` is set to `icon`.

##### `sha256` (optional)

An optional SHA256 checksum for the media file. This field CANNOT be present if the `embed` field is present.

### `authors` (optional)

This field MUST be stored as an array. It contains objects storing the information about each author. Those objects MUST be EITHER a Team or an individual Author.

**Team objects consist of the following fields:**

##### `team`

Teams MUST have a `team` field that is set to true. A `team` field that is not set to `true` is invalid.

##### `id`

A unique identifier to this team, allowing you to search by Team. There is no required format for how this ID should be generated, however it MUST be unique to the Team

##### `name`

The human readable name of the team. Spaces are allowed.

##### `uri` (optional)

An optional field that can link to a webpage relating to the Team.

##### `members`

An array containing Author objects, who are the members of the team.

**Author objects consist of the following fields:**

##### `id`

A unique identifier to this author. The same requirements for Team IDs should be followed here. This ID MUST be unique to the author.

##### `name`

The human readable name of the author. Spaces are allowed.

##### `uri` (optional)

An optional field that specifies a website relating to the author. It may be the author's personal website, the author's profile on a certain host, or any valid URL.

---

### `categories` (optional)

This field MUST be stored as an array. It contains Category Objects. Those objects MUST be made up of the following fields:

##### `name`

The human readable name of the category. Examples include "Magic", "Technology", and "World Gen". It is NOT RECOMMENDED to use the name of a modloader as a category.

##### `uri` (optional)

An optional link to the category on a host, where users could see Projects within that category.

##### `icon` (optional)

This is an object that contains the following fields:

##### `sha256` (optional)

An optional SHA256 checksum of the icon specified in `uri`.

##### `uri`

The URI of this icon. This field is required to be present if the `icon` object is present.

---

### `license` (optional)

This is an object that contains information about the license that a project is under. It contains the following fields:

##### `spdx`

The SPDX identifier of the license. [You can find a list of SPDX license names and identifiers here](https://spdx.org/licenses/).

##### `uri`

A URL linking to the full license.

##### `modpackUsage` (optional)

A field describing usage of the Project in modpacks. Valid options:

- `alwaysAllow` - This Project can be used in a modpack without asking for permission.
- `requiresPermission` - Before included in a modpack, permission must be granted from the author.
- `neverAllow` - This Project can never be used in a modpack.

---

### `links` (optional)

This field MUST be stored as an array. It contains a collection of links relating to the Project. Each Link Object stored inside this arrow MUST contain the following fields:

##### `rel`

This field MUST be stored as an array OR a String. If stored as an array, it MUST contain Strings. This field contains information about what the link is. Valid standardized values are:

- `homepage` - The official homepage of a Project
- `documentation` - Documentation of a Project.
- `wiki` - Wiki information about a Project.
- `scm` - The source code of a Project, for example a GitHub URL.
- `issues` - An issue tracker where bugs and feature requests can be filed.
- `community` - A forum or other online discussion platform where users can discuss the Project.

Other values MAY be used, such as `bitbucket` or `github`. However, it is RECOMMENDED to use generic terminology instead of specific names. If you are unsure whether a value COULD become standardized, please prefix it with `x-`.

##### `name`

A human-readable name of the link. For example, "Source Code" or "Project Website".

##### `uri`

The link, to be viewed in a web browser.

---

### `successor` and `predecessor` (optional)

If an older or newer version of a Project exists, but has a separate ID, these fields may be used. These fields MUST be either an array or String representing the respective projects' IDs. If the field is an array, the Projects should be listed in chronological order from when they were first released.

```
"successor": [
  "first-successor",
  "second-successor
]
```

In the example above, the project `first-successor` was released first. After the release of `first-successor`, the project `second-successor` was released.

---

## Version Information

Projects can contain a `versions` array, which lists Version Objects.

## Version Objects

Version Objects contain fields relating to a certain version of a Project. Below are all the fields that a Version Object may contain.

### `id`

The ID of this version. There is no required format for how to format a version ID, other than that they CANNOT include spaces.

---

### `name`

The human-readable name of this version. It is RECOMMENDED to follow Semantic Versioning, but it is not required. Spaces MAY be used in this name.

---

### `semver` (optional)

An optional Semantic Versioning compatible version number. This field SHOULD ONLY be present if the version's name conforms to Semantic Versioning. This field SHOULD NOT be present if the name does not conform to SemVer.

---

### `releaseDate`

The release date of this version. It MUST be stored as an ISO-8601 conforming String.

---

### `changelog`

The changelog of this version. This MAY be moved into a separate API call for hosts. The changelog MAY contain HTML.

---

### `minecraftVersions`

This field MUST be an array listing compatible Minecraft versions for this Project version.

---

### `dependencies` (optional)

This field MUST be an array containing information about dependencies that may or may not be required for this project. Each object in this array contains the following fields:

##### `type`

The type of dependency this is. Valid options are:

- `asset` - For example, a required library mod
- `framework` - For example, a required modloader

Frameworks exist as a separate type because they may require a more advanced installation process.

##### `src`

This field contains where this dependency can be downloaded. It contains the name of the host, in an all-lowercase string. Examples include: `curseforge`, `diluv` and `modrinth`. If the `type` field is set to `framework`, this field CANNOT be present.

##### `id`

The ID of the required dependency. If the dependency is a required asset and not a framework, then the ID of the asset stored at the host specified in the `src` field should be used. If the dependency is a required framework, then a framework from the list of standardized framework IDs should be used:

- `fabric-loader`
- `forge`
- `liteloader`
- `rift`

##### `required`

This field MUST be present on dependencies with the `asset` type. It CANNOT be present on dependencies with the `framework` type. This field is a boolean that indicates whether a dependency is required in order for the Project requiring it to function properly.

##### `version`

This field MUST be either an array or String. If this field is an array, it MUST contain a list of compatible versions. If this field is a String, it MUST contain a Semantic Versioning comparison String. An example of a comparison string is `>= 25.0.219`

---

### `files`

This field contains the downloadable files of a Version. It MUST be an array of File Objects.
File Objects contain information on a downloadable file. Fields are as follows:

##### `name`

There is no restriction on what this value can be. However, it is RECOMMEND that this be the name of the file. Launchers and other automation tools MAY ignore the value. This value will be shown to users.

##### `sha256`

An SHA256 checksum of the file. This may be used by launchers in order to verify download integrity.

##### `rel` (optional)

The relation of this file. This MUST be one of the following values:

- `primary` - The primary file of this version. This is what launchers should download.
- `source` - Source code of this version
- `deobf` - Deobfuscation file for this version

If this field is not present, the file should be assumed to be `primary`. The values of this field CAN be present in multiple files. For example, a version can have a file for both the Forge and Fabric modloaders, and both of those files can have the `rel` field set to `primary`.

##### `dependencies` (optional)

This field MUST conform to the same specification as the Version's dependencies. BOTH a File and Version can have dependencies.
It is RECOMMENDED to define Framework/Modloader dependencies as File Dependencies and NOT Version Dependencies. When downloading a file, launchers SHOULD pay attention to both the Version and FIle dependencies.

##### `installationMethod`

The installation method of this file. Valid options are:

- `placeInDirectory` - This file must be placed in the directory specified in a separate `directory` field
- `modsDirectory` - A shorthand for `placeInDirectory` with `"directory": "mods"` as it's very common
- `jarMod` - This file is a Jar mod and must be added into minecraft.jar
- `other`

##### `downloads`

This field MUST be an array containing a download source of the file, stored as a String. The download sources go from highest priority to lowest priority inside the array. For example:

```
"downloads": [
  "https://firsthostingsite.com/example.jar",
  "https://secondhostingsite.com/example.jar
]
```

If a launcher was trying to download the file, it would first try at `firsthostingsite.com`. If it was unable to download from there, it would then try from `secondhostingsite.com`.
