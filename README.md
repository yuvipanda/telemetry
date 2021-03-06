# Telemetry

[![CircleCI](https://circleci.com/gh/jupyter/telemetry.svg?style=svg)](https://circleci.com/gh/jupyter/telemetry) 
[![codecov](https://codecov.io/gh/jupyter/telemetry/branch/master/graph/badge.svg)](https://codecov.io/gh/jupyter/telemetry)

Telemetry for Jupyter Applications and extensions.


## Install

Jupyter's Telemetry library can be installed from PyPI.
```
pip install jupyter_telemetry
```

## Basic Usage

Telemetry provides a configurable traitlets object, `EventLog`, for structured event-logging in Python. It leverages Python's standard `logging` library for filtering, handling, and recording events. All events are validated (using [jsonschema](https://pypi.org/project/jsonschema/)) against registered [JSON schemas](https://json-schema.org/). 

Let's look at a basic example of an `EventLog`.
```python
import logging
from jupyter_telemetry import EventLog


eventlog = EventLog(
    # Use logging handlers to route where events
    # should be record.
    handlers=[
        logging.FileHandler('events.log')
    ],
    # List schemas of events that should be recorded.
    allowed_schemas=[
        'uri.to.event.schema'
    ]
)
```

EventLog has two configurable traits:
* `handlers`: a list of Python's `logging` handlers.
* `allowed_schemas`: a list of event schemas to record.

Event schemas must be registered with the `EventLog` for events to be recorded. An event schema looks something like:
```json
{
  "$id": "url.to.event.schema",
  "title": "My Event",
  "description": "All events must have a name property.",
  "type": "object",
  "properties": {
    "name": {
      "title": "Name",
      "description": "Name of event",
      "type": "string"
    }
  },
  "required": ["name"],
  "version": 1
}
```
2 fields are required:
* `$id`: a valid URI to identify the schema (and possibly fetch it from a remote address).
* `version`: the version of the schema.

The other fields follow standard JSON schema structure.

Schemas can be registered from a Python `dict` object, a file, or a URL. This example loads the above example schema from file.
```python
# Register the schema.
eventlog.register_schema_file('schema.json')
```

Events are recorded using the `record_event` method. This method validates the event data and routes the JSON string to the Python `logging` handlers listed in the `EventLog`.
```python
# Record an example event.
event = {'name': 'example event'}
eventlog.record_event(
    schema_id='url.to.event.schema',
    version=1,
    event=event
)
```
