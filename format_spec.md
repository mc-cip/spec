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

A short, one sentence summary of the Project. Any form of rich text such as HTML CANNOT be used.

---

## Detailed Project Information

### `description` (optional)

A long description of the project. HTML MAY be used. It is not recommended to use Markdown as there may be rendering inconsistencies. Any form of code execution such as JavaScript or WebAssembly SHOULD NOT be used. HTML forms, buttons, or other form of interactive elements SHOULD NOT be used. iframes to external domains MAY be used, but it is recommended to embed iframes only with a trustworthy website. It is recommended for launchers to take precautions and be cautions about loaded HTML.

---

### `media` (optional)

This field MUST be stored as an array. This contains Media Item objects, which have the following fields.

##### `rel`

The relation of this Media Item. Valid options are:

- `icon` - This is the main icon of this project.
- `gallery` - This item should be shown in a list of other items marked `gallery`.

This field CAN be an array of multiple `rel` values. Currently this is not needed, however it exists so more `rel` values could be added later.

The `icon` relation CAN ONLY be present in ONE Media Item. It CANNOT be present in multiple Media Items. It is NOT required.

The `gallery` relation CAN be present in multiple Media Items.

If an item with the `icon` relation is not present, launchers should show a placeholder.

##### `type`

The type of this Media Item. Valid options are `video` and `image`. This field exists to indicate to launchers what this item is and to allow them to render correctly. Launchers SHOULD NOT infer type based on file extension.

##### `embed`

The `embed` field is valid for both `video` and `image`. It contains a URI to a webpage which SHOULD be embedded when displaying this Media Item. It is recommended for hosts to only allow trustworthy domains to be embedded.

There is no difference between embedded videos and images, as they both display webpages, however the type field MUST still be specified.

##### `src`

The `src` field is valid for both `video` and `image`. It contains a direct URI to either a playable video file or an image file. It is RECOMMENDED for images to be in the WebP or PNG formats, and it is RECOMMENDED for videos to be in the WebM format. Recommended video codecs include VP9 and AV1.

Base64 encoded images, which are typically prefixed with `data:base64` are NOT RECOMMENDED, however they CAN be present. If a Base64 encoded image is present, it MUST be prefixed with `data:base64`.

`embed` and `src` are conflicting fields and both fields CANNOT be present in the same Media Item.

##### `caption` (optional)

An optional string which can be displayed as a caption for the current Media Item. Any form of rich text such as HTML CANNOT be used. `caption` CANNOT be present when `rel` is set to `icon`.

##### `sha256` (optional)

An optional SHA256 checksum for the media file. This field CANNOT be present if the `embed` field is present.

---

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

###### `sha256` (optional)

An optional SHA256 checksum of the icon specified in `uri`.

###### `uri`

The URI of this icon. This field is required to be present if the `icon` object is present.

---

### `license` (optional)

This is an object that contains information about the license that a project is under. It contains the following fields:

##### `id`

The identifier of the license. Valid options are:

- [All SPDX License Identifiers](https://spdx.org/licenses/)
- `ARR` - All Rights Reserved
- `MMPL` - Minecraft Mod Public License
- `Generic-Open` - An generic open source license
- `Generic-Proprietary` - An generic proprietary license

Use the `Generic-Open` and `Generic-Proprietary` IDs for when a Project is using a custom license or a license not specified in the list above.

##### `name`

The human readable name of the license. It is recommended to keep it short and similar to the identifier specified in `id`. For identifiers users may not be familiar with, it is recommended to expand them. For example, use `All Rights Reserved` instead of `ARR`. Similarly, use `Minecraft Mod Public License` instead of `MMPL`.

If using the `Generic-Open` or `Generic-Proprietary` IDs, use the actual name of the license for this field. If the license is specific to the Project, it's recommended to use `Custom License` as the name.

##### `uri`

A URL linking to a readable copy of the license.

##### `modpackUsage` (optional)

A field describing usage of the Project in modpacks. Valid options:

- `alwaysAllow` - This Project can be used in a modpack without asking for permission.
- `requiresPermission` - Before included in a modpack, permission must be granted from the author.
- `neverAllow` - This Project can never be used in a modpack.

This field overrides any restrictions stated in the Project's license. If you are a mod author and set this field to `alwaysAllow`, you are granting permission for redistribute the files linked in this metadata to be redistributed inside a modpack.

If you are unsure whether or not you want your mod to be distributed in modpacks, `requiresPermission` is recommended.

If this field is not specified, launchers should assume `alwaysAlllow`.

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

Projects can contain a `versions` array, which lists Version Objects. These objects are stored in chronological order from most recent to least recent.

## Version Objects

Version Objects contain fields relating to a certain version of a Project. Below are all the fields that a Version Object may contain.

### `id`

The ID of this version. There is no required format for how to format a version ID, other than that they CANNOT include spaces.

---

### `name`

The human-readable name of this version. It is RECOMMENDED to follow Semantic Versioning, but it is not required. Spaces MAY be used in this name.

---

### `semver` (optional)

An optional Semantic Versioning compatible version number. It is HIGHLY RECOMMENDED that this field only be present when a version's name conforms to SemVer. This field CAN be present when the version's name does not conform to SemVer, however it's recommended to only be present if the name conforms.

Launchers can use this field to compare different versions. However, because many Minecraft mods do not conform to SemVer, it's recommended for launchers to compare using other fields, such as `releaseDate`.

---

### `releaseDate`

The release date of this version. It MUST be stored as an ISO-8601 conforming String. This MUST include UTC time at the end. A valid example is `2020-01-01T12:00:00Z`. This example date is the 1st of January, 2020 at 12:00:00 UTC. Spaces and time zone offsets (e.g. `+01:00`) CANNOT be used. The `Z` at the end of the string MUST be included and it MUST be capitalized. The `T` separating the date and time MUST be included and it MUST be capitalized. Other values, such as `2020-W32` are NOT allowed. The ONLY allowed format is the one demonstrated in the example.

If the time of a release is unknown, the value SHOULD default to `12:00:00Z`. If the date is completely unknown or hosts do not wish to provide inaccurate data, the value `unknown` CAN be passed.

---

### `changelog`

The changelog of this version. The changelog MAY contain HTML.

---

### `dependencies` (optional)

This field MUST be an array containing information about dependencies that may or may not be required for this project.

Required Minecraft versions MUST be stored as a dependency. They CANNOT contain the `src`, `required`, or `installation` fields. The ID MUST be `minecraft`. `src` is not specified as it is up to launchers to determine where to gather information about the Minecraft versions. They may gather it directly from Mojang, from the Index, or from another source.

Dependency objects contains the following fields:

##### `src`

This field contains where this dependency can be downloaded. It MUST either a URL or the name of the host stored as an all lowercase string.

This field CAN be an array of multiple Hosts that contain the same project with the same ID.

For hosts that utilize the recommended MCIP Standardized API (see **format_introduction.md**), this field MUST be the base URL of the API. For example, if the base URL of the `example` host is `api.example.com`, then this field MUST be `https://api.example.com`. URLs MUST contain `https://` and the non-secure `http://` prefix is NOT allowed. The value CANNOT contain a trailing slash.

For hosts that do not utilize the MCIP Standardized API, then this field MUST be an all-lowercase String representing the name of the host. Examples include `diluv`, `modrinth`, and `curseforge`. Using an `https://` prefix for hosts that do not use the MCIP Standardized API is NOT allowed.

Using the all-lowercase string is NOT RECOMMENDED, as it requires more implementation for launchers and can lead to incompatibility. It is HIGHLY RECOMMENDED to implement the MCIP Standardized API.

Information on how launcher developers should handle information contained in the `src` field is available in **format_introduction.md**.

If a source is completely unknown or there is no source that can be specified for whatever reason, the value `unknown` can be passed. From there it is up to launchers to figure out where to download this file. Because of this, it is NOT RECOMMENDED to pass the `unknown` value.

For dependencies that cannot be automatically downloaded at all, set this value to `manual`. This value SHOULD ONLY be used in causes where it is impossible to automate. For example, CurseForge _is_ possible to automate, so dependencies that are on CurseForge should not be set to `manual`, instead they should be set to `curseforge`. An example that _should_ be set to manual is the mod Galacticraft.

##### `id`

The ID of the required dependency. This should be the ID for the project that can be found at the host specified in `src`. If this project is a Framework, it MUST be one of the values listed in **format_values.md** for Framework IDs. Hosts serving Framework metadata MUST use these standardized IDs.

##### `required`

This field MUST be present on dependencies with the `asset` type. This field is a boolean that indicates whether a dependency is required in order for the Project requiring it to function properly.

##### `version`

This field MUST be either an array or String. If this field is an array, it MUST contain a list of compatible versions. If this field is a String, it MUST contain a Semantic Versioning comparison String. An example of a comparison string is `>= 25.0.219`

##### `installation` (optional)

This field describes how this version can be installed. It is NOT required. For more information, please view `files.installation`.

---

### `files`

This field contains the downloadable files of a Version. It MUST be an array of File Objects.
File Objects contain information on a downloadable file. Fields are as follows:

##### `name`

There is no restriction on what this value can be. However, it is RECOMMEND that this be the name of the file. Launchers and other automation tools MAY ignore the value. This value will be shown to users.

##### `sha256`

An SHA256 checksum of the file. This may be used by launchers in order to verify download integrity.

##### `rel`

The relation of this file. Possible values for this field are available in **format_values.md**. The values of this field CAN be present in multiple files. For example, a version can have a file for both the Forge and Fabric modloaders, and both of those files can have the `rel` field set to `primary`.

##### `dependencies` (optional)

This field MUST conform to the same specification as the Version's dependencies. BOTH a File and Version can have dependencies.

It is RECOMMENDED to define Framework/Modloader dependencies as File Dependencies and NOT Version Dependencies. When downloading a file, launchers SHOULD pay attention to both the Version and FIle dependencies.

##### `installation`

The installation method of this file. This MUST be stored as an object. The object MUST contain the field `method`. Information on the possible values in the installation object is [currently available as a Pull Request](https://github.com/mc-cip/spec/pull/7).

The installation field CAN be present for BOTH version and file objects, similar to dependencies. The installation method in the file will override the installation method in the version if present.

If the installation field is present in the Version, it is not required in the file. If it's NOT present in the version, it MUST be present for the file. The per-version installation field exists for Frameworks to avoid unnecessary fields and to be clearer about how a version should be installed. For an example Framework in the MCIP format, please view **format_example_fabric.md**.

##### `downloads`

This field MUST be an array containing a download source of the file, stored as a String. The download sources go from highest priority to lowest priority inside the array. For example:

```
"downloads": [
  "https://firsthostingsite.com/example.jar",
  "https://secondhostingsite.com/example.jar
]
```

If a launcher was trying to download the file, it would first try at `firsthostingsite.com`. If it was unable to download from there, it would then try from `secondhostingsite.com`.
