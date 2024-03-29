---
layout: default
title: Notes on Siren
nav_order: 11
---

# Notes on Siren

We use [Siren Hypermedia Format](https://github.com/kevinswiber/siren) for exchange. We like it a lot but have found some limitations which needed us to deviate from the standard.

## Action parameters

Siren [Actions](https://github.com/kevinswiber/siren#actions-1) uses [HTML5 input types](https://github.com/kevinswiber/siren#fields-1) to specify parameters for actions.
We wanted to have a richer experience and decided to use [JSON schema](https://json-schema.org/) to hand us information about the parameters of an action.

To stay close to the standard we:

- `fields` contain exactly one field and the `name` of that field is set to the parameter object name.
- Add a URL to the actions schema in the `class` of a field. The server is expected to deliver the JSON schema when this link is resolved.
- `type` is set to `"application/json"` to indicate we expect a JSON.
- `value` may contain prefilled values as a JSON object

Example:

```json
{
  "class": [
    "CustomersRoot"
  ],
  "title": "The Customers API",
  "properties": {},
  "entities": [],
  "actions": [
    {
      "name": "CreateCustomer",
      "title": "Request creation of a new Customer.",
      "method": "POST",
      "href": "http://localhost:5000/Customers/CreateCustomer",
      "type": "application/json",
      "class": [
        "ParameterAction"
      ],
      "fields": [
        {
          "name": "CreateCustomerParameters",
          "type": "application/json",
          "class": [
            "http://localhost:5000/ActionParameterTypes/CreateCustomerParameters"
          ]
        }
      ]
    },
  ],
  ...
}
```

An acceptable JSON parameter (send as array of objects to stay close to the specification) for the action looks like this:

```json
[{"CreateCustomerParameters": 
  {
    "Name":"Hans Schmid"
  }
}]
```

Also we accept a short form without the container array:

```json
{
  "CreateCustomerParameters":   
  {
    "Name":"Hans Schmid"
  }
}
```

## File upload

File upload requires the server to present some information to the clients.

We send the following structure:

- `type` of the action is `multipart/form-data` to indicate how to send the data
- `fields` contain the information for the upload
- `type` is `file` to stay close to the siren standard using HTML5 input types
- `accept`: a comma separated list of file type specifiers. See: [MDN: Unique file type specifiers](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/file#unique_file_type_specifiers)
- `maxFileSizeBytes`: max size of the upload
- `allowMultiple`: Is it allowed to send multiple files

Example:

```json
{
  ...
  "actions": [
    {
      "name": "Upload",
      "title": "Uploads data to the system",
      "method": "POST",
      "href": "https://myhost.com/api/Data/UploadData",
      "type": "multipart/form-data",
      "class": [
        "FileUploadAction"
      ],
      "fields": [
        {
          "name": "UploadFiles",
          "type": "file",
          "accept": ".png, image/*'",
          "maxFileSizeBytes": 209715200,
          "allowMultiple": false
        }
      ]
    },
```
