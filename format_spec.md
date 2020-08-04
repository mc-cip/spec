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

The ID of the Project. There is no standard or required format on how to store an ID, other than it CANNOT include spaces. It may be a slug, UUID, random numer, or other method of storing IDs. It is RECOMMENDED that IDs are not tied to the name of a project.

---

### `name`

The name of a Project. This name MUST be a human readable name and SHOULD not contain unnecessary information. For instance, the name field should not specify where the project is being hosted, or the authors of a project.

---

### `summary` (optional)

A short, one sentence summary of the Project. HTML and Markdown SHOULD NOT be used.

---

### `icon` (optional)

A URL to an icon of the Project. It is RECOMMENDED to use JPEG or PNG images. Images encoded into the field, such as a Base64 encoded PNG, SHOULD NOT be used.

---

## Detailed Project Information

### `description` (optional)

A long description of the project. HTML and Markdown MAY be used. Any form of code execution such as JavaScript or WebAssembly SHOULD NOT be used. HTML forms, buttons, or other form of interactive elements SHOULD NOT be used. iframes to external domains MAY be used, but it is recommended to embed iframes only with a trustworthy website.

---

### `gallery` (optional)

This field MUST be stored as an array. This contains Gallery Item objects, which have the following fields.

##### `type`

The type of this Gallery Item. Valid options are `video` and `image`.

##### `embed`

The `embed` field is valid for both `video` and `image`. It contains a URI to a webpage which SHOULD be embedded when displaying this Gallery Item. It is recommended for hosts to only allow trustworthy domains to be embedded.

##### `src`

The `src` field is valid for both `video` and `image`. It contains a direct URI to either a playable video file or an image file. It is RECOMMENDED for images to be in PNG and JPEG formats, and it is RECOMMENDED for videos to be in the MP4 format.

##### `caption` (optional)

An optional string which can be displayed as a caption for the current Gallery Item. Markdown and HTML CANNOT be used.

`embed` and `src` are conflicting fields and both fields CANNOT be present in the same Gallery Item.

### `authors` (optional)

This field MUST be stored as an array. It contains objects storing the information about each author. Those objects MUST be made up of the following fields:

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

A URL to an icon of the Category. The same requirements and recommendations of a Project's Icon apply.

### `license` (optional)

This is an object that contains information about the license that a project is under. It contains the following fields:

##### `name`

The human-readable name of the license. Examples include: "MIT" and "LGPLv3".

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

If a newer version of a Project has been released and it does not have the same ID as the previous version, the `successor` and `predecessor` fields may be set for the appropriate Projects. This fields MUST contain the ID of the predecessor or successor project.

---

## Version Information

Projects MAY contain a `versions` array, which is made up of Version Objects. If you are writing an API that uses the MCIP format, this field MAY be separated into a separate API call in order to save bandwidth. The `versions` field MAY also be paginated in API calls.

## Version Objects

Version Objects contain fields relating to a certain version of a Project. Below are all the fields that a Version Object may contain.

### `id`

The ID of this version. There is no required format for how to format a version ID, other than that they CANNOT include spaces.

---

### `name`

The human-readable name of this version. It is RECOMMENDED to follow Semantic Versioning, but it is not required. Spaces MAY be used in this name.

---

### `semver` (optional)

An optional Semantic Versioning compatible version number.

---

### `releaseDate`

The release date of this verison. It MUST be stored as an ISO-8601 conforming String.

---

### `changelog`

The changelog of this version. This MAY be moved into a separate API call for hosts. The changelog MAY contain HTML, Markdown, or other forms of rich text.

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

##### `src`

This field contains where this dependency can be downloaded. It contains the name of the host, in an all-lowercase string. Examples include: `curseforge`, `diluv` and `modrinth`. If the `type` field is set to `framework`, this field CANNOT exist;

##### `id`

The ID of the required dependency. If the dependency is a required asset and not a framework, then the ID of the asset stored at the host specified in the `src` field should be used. If the dependency is a required framework, then a framework from the list of standardized framework IDs should be used:

- `fabric-loader`
- `forge`
- `liteloader`
- `rift`

##### `required`

This field MUST be present on dependencies with the `asset` type. It CANNOT be present on dependencies with the `framework` type. This field is a boolean that indicates whether a dependecy is required in order for the Project requiring it to function properly.

##### `version`

This field MUST be either an array or String. If this field is an array, it MUST contain a list of compatible versions. If this field is a String, it MUST contain a Semantic Versioning comparison String. An example of a comparison string is `>= 25.0.219`

---

### `files`

This field contains the downloadable files of a Version. It MUST be an array of File Objects.
File Objects contain information on a downloadable file. Fields are as follows:

##### `name`

The full name of this file, including file extension.

##### `sha256` (optional)

An SHA256 checksum of the file. This may be used by launchers in order to verify download integiry.

##### `frameworks` (optional)

This field MUST be an array listing Frameworks supported by this file. If no Frameworks are required, this field MAY be omitted or empty. This field MUST contain Strings that use the standard Framework names mentioned in `dependencies.id`

##### `installationMethod` (optional)

The installation method of this file. Valid options are:

- `modsDirectory` - This file must be placed in the `mods` directory
- `jarMod` - This file is a Jar mod and must be added into minecraft.jar
- `other`

##### `download`

This field MUST be an array listing available download sources for this file. Each object containing in this array MUST contain the `uri` field, which is a URL pointing to a direct download of the file.

---

# API and Field Modularization Advice

The MCIP format includes many fields. It would be innefficient for Hosts and API providers to return all information about a Project at once. Because of this, it's recommended to follow these recommended guidelines:

**Separate Project information as needed**

For example, a user doesn't always need a Project's full description when using a Search endpoint. Instead, only return basic information for Search endpoints, such as `id`, `name`, `icon` and `summary`.

**Provide "compressed" versions**

If you have an API endpoint that lists all the versions of a Project, there's no need to return unnecessary fields like download links for every version. Instead, only return the `id`, `name`, `semver`, and `releaseDate` fields. You can then have a separate endpoint which can be used to get full information about a version using its ID.

**Separate changelogs if needed**.

If versions contain large changelogs, consider moving the `changelog` object into a separate request.

**Paginate Versions**

If Projects have a large amount of versions, consider paginating them. Instead of returning all versions within one request, return the first 10 or 20. From their, API users can add a `?index` paramter to the query to specify where to start getting versions at the next call.
