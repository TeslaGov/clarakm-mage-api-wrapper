# Mage API Wrapper

## Introduction
This script can be imported and used in other python scripts to simplify automated operations relying on the MAGE API.

Common examples include:
* Creating events
* Editing event forms and observations
* Copying observations from one event to another

## Setup
1. Clone the repository from GitHub.
2. Add a `config.py` file to your project folder containing your credentials and MAGE server url. This file should not be uploaded to shared spaces.
3. Copy `mage_api.py` so it resides alongside your other script(s).
4. Import `mage_api.py` in your python script(s) to make use of the below functionality.

### Installing Dependencies

To manage dependencies, this repository includes a `requirements.txt` file. You can install dependencies easily via:

```shell
pip install -r requirements.txt
```

For your own projects, you should include a `requirements.txt` file as well. You can generate one by:

```shell
pip install pipreqs
pipreqs .
```

### Example `config.py`

```py
mage_environment = 'https://your-mage-url'

credentials = {
  'username': 'your_mage_username',
  'password': 'your_mage_password'
}
```


## Getting Started

First, create a session object which will be used to make all your API calls. This will log you in using the information in `config.py` and store your own MAGE user info as the `own_user` attribute.

```py
import mage_api

# instantiate MAGE session
session = mage_api.MageClient()

#this is just for demonstration
print(session.own_user)
```

To make requests, use one of the functions documented below like this:

```py
import mage_api

# instantiate MAGE session
session = mage_api.MageClient()

# get document for event with an id of 2000
my_event = session.get_event(2000)
```

API responses are returned as `formattedResponse` objects with the following attributes:

- `response_code` - indicates success or failure ([Mozilla Reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status))
- `response_body` - any JSON responses formatted as dictionaries, or strings (usually error messages) returned
- `content` - any downloaded files

Generally, you'll want to access the `response_body`, which will contain any information for observations or events fetched or added.

```py
import mage_api

# instantiate MAGE session
session = mage_api.MageClient()

# get document for event with an id of 2000
my_event = session.get_event(2000)

# get JSON response as dictionary
my_event_document = my_event.response_body

# you can also use pretty_print() for more info
my_event.pretty_print()
```

## Supported MAGE Operations

The following MAGE operations are supported with simple-to-use class functions (e.g. `session.get_event()`). The list isn't comprehensive at this time, but adding new functions is generally straightforward. Some operations also make use of multiple API calls (e.g. `session.clone_observations()`). Some functions below don't require API calls, and should be accessed directly. They are listed as such (e.g. `mage_api.get_event_forms()`). These usually require you to retrieve a JSON response using another function and pass the dictionary as a parameter.

### Authentication

#### log_out

`log_out` Log out, use this when you're finished

**Parameters**:
_None_

**Returns:**
A string indicating successful logout.

### Events

#### get_event

`get_event` Request document for MAGE event, including forms and other metadata.

**Parameters**:
- `event_id`: `int` numerical ID of requested event

**Returns:**
`formattedResponse` object with attributes:

- `response_code`
- `response_body` (requested event document)

#### get_events

`get_events` Request documents for all visible MAGE events.

**Parameters**:
- `populate`: `bool` include `teams` and `layers` in event documents

**Returns:**
`formattedResponse` object with attributes:
- `response_code`
- `response_body` (list of visible events)

#### create_event

`create_event` Creates new MAGE event and adds user (self) to that event.

**Parameters**:
- `name`: `string` name for new event
- `description`: `string` description for new event

**Returns:**
`formattedResponse` object with attributes:
- `response_code`
- `response_body` (created event document)

#### get_event_forms

`get_event_forms` Extracts list of forms from already retrieved event document.

**Parameters**:
- `event_document`: `dict` event document fetched

**Returns:**
A list of form documents as dictionaries.
  
#### get_event_team

`get_event_team` Extract team ID from already retrieved event document (does not make API call).

**Parameters**:
- `event_document`: `dict` event document fetched

**Returns:**
Team id as an `int`.

### Observations

#### get_observations

`get_observations` Gets all observations meeting specified criteria from specified event.

**Parameters**:
- `event_id`: `int` numerical id of event
- `start_date`: `str` filter for observations created after this date/time
- `end_date`: `str` filter for observations created before this date/time
- `observation_start_date`: `str` filter for observations with timestamps after this date/time
- `observation_end_date`: `str` filter for observations with timestamps before this date/time
- `states`: `str` filter for only `active` or `archive` observations

**Returns:**
`formattedResponse` object with attributes:
- `response_code`
- `response_body` (list of requested observation documents)
  
#### create_observation

`create_observation` Creates new observation using provided dictionary and event id.

**Parameters**:
- `observation`: `dict` dictionary corresponding to valid observation object (format depends on event and forms)
- `event_id`: `int` numerical id for destination event

**Returns:**
`formattedResponse` object with attributes:
- `response_code`
- `response_body` (created observation document)
  
#### clone_event_observations

`clone_event_observations` Creates copies of observations from one event in another, including forms and attachments.

Steps:
1. Copies forms to event (overwrites existing forms of same name)
2. Gets specified observations from first event
3. Copies observations to second event, downloading and re-uploading attachments as needed

**Parameters**:
- `origin_event_id`: `int` numerical id for event you wish to copy observations FROM
- `target_event_id`: `int` numerical id for event you wish to copy observations TO
- `start_date`: `str` filter for observations created after this date/time
- `end_date`: `str` filter for observations created before this date/time
- `observation_start_date`: `str` filter for observations with timestamps after this date/time
- `observation_end_date`: `str` filter for observations with timestamps before this date/time

**Returns:**
`formattedResponse` object with attributes:
- `response_code`
- `response_body` (list of requested observations from origin event)
  
#### get_observation_attachments

`get_observation_attachments` Downloads attachments for provided observation document.

**Parameters**:
- `observation`: `dict` previously retrieved observation document

**Returns:**
`attachments_mapping` dict:

```
{
    'observationId': observation_id,
    'eventId': event_id,
    'attachments': [attachment_paths]
}
```
  
#### create_observation_attachments

`create_observation_attachments` Uploads attachments using provided dict to determine event id, observation id, and attachment locations.

**Parameters**:
- `attachments_mapping`: `dict`:

```
{
    'observationId': observation_id,
    'eventId': event_id,
    'attachments': [attachment_paths]
}
```

**Returns:**
A list of `formattedResponse` objects for upload requests with attributes:
- `response_code`
- `response_body` (list of requested observations from origin event)

### Forms

#### create_form

`create_form` Create new form in event.

**Parameters**:
- `form`: `dict` dictionary corresponding to a valid form object
- `event_id`: `int` numerical id for destination event

**Returns:**
`formattedResponse` object with attributes:
- `response_code`
- `response_body` (created form document)
  
#### update_form

`update_form` Replace specified form with provided dictionary.

**Parameters**:
- `form`: `dict` dictionary corresponding to a valid form object
- `event_id`: `int` numerical id for destination event
- `form_id`: `int` numerical id for destination form

**Returns:**
`formattedResponse` object with attributes:
- `response_code`
- `response_body` (updated form document)

#### clone_forms

`clone_forms` Create new form in event.

**Parameters**:
- `forms_list`: `list` list of dictionaries corresponding to valid form objects
- `event_id`: `int` numerical id for destination event

**Returns:**
A dictionary mapping provided form names and ids to created forms and ids.

### Teams

#### session.add_user_to_team

`add_user_to_team` Add specified user to specified team.

**Parameters**:
- `user`: `dict` user object
- `team_id`: `int` numerical id for destination team

**Returns:**
`formattedResponse` object with attributes:
- `response_code`
- `response_body` (updated team document)
  
### Roles

#### get_roles

`get_roles` Fetch roles for own MAGE user.

**Parameters**:
_None._

**Returns:**
`formattedResponse` object with attributes:
- `response_code`
- `response_body` (list of available roles for user)

#### grant_event_role

`grant_event_role` Grant event-specific role to specified user.

**Parameters**:
- `event_id`: `int` numerical id for destination event
- `user_id`: `str` id for user to grant role
- `role`: `str` role to grant (owner, manager, guest)

**Returns:**
`formattedResponse` object with attributes:
- `response_code`
- `response_body` (created role document)