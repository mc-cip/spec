# Introduction to the MCIP Format

The MCIP Format is an open format for Minecraft Projects. A Minecraft Project is a Mod, Resource/Texture Pack, World, or Modpack.
The Format stores metadata about Projects such as their description, authors, download links, and more.

## Goals

- Create an open standard that can be used by Hosts like Diluv and Modrinth.
- Encourage open discussion and debate about the properties and metadata stored in the format.
- Make Minecraft Project distribution open and allow easy interoperability between Hosts.

## Users of the Format

The MCIP format is designed to be used by anyone who works with Minecraft assets. Examples include:

- Hosts, such as Diluv, Modrinth, and CurseForge
- Launchers, such as MultiMC and GDLauncher
- Mod Developers, who can provide metadata about their mods

## Format Documentation

Documentation on the Format specification is provided in **format_spec.md**.

## Implementing the Format

- Hosts that provide an API are encouraged to allow API users to request data be returned in the MCIP Format.
- Launcher developers are encouraged to allow browsing and installing Projects that are served by Hosts providing MCIP Format-compliant data.

### API and Field Modularization Advice

The MCIP format includes many fields. It would be inefficient for Hosts and API providers to return all information about a Project at once. Because of this, it's recommended to follow these guidelines:

**Separate Project information as needed**

For example, a user doesn't always need a Project's full description when using a Search endpoint. Instead, only return basic information for Search endpoints, such as `id`, `name`, `icon` and `summary`.

**Provide "compressed" versions**

If you have an API endpoint that lists all the versions of a Project, there's no need to return unnecessary fields like download links for every version. Instead, only return the `id`, `name`, `semver`, and `releaseDate` fields. You can then have a separate endpoint which can be used to get full information about a version using its ID.

**Separate changelogs if needed**.

If versions contain large changelogs, consider moving the `changelog` object into a separate request.

**Paginate Versions**

If Projects have a large amount of versions, consider paginating them. Instead of returning all versions within one request, return the first 10 or 20. From their, API users can add a `?index` parameter to the query to specify where to start getting versions at the next call.
