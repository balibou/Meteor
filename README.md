# meteor Style Guide
All best practices for Meteor (in progress)

## Table of Contents

1. [Settings File](#settings file)

## Settings File

- Always create at the root of your application 2 files: settings-development.json and settings-production.json to store API keys and other infos. Ignore these files in your version control !

````{
  "public": {
    "publicInfo": "This is accessible on client"
  },
  "private": {
    "privateInfo": "This is accessible on server"
  }
}````

In your app, retrieve data with ``Meteor.settings.public.publicInfo``.

Launch your meteor app with ``meteor --settings settings.json``.
