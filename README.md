# Mage API Wrapper

## Introduction
This script can be imported and used in other python scripts to simplify automated operations relying on the MAGE API.

Common examples include:
* Creating events
* Editing event forms and observations
* Copying observations from one event to another

## Setup
1. download from GitHub or other source and make sure `mage_api.py` is in the same folder as your other project scripts or can be imported from elsewhere

2. Add a `config.py` file to your project folder containing your credentials and MAGE server url. This file should not be uploaded to shared spaces.

**Example**
```
mage_environment = 'https://your-mage-url'

credentials = {
  'username': 'your_mage_username',
  'password': 'your_mage_password'
}
```

After the above setup, `mage_api.py` can be imported in your python scripts to make use of the below functionality.

## Getting Started
First, create a session object which will be used to make all your API calls. This will log you in using the information in `config.py` and store your own MAGE user info as the `own_user` attribute.
```
import mage_api


# instantiate MAGE session
session = mage_api.MageClient()

#this is just for demonstration
print(session.own_user)
```

To make requests, use one of the functions documented below like this:

```
import mage_api


# instantiate MAGE session
session = mage_api.MageClient()

# get document for event with an id of 2000
my_event = session.get_event(2000)
```

API responses are returned as `formattedResponse` objects with the following attributes:
* response_code - indicates success or failure ([Mozilla Reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status))

* response_body - any JSON responses formatted as dictionaries, or strings (usually error messages) returned

* content - any downloaded files

Generally, you'll want to access the `response_body`, which will contain any information for observations or events fetched or added.

```
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

The following MAGE operations are supported with simple-to-use class functions (e.g. `session.get_event()`).

The list isn't comprehensive at this time, but adding new functions is generally straightforward. 

Some operations also make use of multiple API calls (e.g. `session.clone_observations()`).

Some functions below don't require API calls, and should be accessed directly. They are listed as such (e.g. `mage_api.get_event_forms()`). These usually require you to retrieve a JSON response using another function and pass the dictionary as a parameter.

## Authentication

### session.log_out

_session.log_out()_

Log out, use this when you're finished

**Parameters**:
_None_

**Returns:**
* string indicating successful logout

## Events

### session.get_event

_session.get_event(event_id)_

Request document for MAGE event, including forms and other metadata.

**Parameters**:
* _event_id_: int<br>numerical ID of requested event

**Returns:**
* `formattedResponse` object with attributes:
  * _response_code_
  * _response_body_ (requested event document)

##
### session.get_events

_session.get_events(populate=True)_

Request documents for all visible MAGE events.

**Parameters**:
* _populate_: bool<br>include `teams` and `layers` in event documents

**Returns:**
* `formattedResponse` object with attributes:
  * _response_code_
  * _response_body_ (list of visible events)

##
### session.create_event

_session.create_event(name, description)_

Creates new MAGE event and adds user (self) to that event.

**Parameters**:
* _name_: str<br>name for new event

* _description_: str<br>description for new event

**Returns:**
* `formattedResponse` object with attributes:
  * _response_code_
  * _response_body_ (created event document)

##
### mage_api.get_event_forms

_mage_api.get_event_forms(event_document)_

Extracts list of forms from already retrieved event document

**Parameters**:
* _event_document_: dict<br>event document fetched

**Returns:**
* list of form documents as dictionaries
  
##
### mage_api.get_event_team

_mage_api.get_event_team(event_document)_

extract team ID from already retrieved event document (does not make API call)

**Parameters**:
* _event_document_: dict<br>event document fetched

**Returns:**
* Team id as int

## Observations

### session.get_observations

_session.get_observations(event_id, start_date=None, end_date=None, observation_start_date=None, observation_end_date=None, states='active')_

Gets all observations meeting specified criteria from specified event

**Parameters**:
* _event_id_: int<br>numerical id of event

* _start_date_: str<br>filter for observations created after this date/time

* _end_date_: str<br>filter for observations created before this date/time

* _observation_start_date_: str<br>filter for observations with timestamps after this date/time

* _observation_end_date_: str<br>filter for observations with timestamps before this date/time

* _states_: str<br>filter for only `active` or `archive` observations

**Returns:**
* `formattedResponse` object with attributes:
  * _response_code_
  * _response_body_ (list of requested observation documents)
  
##
### session.create_observation

_session.create_observation(observation, event_id)_

Creates new observation using provided dictionary and event id

**Parameters**:
* _observation_: dict<br>dictionary corresponding to valid observation object (format depends on event and forms)

* _event_id_: int<br>numerical id for destination event

**Returns:**
* `formattedResponse` object with attributes:
  * _response_code_
  * _response_body_ (created observation document)
  
##
### session.clone_event_observations

_session.clone_event_observations(origin_event_id, target_event_id, start_date=None, end_date=None,
                                 observation_start_date=None, observation_end_date=None)_

Creates copies of observations from one event in another, including forms and attachments

Steps:
1. Copies forms to event (overwrites existing forms of same name)
2. Gets specified observations from first event
3. Copies observations to second event, downloading and re-uploading attachments as needed

**Parameters**:
* _origin_event_id_: int<br>numerical id for event you wish to copy observations FROM

* _target_event_id_: int<br>numerical id for event you wish to copy observations TO

* _start_date_: str<br>filter for observations created after this date/time

* _end_date_: str<br>filter for observations created before this date/time

* _observation_start_date_: str<br>filter for observations with timestamps after this date/time

* _observation_end_date_: str<br>filter for observations with timestamps before this date/time

**Returns:**
* `formattedResponse` object with attributes:
  * _response_code_
  * _response_body_ (list of requested observations from origin event)
  
##
### session.get_observation_attachments

_session.get_observation_attachments(observation)_

Downloads attachments for provided observation document

**Parameters**:
* _observation_: dict<br>previously retrieved observation document

**Returns:**
* `attachments_mapping` dict:
```
{
    'observationId': observation_id,
    'eventId': event_id,
    'attachments': [attachment_paths]
}
```
  
##
### session.create_observation_attachments

_session.create_observation_attachments(attachments_mapping)_

Uploads attachments using provided dict to determine event id, observation id, and attachment locations

**Parameters**:
* _attachments_mapping_: dict<br>
```
{
    'observationId': observation_id,
    'eventId': event_id,
    'attachments': [attachment_paths]
}
```

**Returns:**
* list of `formattedResponse` objects for upload requests with attributes:
  * _response_code_
  * _response_body_ (list of requested observations from origin event)

## Forms

### session.create_form

_session.create_form(form, event_id)_

Create new form in event

**Parameters**:
* _form_: dict<br>dictionary corresponding to a valid form object

* _event_id_: int<br>numerical id for destination event

**Returns:**
* `formattedResponse` object with attributes:
  * _response_code_
  * _response_body_ (created form document)
  
##
### session.update_form

_session.update_form(form, event_id, form_id)_

Replace specified form with provided dictionary

**Parameters**:
* _form_: dict<br>dictionary corresponding to a valid form object

* _event_id_: int<br>numerical id for destination event

* _form_id_: int<br>numerical id for destination form

**Returns:**
* `formattedResponse` object with attributes:
  * _response_code_
  * _response_body_ (updated form document)

##
### session.clone_forms

_session.clone_forms(forms_list, event_id)_

Create new form in event

**Parameters**:
* _forms_list_: list<br>list of dictionaries corresponding to valid form objects

* _event_id_: int<br>numerical id for destination event

**Returns:**
* dictionary mapping provided form names and ids to created forms and ids

## Teams

#### session.add_user_to_team

_session.add_user_to_team(user, team_id)_

Add specified user to specified team

**Parameters**:
* _user_: dict<br>user object

* _team_id_: int<br>numerical id for destination team

**Returns:**
* `formattedResponse` object with attributes:
  * _response_code_
  * _response_body_ (updated team document)
  
## Roles

### session.get_roles

_session.get_roles()

Fetch roles for own MAGE user

**Parameters**:
_None_

**Returns:**
* `formattedResponse` object with attributes:
  * _response_code_
  * _response_body_ (list of available roles for user)
  
##
### session.grant_event_role

_session.grant_event_role(event_id, user_id, role)_

Grant event-specific role to specified user

**Parameters**:
* _event_id_: int<br>numerical id for destination event

* _user_id_: str<br>id for user to grant role

* _role_: str<br>role to grant (owner, manager, guest)

**Returns:**
* `formattedResponse` object with attributes:
  * _response_code_
  * _response_body_ (created role document)