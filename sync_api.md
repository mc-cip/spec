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
	"last_updated": "1337013370",
	"projects": [
		{"name": "example_project", "id": "uuid here", "last_updated": "1337001234"},
		{"name": "example_project_2", "id": "uuid here", "last_updated": "1234567890"},
		{"name": "example_repo_1:example_imported_project", "id": "uuid here", "last_updated": "1234566666"},
		{"name": "example_repo_1:example_repo_2:example_repo_3:example_nested_imported_project", "id": "uuid here", "last_updated": "1111111111"}
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

`GET https://example.com/api/project_list_v1?last_updated=1337013370`

Result:

```
{
	"last_updated": "1337031337",
	"projects": [
		{"name": "example_project", "id": "uuid here", "last_updated": "1337031334"},
		{"name": "example_project_2", "id": "uuid here", "last_updated": "1337031335"},
		{"name": "example_repo_1:example_imported_project_2", "id": "uuid here", "last_updated": "1337031336"},
	],
	"deleted": [
		{"name": "example_repo_1:example_repo_2:example_repo_3:example_nested_imported_project"}
	],
	
	// optional
	"suggested_polling_rate": 300,
}
```

This response only contains projects which were created, updated, or deleted, since the last request.
The `last_updated` field from the previous response is used as a parameter in the request.

`last_updated` doesn't have to be a timestamp or even a number. However, it must always be represented in JSON as a string. Clients shouldn't try to compare `last_updated` values. For example, they could be timestamps, sequence numbers, or git hashes.

The `last_updated` value for a project is not necessarily related to the `last_updated` value of the `projects_list`.

The presence of `deleted` indicates that the response is a delta response. The server can always choose to send a full response, in which case the `deleted` key is not present, and all projects which are not present in the response have been deleted.

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

# Project metadata

Query:

`GET https://example.com/api/project_v1/example_repo_1:example_imported_project_2`

Response:

```
???
```

TODO!

No deltas here. `last_updated` from the project list tells you whether the project was updated at all.
