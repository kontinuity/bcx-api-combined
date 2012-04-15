Accesses
========

Get accesses
------------

* `GET /projects/1/accesses.json` will return all the people with access to the project.
* `GET /calendars/1/accesses.json` will return all the people with access to the calendar.

```json
[
  {
    "id": 149087659,
    "name": "Jason Fried",
    "email_address": "jason@37signals.com",
    "updated_at": "2012-03-22T16:56:48-05:00",
    "url": "https://basecamp.com/999999999/api/v1/people/149087659-jason-fried.json"
  },
  {
    "id": 1071630348,
    "name": "Jeremy Kemper",
    "email_address": "jeremy@37signals.com",
    "updated_at": "2012-03-22T16:56:48-05:00",
    "url": "https://basecamp.com/999999999/api/v1/people/1071630348-jeremy-kemper.json"
  }
]
```


Grant access
------------

* `POST /projects/1/accesses.json` will grant access to the project for the existing `ids` of people already on the account or new people via `email_addresses`. (Same goes for calendars with /calendars/ instead)

```json
{
  "ids": [ 5, 6, 10 ],
  "email_addresses": [ "someone@example.com", "someoneelse@example.com" ]
}
```

You can get the ids of existing people on the account from the [people API](https://github.com/37signals/bcx-api/blob/master/sections/people.md).

This will return `204 No Content` if the access was granted successfully. If the authenticated user does not have access to this project, `404 Not Found` will be returned.


Revoke access
-------------

* `DELETE /projects/1/accesses/1.json` will revoke the access of the person who's id is mentioned in the URL.  (Same goes for calendars with /calendars/ instead)

This will return `204 No Content` if the revoke was a success. If the user does not have access to revoke access from the project, `403 Forbidden` will be returned. If the authenticated user does not have access to this project, `404 Not Found` will be returned.
Attachments
===========

Submitting files to Basecamp is a two step process:

1. Create the attachment, receive a token verifying the upload was successful ("Create attachment" endpoint)
2. Attach the file to a comment, message, or upload. See the following endpoints for attaching:
   * [Create uploads](https://github.com/37signals/bcx-api/blob/master/sections/uploads.md)
   * [Create comments](https://github.com/37signals/bcx-api/blob/master/sections/comments.md)
   * [Create messages](https://github.com/37signals/bcx-api/blob/master/sections/messages.md)

Create attachment
-----------------

* `POST /attachments.json` uploads a file. The request body should be the
binary data of the attachment. Make sure to set the `Content-Type` and
`Content-Length` headers.

Once the upload is successful, you'll get a `200 OK` response, and we'll give
you a token back that you'll need to save locally to attach the file.

```json
{
  "token": "4f71ea23-134660425d1818169ecfdbaa43cfc07f4e33ef4c"
}
```

With `curl`, here's an example:

```
curl --data-binary @logo.png \
       -u user:pass \
       -H 'Content-Type: image/png' \
       -H 'User-Agent: Rapp (david@37signals.com)' \
       https://basecamp.com/999999999/api/v1/attachments.json
```

*Note:* Uploading can take a while, if the file is big! Make sure to account for this in your implementation.


Get attachments
---------------

* `GET /projects/1/attachments.json` will show attachments for this
project with file metadata, urls, and associated attachables (Uploads, Messages,
or Comments) with a `200 OK` response.

If you need more information about what the attachment is attached to, you can
make another request to the `attachable`'s `url` parameter.

We will return 50 attachments per page. If the
result set has 50 attachments, it's your responsibility to check the next page 
to see if there are any more attachments -- you do this by adding `&page=2` to the 
query, then `&page=3` and so on.

```json
[
  {
    "key": "40b8a84cb1a30dbe04457dc99e094b6299deea41",
    "name": "bearwave.gif",
    "byte_size": 508254,
    "content_type": "image/gif",
    "created_at": "2012-03-27T22:48:49-04:00",
    "url": "https://basecamp.com/1111/api/v1/projects/2222/attachments/3333/40b8a84cb1a30dbe04457dc99e094b6299deea41/original/bearwave.gif",
    "creator": {
      "id": 73,
      "name": "Nick Quaranto"
    },
    "attachable": {
      "id": 70219655,
      "type": "Upload",
      "url": "https://basecmap.com/1111/api/v1/projects/2222/uploads/70219655.json"
    }
  }
  {
    "key": "773c74212f81f5c7d66917fb7236d5aece36c56a",
    "name": "report.pdf",
    "byte_size": 508254,
    "content_type": "application/pdf",
    "created_at": "2012-03-27T22:48:49-04:00",
    "url": "https://basecamp.com/1111/api/v1/projects/2222/attachments/4444/773c74212f81f5c7d66917fb7236d5aece36c56a/original/report.pdf",
    "creator": {
      "id": 73,
      "name": "Nick Quaranto"
    },
    "attachable": {
      "id": 12092382,
      "type": "Message",
      "url": "https://basecmap.com/1111/api/v1/projects/2222/messages/12092382.json"
    }
  }
]
```
Authentication
==============

> Speak, friend, and enter.

All Basecamp API requests are authenticated by passing along an OAuth 2 token or your own username & password.


Username & Password
-------------------

To hit the ground running, just use HTTP Basic authentication with your own login info:

```shell
curl -u username:password -H 'User-Agent: MyApp (yourname@example.com)' https://basecamp.com/999999999/api/v1/projects.json
```

_Never ask a user for their Basecamp login info!_

You're free to use your own username & password to access your own account and
to get started with the API. OAuth 2 is a simple protocol, but it's yet another
speed bump to getting an integration off the ground.

Your HTTP client software includes built-in support for HTTP Basic authentication.
Just provide your username and password.


OAuth 2
-------

For a full app integration, you wouldn't want to get into the business of asking
customers for their passwords -- or storing them! -- so we offer a simple way to
ask a user for access to his account. You get an API access token back without
ever having to see his password or ask him to copy/paste an API key.

1. [Grab an OAuth 2 library](http://oauth.net/code/).
2. Register your app at [integrate.37signals.com](https://integrate.37signals.com). You'll be assigned a `client_id` and `client_secret`. You'll need to provide a `redirect_uri`: a URL where we can send a verification code. Just enter a dummy URL like `http://myapp.com/oauth` if you're not ready for this yet.
3. Configure your OAuth 2 library with your `client_id`, `client_secret`, and `redirect_uri`. Tell it to use `https://launchpad.37signals.com/authorization/new` to request authorization and `https://launchpad.37signals.com/authorization/token` to get access tokens.
4. Try making an authorized request to `https://launchpad.37signals.com/authorization.json` to dig in and test it out!


OAuth 2 from scratch
--------------------

If you're going bare-metal and developing your own OAuth 2 client, you have a bit more work to do.

**TL;DR** request access, receive a verification code, trade it for an access token.


The typical flow for a web app:

1. Your app requests authorization by redirecting your user to Launchpad:

        https://launchpad.37signals.com/authorization/new?type=web_server&client_id=your-client-id&redirect_uri=your-redirect-uri

2. We authenticate their 37signals ID and ask whether it's ok to give access to your app. [Example of what this screen looks like](https://launchpad.37signals.com/authorization/new?type=web_server&client_id=0bf18204f5a28003bf7b9abb7e1db5e649d86ef4&redirect_uri=moist%3A%2F%2Foauth)

3. We redirect the user back to your app with a time-limited verification code.

4. Your app makes a backchannel request to trade the verification code for an access token. We authenticate your app and issue an access token:

        POST https://launchpad.37signals.com/authorization/token?type=web_server&client_id=your-client-id&redirect_uri=your-redirect-uri&client_secret=your-client-secret&code=verification-code

5. Your app uses the token to authorize API requests to any of the 37signals ID's accounts. Set the Authorization request header:

        Authorization: Bearer <tokenhere>

6. To get info about the 37signals ID you authorized and the accounts you have access to, make an authorized request to `https://launchpad.37signals.com/authorization.json` (or `/authorization.xml`).

Implementation notes:

* Start by reading the [draft spec](http://tools.ietf.org/html/draft-ietf-oauth-v2)
* We implement draft 5 and will update our implementation as the final spec converges. Be prepared for changes along the way.
* We support the web_server and user_agent flows, not the client_credentials or device flows.
* We issue refresh tokens. Use them to request a new access token when it expires (2 week lifetime, currently).
* We return more verbose errors than what's given in the spec to help with client development. We'll move these to a separate parameter later.
Calendar events
===============

> <Clever quote about todo lists>

A calendar event is an entry on the calendar (not to be confused with "events", which track activity in Basecamp -- yes, they could have been named better!).

The `starts_at` and `ends_at` are either dates if the calendar event is an all day affair or times with timezones if they're not.


Get calendar events
-------------------

* `GET /projects/1/calendar_events.json` will return upcoming calendar events for the project.
* `GET /calendars/1/calendar_events.json` will return upcoming calendar events for the calendar.
* `GET /projects/1/calendar_events/past.json` will return past calendar events for the project.
* `GET /calendars/1/calendar_events/past.json` will return past calendar events for the calendar.

```json
[
  {
    "id": 883432030,
    "summary": "something coming up",
    "description": "",
    "created_at": "2012-03-28T11:50:00-05:00",
    "updated_at": "2012-03-28T12:24:59-05:00",
    "all_day": false,
    "starts_at": "2012-03-28T07:00:00-05:00",
    "ends_at": "2012-03-28T07:00:00-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/calendar_events/883432030-something-coming-up.json"
  },
  {
    "id": 883432031,
    "summary": "More stuff for later",
    "description": "Details will follow",
    "created_at": "2012-03-28T12:29:16-05:00",
    "updated_at": "2012-03-28T12:29:16-05:00",
    "all_day": true,
    "starts_at": "2012-03-28",
    "ends_at": "2012-03-28",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/calendar_events/883432031-more-stuff-for-later.json"
  }
]
```

Get calendar event
------------------

* `GET /projects/1/calendar_events/1.json` will return the specified calendar event. 
* `GET /calendars/1/calendar_events/1.json` will return the specified calendar event. 

```json
{
  "id": 883432030,
  "summary": "something coming up",
  "description": "",
  "created_at": "2012-03-28T11:50:00-05:00",
  "updated_at": "2012-03-28T12:24:59-05:00",
  "all_day": false,
  "starts_at": "2012-03-28T07:00:00-05:00",
  "ends_at": "2012-03-28T07:00:00-05:00",
  "creator": {
    "id": 149087659,
    "name": "Jason Fried"
  },
  "comments": [
    {
      "id": 1028592772,
      "content": "let's get it taken care of?",
      "created_at": "2012-03-28T12:24:59-05:00",
      "updated_at": "2012-03-28T12:24:59-05:00",
      "attachments": [],
      "creator": {
        "id": 149087659,
        "name": "Jason Fried"
      }
    }
  ]
}

```


Create calendar event
---------------------

* `POST /projects/1/calendar_events.json` will create a new calendar event for a project.
* `POST /calendars/1/calendar_events.json` will create a new calendar event for a calendar.

Examples:

```json
{
  "summary": "My single, all-day event",
  "description": "Details to follow",
  "all_day": true,
  "starts_at": "2012-03-28"
}
```

```json
{
  "summary": "My all-day event spanning two days",
  "description": "Details to follow",
  "all_day": true,
  "starts_at": "2012-03-28",
  "ends_at": "2012-03-30"
}
```

```json
{
  "summary": "My single event for a specific time",
  "description": "Details to follow",
  "starts_at": "2012-03-28T11:50:00-05:00"
}
```

```json
{
  "summary": "My multi-day event starting at a specific time",
  "description": "Details to follow",
  "starts_at": "2012-03-28T11:50:00-05:00",
  "ends_at": "2012-03-28T07:00:00-05:00"
}
```

This will return `201 Created`, with the URL of the new calendar_event in the `Location` header and a JSON representation of the event in the response body, if the creation was a success. If the dates are not in the proper format, you'll get a `400 Bad Request`.


Update calendar event
---------------------

* `PUT /projects/1/calendar_events/1.json` will update the specific calendar event on a project.
* `PUT /calendars/1/calendar_events/1.json` will update the specific calendar event on a calendar.

```json
{
  "summary": "My all-day event spanning two days",
  "description": "Details to follow",
  "all_day": true,
  "starts_at": "2012-03-28",
  "ends_at": "2012-03-30"
}
```

This will return `200 OK` if the creation was a success, with a JSON representation of the resource in the response body. If the dates are not in the proper format, you'll get a `400 Bad Request`.


Delete calendar event
---------------------

* `DELETE /projects/1/calendar_events/1.json` will delete the calendar event specified and return `204 No Content` if that was successful. (The same for /calendars/)
Calendars
=========

Get calendars
-------------

* `GET /calendars.json` will return all calendars sorted alphabetically.

```json
[
  {
    "id": 336154974,
    "name": "Board Meetings",
    "updated_at": "2012-03-27T13:19:29-05:00",
    "url": "https://basecamp.com/999999999/api/v1/calendars/336154974-board-meetings.json"
  },
  {
    "id": 237581901,
    "name": "General",
    "updated_at": "2012-03-27T13:19:29-05:00",
    "url": "https://basecamp.com/999999999/api/v1/calendars/237581901-general.json"
  }
]
```


Get calendar
------------

* `GET /calendars/1.json` will return the specified calendar.

```json
{
  "id": 567469885,
  "name": "Vacation",
  "created_at": "2012-03-28T13:14:30-05:00",
  "updated_at": "2012-03-28T13:26:07-05:00",
  "creator": {
    "id": 149087659,
    "name": "Jason Fried"
  },
  "accesses": {
    "count": 3,
    "updated_at": "2012-03-28T13:14:31-05:00",
    "url": "https://basecamp.com/999999999/api/v1/calendars/567469885-vacation/accesses.json"
  },
  "calendar_events": {
    "count": 1,
    "updated_at": "2012-03-28T13:26:07-05:00",
    "urls": {
      "upcoming": "https://basecamp.com/999999999/api/v1/calendars/567469885-vacation/calendar_events.json",
      "past": "https://basecamp.com/999999999/api/v1/calendars/567469885-vacation/calendar_events/past.json"
    }
  }
}
```


Create calendar
---------------

* `POST /calendars.json` will create a new calendar from the parameters passed.

```json
{
  "name": "This is my new calendar!"
}
```

This will return `201 Created`, with the location of the new calendar in the `Location` header along with a representation of the calendar in JSON in the response body if the creation was a success (See the **Get calendar** endpoint).


Update calendar
---------------

* `PUT /calendars/1.json` will update the calendar from the parameters passed.

```json
{
  "name": "This is a new name for the calendar!"
}
```

This will return `200 OK` if the update was a success, along with a represenation of the calendar in JSON (See the **Get calendar** endpoint). If the user does not have access to update the calendar, you'll see `403 Forbidden`.


Delete calendar
---------------

* `DELETE /calendars/1.json` will delete the calendar specified and return `204 No Content` if that was successful. If the user does not have access to delete the calendar, you'll see `403 Forbidden`.
Comments
========

> HATERS GONNA HATE


Get comments
------------

Comments are included on the [topics](https://github.com/37signals/bcx-api/blob/master/sections/topics.md) directly. So to see all comments for a message, you'd just GET that message and they're included and look like this:

```json
{
  "comments": [
    {
      "id": 1028592764,
      "content": "Yeah, really, welcome!",
      "created_at": "2012-03-22T16:56:48-05:00",
      "updated_at": "2012-03-22T16:56:48-05:00",
      "creator": {
        "id": 149087659,
        "name": "Jason Fried"
      }
    }
  ]
}
```


Create comment
--------------

* `POST /projects/1/<section>/1/comments.json` will create a new comment from the parameters passed for the commentable described via <section>/<id> -- for example /projects/1/messages/1/comments.json or /projects/1/todos/1/comments.json.

```json
{
  "content": "Imma let you finish, but...",
}
```

This will return `201 Created`, with a representation of the comment just created in the response body if the creation was a success. The topic can be accessed via the `topic_url` parameter. For example:

```json
{
  "id": 1028592764,
  "content": "Yeah, really, welcome!",
  "created_at": "2012-03-22T16:56:48-05:00",
  "updated_at": "2012-03-22T16:56:48-05:00",
  "creator": {
    "id": 149087659,
    "name": "Jason Fried"
  },
  "topic_url": "https://basecamp.com/9999999/api/v1/messages/888888.json"
}
```

### Attaching files

Attaching files to a comment requires both the token and the name of the attachment. The
token is returned from the [Create attachments](https://github.com/37signals/bcx-api/blob/master/sections/attachments.md)
endpoint, which you must hit first before creating an upload.

The `name` parameter *must* be a valid filename with an extension. Multiple
attachments are allowed.

```json
{
  "content": "Here's the stuff",
  "attachments": [
    {
      "token": "4f71ea23-134660425d1818169ecfdbaa43cfc07f4e33ef4c",
      "name": "final_mockup.png"
    },
    {
      "token": "4f71ea23-458294fc0d87927301c5d54b69a7517602939e2c",
      "name": "draft_agreement.png"
    }
  ]
}
```


Delete comment
-------------

* `DELETE /projects/1/comments/1.json` will delete the comment specified and return `204 No Content` if that was successful. If the user does not have access to delete the comment, you'll see `403 Forbidden`.
Documents
========

All documents are automatically version-tracked. The API only exposes the most recent version of a document, though. Also, in the web UI we provide lock tracking to make sure people don't overwrite each other's work. There's no such automatic protection via the API. You're responsible yourself for managing this. Of course, everything is versioned so there won't be any lost data.

Get documents
-------------

* `GET /projects/1/documents.json` will return all the documents on the project ordered alphabetically by `title`.

```json
[
  {
    "id": 963979453,
    "title": "Manifesto",
    "updated_at": "2012-03-27T13:39:33-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/documents/963979453-manifesto.json"
  },
  {
    "id": 243535881,
    "title": "Really important notes",
    "updated_at": "2012-03-27T13:39:12-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/documents/243535881-really-important.json"
  }
]
```


Get document
------------

* `GET /projects/1/documents/1.json` will return the specified document along with all comments.

```json
{
  "id": 963979453,
  "title": "Manifesto",
  "content": "Do this<br>Then that<br>Finally just so!",
  "created_at": "2012-03-27T13:19:29-05:00",
  "updated_at": "2012-03-27T13:53:24-05:00",
  "last_updater": {
    "id": 149087659,
    "name": "Jason Fried"
  },
  "comments": [
    {
      "content": "I think there should be more sass to it.",
      "created_at": "2012-03-27T13:53:24-05:00",
      "updated_at": "2012-03-27T13:53:24-05:00",
      "creator": {
        "id": 149087659,
        "name": "Jason Fried"
      }
    }
  ]
}
```


Create document
---------------

* `POST /projects/1/documents.json` will create a new document from the parameters passed.

```json
{
  "title": "Very important business notes",
  "content": "The TPS report is due on Monday morning!"
}
```

This will return `201 Created`, with the location of the new project in the `Location` header along with a JSON representation of the document in the response body, if the creation was a success. See the **Get document** endpoint for more info.


Update document
---------------

* `PUT /projects/1/documents/1.json` will update the message from the parameters passed.

```json
{
  "title": "Really, super duper important business notes",
  "content": "Now I want the report by SUNDAY!"
}
```

This will return `200 OK` if the update was a success, along with a JSON representation of the document in the response body. See the **Get document** endpoint for more info.


Delete document
--------------

* `DELETE /projects/1/documents/1.json` will delete the document specified and return `204 No Content` if that was successful. If the user does not have access to delete the document, you'll see `403 Forbidden`.
Events
======

> <Clever quote about accesses>

All actions in Basecamp generate an event for the progress log. If you start a new todo list, there's an event. If you give someone access to a project, there's an event. If you add a comment. You get the drill.

If you're using this API for polling, please make sure that you're using the `since` parameter to limit the result set. Use the `created_at` time of the first item on the list for subsequent polls. If there's nothing new since that date, you'll get `[]` back.


Get global events
-----------------

* `GET /events.json?since=2012-03-24T11:00:00-06:00` will return all the events on the account since 11am CST March 24, 2012. We will return 50 events per page. If the result set has 50 entries, it's your responsibility to check the next page to see if there are any more events -- you do this by adding `&page=2` to the query, then `&page=3` and so on.

```json
[
  {
    "id": 1054456336,
    "created_at": "2012-03-24T11:00:50-05:00",
    "updated_at": "2012-03-24T11:00:50-05:00",
    "creator": {
      "id": 149087659,
      "name": "Jason Fried"
    },
    "bucket": {
      "id": 605816632,
      "name": "BCX",
      "type": "Project",
      "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx.json"
    },
    "summary": "re-assigned a to-do to Funky ones: Design it",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/todos/223304243-design-it.json"
  },
  {
    "id": 1054456334,
    "created_at": "2012-03-24T11:00:39-05:00",
    "updated_at": "2012-03-24T11:00:39-05:00",
    "creator": {
      "id": 149087659,
      "name": "Jason Fried"
    },
    "bucket": {
      "id": 605816632,
      "name": "BCX",
      "type": "Project",
      "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx.json"
    },
    "summary": "created a to-do list: lists",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/todolists/1056802576-lists.json"
  },
  {
    "id": 973672263,
    "created_at": "2012-03-24T09:53:35-05:00",
    "updated_at": "2012-03-24T09:53:35-05:00",
    "creator": {
      "id": 149087659,
      "name": "Jason Fried"
    },
    "bucket": {
      "id": 605816632,
      "name": "BCX",
      "type": "Project",
      "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx.json"
    },
    "summary": "commented on Prep the materials before the board meeting with Bezos",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/calendar_events/174886926-prep-the-materials.json"
  }
]
```

As you can see from the result set, the summary only contains a text version of what happened. You'll want to check the url of the eventable to get the latest structured data. Also note that if the `created_at` and `updated_at` fields are different, it means that the event was updated within the 15 minutes "correction window" we allow for people to fix spelling mistakes etc.

The bucket type can either be `Project` or `Calendar`.


Get project events
------------------

* `GET /projects/1/events.json?since=2012-03-24T11:00:00-06:00` will return all the events on the project like the global GET, except there won't be a bucket included. Similar use of the since term and the pagination.

```json
[
  {
    "id": 1054456336,
    "created_at": "2012-03-24T11:00:50-05:00",
    "updated_at": "2012-03-24T11:00:50-05:00",
    "creator": {
      "id": 149087659,
      "name": "Jason Fried"
    },
    "summary": "re-assigned a to-do to Funky ones: Design it",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/todos/223304243-design-it.json"
  },
  {
    "id": 1054456335,
    "created_at": "2012-03-24T11:00:44-05:00",
    "updated_at": "2012-03-24T11:00:44-05:00",
    "creator": {
      "id": 149087659,
      "name": "Jason Fried"
    },
    "summary": "added a to-do: t",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/todos/1046098402-t.json"
  }
]
```


Get person events
------------------

* `GET /people/1/events.json?since=2012-03-24T11:00:00-06:00` will return all the events by that person like the global GET, except there won't be creator parameter. Similar use of the since term and the pagination.

```json
[
  {
    "id": 1054456336,
    "created_at": "2012-03-24T11:00:50-05:00",
    "updated_at": "2012-03-24T11:00:50-05:00",
    "bucket": {
      "id": 605816632,
      "name": "BCX",
      "type": "Project",
      "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx.json"
    },
    "summary": "re-assigned a to-do to Funky ones: Design it",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/todos/223304243-design-it.json"
  },
  {
    "id": 1054456334,
    "created_at": "2012-03-24T11:00:39-05:00",
    "updated_at": "2012-03-24T11:00:39-05:00",
    "bucket": {
      "id": 605816632,
      "name": "BCX",
      "type": "Project",
      "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx.json"
    },
    "summary": "created a to-do list: lists",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/todolists/1056802576-lists.json"
  }
]
```Messages
========

> What a lot we lost when we stopped writing letters. You can't reread a phone call. - Liz Carpenter


Get messages
------------

Messages are listed alongside all the other [topics](https://github.com/37signals/bcx-api/blob/master/sections/topics.md), so there is no individual index for them.


Get message
-----------

* `GET /projects/1/messages/1.json` will return the specified message.

```json
{
  "id": 936075699,
  "subject": "Welcome!",
  "created_at": "2012-03-22T16:56:51-05:00",
  "updated_at": "2012-03-22T16:56:51-05:00",
  "content": "This is a new message",
  "creator": {
    "id": 149087659,
    "name": "Jason Fried"
  },
  "comments": [
    {
      "id": 1028592764,
      "content": "Yeah, really, welcome!",
      "created_at": "2012-03-22T16:56:48-05:00",
      "updated_at": "2012-03-22T16:56:48-05:00"
      "creator": {
        "id": 149087659,
        "name": "Jason Fried"
      }
    }
  ]
}
```


Create message
--------------

* `POST /projects/1/messages.json` will create a new message from the parameters passed.

```json
{
  "subject": "Hello everyone",
  "content": "This is going to be a GREAT Saturday!"
}
```

This will return `201 Created`, with the location of the new project in the `Location` header along with the current JSON representation of the message  if the creation was a success. See the **Get message* endpoint for more info.

### Attaching files

Attaching files to a message requires both the token and the name of the attachment. The
token is returned from the [Create attachments](https://github.com/37signals/bcx-api/blob/master/sections/attachments.md)
endpoint, which you must hit first before creating an upload.

The `name` parameter *must* be a valid filename with an extension. Multiple
attachments are allowed.

```json
{
  "subject": "Totally done!",
  "content": "I finished the reports, check them out!",
  "attachments": [
    {
      "token": "4f73595a-39a6fd18317b1eeffb9c4734e95a179aa4b1b7c8",
      "name": "cover_page.pdf"
    },
    {
      "token": "4f73595f-78efbe63c77a4f5c752ce7d113d0361220f70b69",
      "name": "final_draft.pdf"
    }
  ]
}
```


Update message
--------------

* `PUT /projects/1/messages/1.json` will update the message from the parameters passed.

```json
{
  "subject": "This is a new subject for the message!",
  "content": "And new content..."
}
```

This will return `200 OK` if the update was a success, along with the current JSON representation of the message in the response body. If the user does not have access to update the message, you'll see `403 Forbidden`. See the **Get message** endpoint for more info.


Delete message
-------------

* `DELETE /projects/1/messages/1.json` will delete the message specified and return `204 No Content` if that was successful. If the user does not have access to delete the message, you'll see `403 Forbidden`.
People
======

> The major problems of our work are not so much technological as sociological in nature - T. DeMarco & T. Lister


Get people
----------

* `GET /people.json` will return all people on the account.

```json
[
  {
    "id": 149087659,
    "name": "Jason Fried",
    "email_address": "jason@37signals.com",
    "updated_at": "2012-03-22T16:56:48-05:00",
    "url": "https://basecamp.com/999999999/api/v1/people/149087659-jason-fried.json"
  },
  {
    "id": 1071630348,
    "name": "Jeremy Kemper",
    "email_address": "jeremy@37signals.com",
    "updated_at": "2012-03-22T16:56:48-05:00",
    "url": "https://basecamp.com/999999999/api/v1/people/1071630348-jeremy-kemper.json"
  }
]
```

Get person
----------

* `GET /people/1.json` will return the specified person.
* `GET /people/me.json` will return the current person.

```json
{
  "id": 149087659,
  "name": "Jason Fried",
  "email_address": "jason@37signals.com",
  "created_at": "2012-03-22T16:56:51-05:00",
  "updated_at": "2012-03-23T13:55:43-05:00",
  "events": {
    "count": 19,
    "updated_at": "2012-03-23T13:55:43-05:00",
    "url": "https://basecamp.com/999999999/api/v1/people/149087659-jason-fried/events.json"
  }
}
```


Create person
-------------

New people can be invited directly to projects via the [accesses API](https://github.com/37signals/bcx-api/blob/master/sections/accesses.md).


Delete person
------------

* `DELETE /people/1.json` will delete the person specified and return `204 No Content` if that was successful. If the user does not have access to delete the person, you'll see `403 Forbidden`.
Projects
========

> Operations keeps the lights on, strategy provides a light at the end of the tunnel, 
> but project management is the train engine that moves the organization forward - Joy Gumz


Get projects
------------

* `GET /projects.json` will return all active projects.
* `GET /projects/archived.json` will return all archived projects.

```json
[
  {
    "id": 605816632,
    "name": "BCX",
    "description": "The Next Generation",
    "updated_at": "2012-03-23T13:55:43-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx.json",
    "archived": false
  },
  {
    "id": 684146117,
    "name": "Nothing here!",
    "description": null,
    "updated_at": "2012-03-22T16:56:51-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/684146117-nothing-here.json",
    "archived": false
  }
]
```


Get project
-----------

* `GET /projects/1.json` will return the specified project.

```json
{
  "id": 605816632,
  "name": "BCX",
  "description": "The Next Generation",
  "archived": false,
  "created_at": "2012-03-22T16:56:51-05:00",
  "updated_at": "2012-03-23T13:55:43-05:00",
  "creator": {
    "id": 149087659,
    "name": "Jason Fried"
  },
  "accesses": {
    "count": 5,
    "updated_at": "2012-03-23T13:55:43-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/accesses.json"
  },
  "attachments": {
    "count": 0,
    "updated_at": null,
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/attachments.json"
  },
  "calendar_events": {
    "count": 3,
    "updated_at": "2012-03-22T17:35:50-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/calendar_events.json"
  },
  "documents": {
    "count": 0,
    "updated_at": null,
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/documents.json"
  },
  "topics": {
    "count": 2,
    "updated_at": "2012-03-22T17:35:50-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/topics.json"
  },
  "todolists": {
    "remaining_count": 4,
    "completed_count": 0,
    "updated_at": "2012-03-23T12:59:23-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/todolists.json"
  }
}
```


Create project
--------------

* `POST /projects.json` will create a new project from the parameters passed.

```json
{
  "name": "This is my new project!",
  "description": "It's going to run real smooth"
}
```

This will return `201 Created`, with the location of the new project in the `Location` header along with the current JSON representation of the project if the creation was a success. See the **Get project** endpoint for more info. If the user does not have access to create new projects or the account has reached the project limit, you'll see `403 Forbidden`.


Update project
---------------

* `PUT /projects/1.json` will update the project from the parameters passed.

```json
{
  "name": "This is a new name for the project!",
  "description": "And a new description..."
}
```

This will return `200 OK` if the update was a success along with the current JSON representation of the project. See the **Get project** endpoint for more info. If the user does not have access to update the project, you'll see `403 Forbidden`.


Archiving/activating a project
------------------------------

* `PUT /projects/1.json` with the following JSON payload will archive a project (pass false to activate it again).

```json
{
  "archived": true
}
```

This will return `200 OK` if the update was a success, along with the current JSON representation of the project. If the user does not have access to update the project, you'll see `403 Forbidden`.


Delete project
-------------

* `DELETE /projects/1.json` will delete the project specified and return `204 No Content` if that was successful. If the user does not have access to delete the project, you'll see `403 Forbidden`.
Todo lists
==========

Get todolists
-------------

* `GET /projects/1/todolists.json` will return all todolists with remaining todos on them sorted by position.
* `GET /projects/1/todolists/completed.json` will return all the completed todolists.

```json
[
  {
    "id": 968316918,
    "name": "Launch list",
    "description": "What we need for launch",
    "updated_at": "2012-03-22T16:56:52-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/todolists/968316918-launch-list.json",
    "completed": false,
    "position": 1
  },
  {
    "id": 812358930,
    "name": "Version 2",
    "description": "What we will do next",
    "updated_at": "2012-03-22T16:56:52-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/todolists/812358930-version-2.json",
    "completed": false,
    "position": 1
  }
]
```

Get todolists with assigned todos
---------------------------------

* `GET /people/1/assigned_todos.json` will return all the todolists with todos assigned to the specified person.

```json
[
  {
    "id": 968316918,
    "name": "Launch list",
    "description": "What we need for launch!",
    "updated_at": "2012-03-29T11:00:39-05:00",
    "url": "http://bcx.dev/735644780/api/v1/projects/605816632-bcx/todolists/968316918-launch-list.json",
    "completed": false,
    "position": 1,
    "assigned_todos": [
      {
        "id": 223304243,
        "content": "Design it",
        "due_at": "2012-03-30",
        "comments_count": 0,
        "created_at": "2012-03-27T13:19:30-05:00",
        "updated_at": "2012-03-29T11:00:38-05:00",
        "url": "http://bcx.dev/735644780/api/v1/projects/605816632-bcx/todos/223304243-design-it.json",
        "position": 1
      }
    ]
  },
  {
    "id": 812358930,
    "name": "Version 2",
    "description": "What we will do next",
    "updated_at": "2012-03-29T10:50:33-05:00",
    "url": "http://bcx.dev/735644780/api/v1/projects/605816632-bcx/todolists/812358930-version-2.json",
    "completed": false,
    "position": 2,
    "assigned_todos": [
      {
        "id": 270524416,
        "content": "Fix all the bugs",
        "due_at": null,
        "comments_count": 0,
        "created_at": "2012-03-27T13:19:30-05:00",
        "updated_at": "2012-03-29T10:50:33-05:00",
        "url": "http://bcx.dev/735644780/api/v1/projects/605816632-bcx/todos/270524416-fix-all-the-bugs.json",
        "position": 1
      }
    ]
  }
]
```


Get todolist
------------

* `GET /projects/1/todolists/1.json` will return the specified todolist including the todos.

```json
{
  "id": 968316918,
  "name": "Launch list",
  "description": "What we need for launch!",
  "created_at": "2012-03-24T09:53:35-05:00",
  "updated_at": "2012-03-24T09:59:35-05:00",
  "completed": false,
  "position": 1,
  "todos": {
    "remaining": [
      {
        "id": 223304243,
        "content": "Design it",
        "due_at": "2012-03-24",
        "comments_count": 0,
        "created_at": "2012-03-24T09:53:35-05:00",
        "updated_at": "2012-03-24T09:55:52-05:00",
        "assignee": {
          "id": 149087659,
          "type": "Person",
          "name": "Jason Fried"
        },
        "position": 1,
        "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/todos/223304243-design-it.json"
      },
      {
        "id": 411008527,
        "content": "Test it",
        "due_at": null,
        "comments_count": 0,
        "created_at": "2012-03-24T09:53:35-05:00",
        "updated_at": "2012-03-24T09:53:35-05:00",
        "assignee": {},
        "position": 2,
        "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/todos/411008527-test-it.json"
      }
    ],
    "completed": [
      {
        "id": 1046098401,
        "content": "Think of it",
        "due_at": null,
        "comments_count": 0,
        "created_at": "2012-03-24T09:59:33-05:00",
        "updated_at": "2012-03-24T09:59:35-05:00",
        "completed_at": "2012-03-24T09:59:35-05:00",
        "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx/todos/1046098401-think-of-it.json",
        "assignee": {},
        "position": 3,
        "completer": {
          "id": 149087659,
          "name": "Jason Fried"
        }
      }
    ]
  }
}
```


Create todolist
---------------

* `POST /projects/1/todolists.json` will create a new todolist from the parameters passed.

```json
{
  "name": "My really important list of stuff to do",
  "description": "I'm serial guys, this stuff matters!"
}
```

This will return `201 Created`, with the URL of the new todolist in the `Location` header along with the current JSON representation of the todolist if the creation was a success. See the **Get todolist** endpoint for more info.


Update todolist
---------------

* `PUT /projects/1/todolists/1.json` will update the todolist from the parameters passed.

```json
{
  "name": "Think of a new title?",
  "description": "And a new description!"
}
```

This will return `200 OK` if the creation was a success along with the current JSON representation of the todolist in the response body. See the **Get todolist** endpoint for more info. If the user does not have access to update the todolist, you'll see `403 Forbidden`.

### Reordering todolists

Updating the `position` of a todolist is also possible through this endpoint by passing an integer between `1` and `n`, where `n` is the number of todolists in this project.

```json
{
  "position": 2
}
```

*Note*: If the position is out of bounds, the todo will be moved to the bottom.

Delete todolist
--------------

* `DELETE /projects/1/todolists/1.json` will delete the todolist specified and return `204 No Content` if that was successful. If the user does not have access to delete the todolist, you'll see `403 Forbidden`.
Todos
=====

Get todos
---------

To get an index of all todos on a list, see [todolists](https://github.com/37signals/bcx-api/blob/master/sections/todolists.md).


Get todo
--------

* `GET /projects/1/todos/1.json` will return the specified todo.

```json
{
  "id": 1,
  "todolist_id": 1000,
  "content": "Design it",
  "completed": false,
  "due_at": "2012-03-27",
  "created_at": "2012-03-24T09:53:35-05:00",
  "updated_at": "2012-03-24T10:56:33-05:00",
  "position": 1,
  "assignee": {
    "id": 149087659,
    "type": "Person",
    "name": "Jason Fried"
  },
  "comments": [
    {
      "id": 1028592764,
      "content": "+1",
      "created_at": "2012-03-24T09:53:34-05:00",
      "updated_at": "2012-03-24T09:53:34-05:00",
      "creator": {
        "id": 127326141,
        "name": "David Heinemeier Hansson"
      }
    }
  ]
}
```


Create todo
-----------

* `POST /projects/1/todolists/1/todos.json` will add a new todo to the specified todolist from the parameters passed. The `due_at` parameter should be in ISO 8601 format (like "2012-03-27T16:00:00-05:00"). The assignee parameters need an `type` field with either `Person` or `Group` specified. The `id` is then the id of that person or group.

```json
{
  "content": "This is my new thing!",
  "due_at": "2012-03-27",
  "assignee": {
    "id": 149087659,
    "type": "Person"
  }
}
```

This will return `201 Created`, with the URL of the new todo in the `Location` header along with the current JSON representation of the todo if the creation was a success. See the **Get todo** endpoint for more info. If the assignee type is unrecognized or the `due_at` is in a wrong format, you'll see a `400 Bad Request`.


Update todo
-----------

* `PUT /projects/1/todos/1.json` will update the todo from the parameters passed. The `completed` field can be set to either `true` or `false` to check or uncheck the todo.

```json
{
  "content": "New content thing!",
  "due_at": "2012-03-27",
  "assignee": {
    "id": 149087659,
    "type": "Person"
  },
  "completed": true
}
```

This will return `200 OK` if the update was a success along with the current JSON representation of the todo in the response body. See the **Get todo** endpoint for more info. If the assignee type is unrecognized or the `due_at` is in a wrong format, you'll see a `400 Bad Request`.

### Reordering todos

Updating the `position` of a todo is also possible through this endpoint by passing an integer between `1` and `n`, where `n` is the number of todos in this list.

```json
{
  "position": 2
}
```

*Note*: If the position is out of bounds, the todo will be moved to the bottom.


Delete todo
----------

* `DELETE /projects/1/todos/1.json` will delete the todo specified and return `204 No Content` if that was successful. If the user does not have access to delete the todo, you'll see `403 Forbidden`.
Topics
========

> <Clever topics quote>

Topics are anything in Basecamp that can have comments: Messages, Calendar Events, Uploads, Todos. Messages will appear in the topics index even before they have any comments, but any other topicable type will only appear if there's at least one comment posted.


Get topics
----------

* `GET /projects/1/topics.json` will return the all the topics for the project. We will return 50 topics per page. If the result set has 50 topics, it's your responsibility to check the next page to see if there are any more topics -- you do this by adding `&page=2` to the query, then `&page=3` and so on.

```json
[
  {
    "id": 1028592764,
    "title": "Prep the materials before the board meeting with Bezos",
    "excerpt": "I'll be there!",
    "created_at": "2012-03-24T09:53:35-05:00",
    "updated_at": "2012-03-24T09:53:35-05:00",
    "attachments": 0,
    "last_updater": {
      "id": 149087659,
      "name": "Jason Fried"
    },
    "topicable": {
      "id": 174886926,
      "type": "CalendarEvent",
      "url": "https://basecamp.com/999999999//api/v1/projects/605816632-bcx/calendar_events/174886926-prep-the-materials.json"
    }
  },
  {
    "id": 936075699,
    "title": "Welcome!",
    "excerpt": "Yeah, really, welcome!",
    "created_at": "2012-03-24T09:53:35-05:00",
    "updated_at": "2012-03-24T09:53:35-05:00",
    "attachments": 1,
    "last_updater": {
      "id": 149087659,
      "name": "Jason Fried"
    },
    "topicable": {
      "id": 936075699,
      "type": "Message",
      "url": "https://basecamp.com/999999999//api/v1/projects/605816632-bcx/messages/936075699-welcome.json"
    }
  }
]
```

The `title` is the original title of the topicable and the `excerpt` is from the latest comment. If a message does not have any comments, the `last_updater` is the creator of the message.Uploads
=======

Uploads show up as "Files" inside of Basecamp.


Create uploads
--------------

* `POST /projects/1/uploads.json` will create a new entry in the "Files" section on the given project, with the given attachment token.

This endpoint will return a `201 Created` if successful, with the URL to the new uplaod in the `Location` header along with the current JSON representation of the upload. See the **Get upload** for more info.

Attaching files requires both the token and the name of the attachment. The
token is returned from the [Create attachments](https://github.com/37signals/bcx-api/blob/master/sections/attachments.md)
endpoint, which you must hit first before creating an upload.

The `name` parameter *must* be a valid filename with an extension. Only one
attachment is allowed.

```json
{
  "content": "Here's the new logo!",
  "attachments": [
    {
      "token": "4f71ea23-134660425d1818169ecfdbaa43cfc07f4e33ef4c",
      "name": "new_logo.png"
    }
  ]
}
```

*Note*: Uploads can only have one attachment, despite the json blob accepting plural `attachments`. This is for consistency across the other endpoints that accept attachments. Hit the endpoint multiple times if you need to create multiple uploads.


Get upload
----------

* `GET /projects/1/uploads/2.json` will show the content, comments, and attachments for this upload.

Each attachment blob includes the `url` parameter, which you can make a
`GET` request (with authentication) in order to directly download the attachment.

```json
{
  "id": 31709,
  "created_at": "2012-03-27T22:48:49-04:00",
  "updated_at": "2012-03-28T11:36:10-04:00",
  "content": "Hi there!",
  "attachments": [
    {
      "key": "40b8a84cb1a30dbe04457dc99e094b6299deea41",
      "name": "bearwave.gif",
      "byte_size": 508254,
      "content_type": "image/gif",
      "created_at": "2012-03-27T22:48:49-04:00",
      "url": "https://basecamp.com/1111/api/v1/projects/2222/attachments/3333/40b8a84cb1a30dbe04457dc99e094b6299deea41/original/bearwave.gif",
      "creator": {
        "id": 73,
        "name": "Nick Quaranto"
      }
    }
  ],
  "comments": [
    {
      "id": 5566323,
      "content": "Testing a comment",
      "created_at": "2012-03-28T11:36:10-04:00",
      "updated_at": "2012-03-28T11:36:10-04:00",
      "attachments": [],
      "creator": {
        "id": 73,
        "name": "Nick Quaranto"
      }
    }
  ]
}
```
