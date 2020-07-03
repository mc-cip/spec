
MCIP is two things: an index format, and a central repository.

The central repository is not the only repository - other people may also publish repositories
which aren't necessarily related to the central one.

# Repository linking

Repositories should support mirroring other repositories. 

Two modes:

* Mirror - simple mirror, always keep up-to-date with the other repository
* Import - import all projects in the other repository into this one, optionally under a namespace prefix.

Note that the namespace prefix only affects the human-readable IDs of projects. Projects may also be identified by some
type of SHA256 hash which does not change when importing.

Note that every repository is responsible for the content on that repository. Creating a project does not guarantee
that all other repositories will mirror it accurately. Repositories are encouraged to add additional information to project pages,
if they have such information and they cannot get it accepted upstream.

# Overall network structure

We envision that:

* Other people may want to run central repositories.
* Groups of mod developers may want to use the same format to publish their mods. Central repositories can pull from it.
* Launcher developers may want to mirror a central repository, perhaps in a more efficient format for their launcher.

Overall, this forms a directed graph, including cycles, of repositories which import from each other. New content may be
introduced into any "upstream" repository and it will propagate to all other interested repositories.

# Trust model

We assume that a repository will not be configured to import from any other repositories which its administrator does not trust.

However, malicious projects will inevitably get introduced anyway, so it's important to be able to trace them.

The namespace prefix system allows one to simply identify the origin of a project. If Repo A has a project with the name
"repo_b:repo_c:mod_name" then it clearly came via Repo B (directly) and Repo C (indirectly).
Repo A's administrator may blacklist the project, but they should also work with Repo C's administrator to blacklist
the project at the source, or Repo B to stop importing from Repo C, if they are uncooperative.

Note: "malicious" here refers to spam, mods that harm computers or worlds, mods that impersonate other authors,
and so on. Repositories are **strongly discouraged** from blacklisting mods over simple disputes such as mods which
refuse to run based on the current player's UUID. However, they are free to add warning labels, or links to patched
versions of the mods.

# Is this a blockchain?

no
