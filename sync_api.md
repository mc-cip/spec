# Sync API

The sync API allows one server (acting as a client in this case) to download all the data from another server and keep its copy up-to-date.

It's a hierarchical API, so the client can skip data blobs it's not interested in or doesn't support.

The base URL in the examples is `https://example.com/api/`

# Top level - project list (`$BASE/project_list_v1`)

Query:

`GET https://example.com/api/project_list_v1`

Result:

```
{
	"last_updated": "1",
	"projects": [
		{"name": "example_project", "id": "uuid here", "last_updated": "1"},
		{"name": "example_project_2", "id": "uuid here", "last_updated": "1"},
		{"name": "example_repo_1:example_imported_project", "id": "uuid here", "last_updated": "1"},
		{"name": "example_repo_1:example_repo_2:example_repo_3:example_nested_imported_project", "id": "uuid here", "last_updated": "1"}
	],
	
	// optional
	"suggested_polling_rate": 300,
}
```

The client can save `last_updated` in order to receive a delta next time it polls.

The primary identifier of a project is the name, not the UUID.

The project UUID is used to avoid cycles. If the client already has the same UUID via a shorter route, then it won't download that project from this server.
Servers which use the same UUID for different projects, or copy the UUID of existing projects, are considered malicious (or just broken).


## Delta fetch

Query:

`GET https://example.com/api/project_list_v1?last_updated=1`

Result:

```
{
	"last_updated": "2",
	"projects": [
		{"name": "example_project", "id": "uuid here", "last_updated": "76"},
		{"name": "example_project_2", "id": "uuid here", "last_updated": "0"},
		{"name": "example_repo_1:example_imported_project_2", "id": "uuid here", "last_updated": "flubber"},
	],
	"deleted_projects": [
		{"name": "example_repo_1:example_repo_2:example_repo_3:example_nested_imported_project"}
	],
	
	// optional
	"suggested_polling_rate": 300,
}
```

This response only contains projects which were created, updated, or deleted, since the last request.
The `last_updated` field from the previous response is used as a parameter in the request.

`last_updated` doesn't have to be a timestamp or even a number. However, it must always be represented in JSON as a string. Clients shouldn't try to compare `last_updated` values. For example, they could be timestamps, sequence numbers, or git hashes. The only rule is: if the value is different from the last known value, then the object in question has changed.

The `last_updated` value for a project is not necessarily related to the `last_updated` value of the `projects_list`.

The presence of `deleted_projects` indicates that the response is a delta response. The server can always choose to send a full response, in which case the `deleted_projects` key is not present, and all projects which are not present in the response have been deleted.

Note that the server must store all project names that were ever deleted, in order to generate delta responses. The server could delete this history, but then it will need to send a full response to each client.

## Polling rate

Clients which poll should avoid using a fast polling rate if they are not receiving incremental responses.

Polling rate is ultimately up to the client administrator, but the recommended defaults and minimum values are the lowest of:

* If the previous two responses were both deltas, or less than two requests have been sent:
    * Minimum: every 5 minutes
    * Default: every 20 minutes
* Otherwise:
    * Minimum: every 2 hours
    * Default: every 12 hours
* If the previous response included a `suggested_polling_rate` key:
    * Minimum: One third of `suggested_polling_rate` seconds
    * Default: Every `suggested_polling_rate` seconds

Note that the non-delta default rate is quite long, as the project list could be quite large. If a servers only hosts a few projects, so it wants a fast polling rate without bothering to implement deltas, it should set `suggested_polling_rate` to a low value.

# Project version list (`$BASE/project/$PROJECT/versions_v1`)

Query:

`GET https://example.com/api/project/example_project/versions_v1`

Response:

```
{
	"last_updated": "1",
	"versions": [
		{"name": "1.0.0", "last_updated": "1"},
		{"name": "1.0.0-interim-fix", "last_updated": "1"}
		{"name": "1.0.1", "last_updated": "1"},
	]
}
```


## Delta

Query:

`GET https://example.com/api/project/example_project/versions_v1?last_updated=1`

Response:

```
{
	"last_updated": "76",
	"versions": [
		{"name": "1.0.1-fabric", "last_updated": "1"},
		{"name": "1.0.1-forge", "last_updated": "1"}
	],
	"deleted_versions": [
		{"name": "1.0.0-interim-fix"}
	]
}
```

Note: one might think that versions are immutable. In practice, sometimes they are not. That decision is up to the originating server. For example, a version may have been initially released with incorrect dependency information, which is later corrected. Or a warning label may be added to a version, to indicate that it's known to cause world corruption. Either one of these counts as a version update. Versions which don't load at all might even be deleted.

Versions don't have UUIDs, because there's no need to combine information from different servers.

# Project version files (`$BASE/project/$PROJECT/version/$VERSION/v1`)

Query:

`GET https://example.com/api/project/example_project/version/1.0.1-fabric/v1`

Response:

```
{
	"last_updated": "1",
	"files": [
		{
			"filename": "example_project-1.0.0.jar",
			"sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855", // lowercase hex; required if there are URLs
			"urls": ["https://blah/blah", "https://blah2/blah2"], // optional
			"rel": "primary",
			"size": 1234, // in bytes; optional
		},
		{
			"filename": "example_project-1.0.0-source.jar",
			"sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855", // lowercase hex; required if there are URLs
			"urls": ["https://blah/blah", "https://blah2/blah2"], // optional
			"rel": ["source"]
		},
		{
			"filename": "crazy_yet_valid_file_entry.jar"
		}
	],
	"modloader": "fabric", // WILL PROBABLY CHANGE THIS
	"equivalent_versions": [{"name": "1.0.1-forge"}]
}
```

`rel` specifies the type of file - possibly more than one for the same file.  
`"rel": "primary"` means this is the "main" download. When you click on "download project", you get this file.  
`"rel": "source"` means this is the "main source" download. When you click on "download source code", you get this file.  
All files should be displayed somewhere, in the order they appear in the JSON, but `rel` allows for shortcuts.  
`rel` is optional; if not specified, it's the same as an empty list.  
If there's only one item in `rel`, it can be specified as a single string (without a list).

`filename` is a default filename for the download. It SHOULD be unique within a version.
It is mandatory. Clients should ignore files without a filename.

If `sha256` is present but `urls` is not (or is empty), then the author knows the file but isn't offering to give you a copy, perhaps for copyright reasons. The client may use any other method to download the file, such as using a copy it already has, downloading from a different server, or using a more advanced API. These methods are out of scope of this document. If a client cannot acquire the file, it should display the file anyway, but disable the download button.  
`sha256` and `urls` could conceivably both blank. This should be very rare, but clients must be prepared to handle it. Clients should display the file anyway, but disable the download button.  
If `urls` are present, but `sha256` is not, clients should ignore `urls`. This is for reliability reasons. Inevitably, one of the URLs will become outdated, and if 
clients used it anyway, they would download the error page and then serve it as the mod file.

`size` is completely optional; if present, clients may use it to improve the user experience, for example by populating progress bars more accurately.

`equivalent_versions` indicates different mod versions which can be substituted for this one.

There are no version deltas, because they don't increase in size over time. `last_updated` tells you whether the version was updated at all.

# Project description (`$BASE/project/$PROJECT/description_v1`)

Query:

`GET https://example.com/api/project/example_project_1/description_v1`

Response:

```
{
	"display_name": "Example Project 1",
	
	// Might be displayed in search results for example.
	// If not present, you might try to derive it from the beginning of summary_one_paragraph or full_description, or just display the mod name by itself.
	"summary_one_sentence": "Test project for the MCIP sync API.",
	
	// Might be displayed in search results for example, in a system which allows more space for search results.
	// If not present, you might use summary_one_sentence, or try to derive it from the beginning of full_description.
	"summary_one_paragraph": "This is a test project for the MCIP sync API. It shows what a project description looks like when represented as JSON. This is the medium-length summary.",
	
	// Displayed on the mod's page, where there's room to display tons of information.
	// TODO: SPECIFY FORMATTING. Probably based on XML / a subset of XHTML.
	// Note that Markdown parsing is ambiguous and limited; if you want to enter Markdown, the originating server must convert it to XML.
	// For now let's assume it's plain text and newlines correspond to paragraph breaks.
	"full_description": "Pretend there are 100 lines of text here detailing every feature in the mod.",
	
	"logo": {
		"sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855", // lowercase hex; optional
		"urls": ["https://blah/blah", "https://blah2/blah2"] // optional
	}
	
	"screenshots": [{
		"sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855", // lowercase hex; optional
		"urls": ["https://blah/blah", "https://blah2/blah2"], // optional
		"caption": "A windmill milling grain" // optional
	}], // multiple screenshots allowed; display them in order
	
	"authors": [{
		"name": "Contoso Corporation", // mandatory. Spaces allowed!
		"link": "http://example.com/", // optional; go here if name is clicked on
	}], // multiple authors allowed; display them in order; when there's only space for one author, display the first one.
	
	"links": [
		{"rel": "homepage", "display_name": "Official homepage", "url": "https://example.com/example_project_1"},
		{"rel": ["wiki", "documentation"], "display_name": "Wiki", "url": "https://example.com/example_project_1_wiki"},
		{"rel": ["github", "scm", "issue_tracker"], "display_name": "GitHub", "url": "https://github.com/github/platform-samples"},
	]
}
```

Note: Unlike downloads, sha256 is optional for images, since a corrupted screenshot or logo doesn't cause much of a problem.

If a client has no way to download a logo or screenshot - because it has no `urls` and the client has no other way to fetch it - it should be treated as if it didn't exist at all. Captions shouldn't be displayed by themselves.

Similarly to download `rel`s, link `rel`s allow for consistent shortcuts. They are not required to be present. There should usually be at most one of each.
The currently standardized values are:

* `"homepage"`: Official homepage
* `"documentation"`: Reference documentation (e.g. a wiki)
* `"wiki"`: A wiki. This doesn't say what kind of information is *on* the wiki...
* `"scm"`": Source Code Management repository - e.g. GitHub. But not necessarily GitHub.
* `"issue_tracker"`": Complaints go here.

Other values MAY be used (e.g. `bitbucket`), but please use common sense so they can be standardized later. If you are unsure whether your custom value is worthy of becoming standardized, make it start with `"x-"`.

Shortcuts MAY also be based on URL; a server may show a "GitHub" shortcut based on the fact that the URL points to a GitHub project)

All links should be displayed *somewhere*, regardless of `rel`. That is what `display_name` is for.  
`rel` is optional; if not specified, it's the same as an empty list. If there's only one item in `rel`, it can be specified as a single string (without a list).


# other stuff

TODO: What else is needed?