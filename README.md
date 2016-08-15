# Meteor Style Guide
Meteor Style Guide with best practices (in progress)

## Table of Contents

1. [Settings File](#settings file)
2. [Methods](#methods)

## Settings File

- Always create at the root of your application 2 files: settings-development.json and settings-production.json to store API keys and other infos. Ignore these files in your version control !

````
{
  "public": {
    "publicInfo": "This is accessible on client"
  },
  "private": {
    "privateInfo": "This is accessible on server"
  }
}
````

In your app, retrieve data with ``Meteor.settings.public.publicInfo``.

Launch your meteor app with ``meteor --settings settings.json``.

## Methods

- Always remove the insecure package with ``meteor remove insecure`` and deny all the client-side updates:

````
// imports/api/tasks/tasks.js

import { Mongo } from 'meteor/mongo';

export const Tasks = new Mongo.Collection('Tasks');

Tasks.allow({
  insert: () => false,
  update: () => false,
  remove: () => false,
});

Tasks.deny({
  insert: () => true,
  update: () => true,
  remove: () => true,
});
````
- Use the validated-method package (in the example it works with the simple-schema package, but you can make it work with check too; have a look on the github repo !)

````
// imports/api/tasks/methods.js

import { Meteor } from 'meteor/meteor';
import { Tasks } from './tasks';
import { SimpleSchema } from 'meteor/aldeed:simple-schema';
import { ValidatedMethod } from 'meteor/mdg:validated-method';

export const insertTask = new ValidatedMethod({
  name: 'tasks.insert',
  validate: new SimpleSchema({
    title: { type: String },
  }).validator(),
  run(task) {
    if (!this.userId) {
      throw new Meteor.Error('unauthorized', 'You must be logged in to add a new task');
    }
    Tasks.insert(task);
  },
});
````

````
// imports/ui/components/add-task.js

import React from 'react';
import { FormGroup, FormControl } from 'react-bootstrap';
import { insertTask } from '../../api/tasks/methods.js';

const handleInsertTask = (event) => {
  const target = event.target;
  const title = target.value.trim();

  if (title !== '' && event.keyCode === 13) {
    insertTask.call({
      title,
    }, (error) => {
      if (error) {
        console.log(error.reason);
      } else {
        target.value = '';
        console.log('Task added!');
      }
    });
  }
};

export const AddTask = () => (
  <FormGroup>
    <FormControl
      type="text"
      onKeyUp={ handleInsertTask }
      placeholder="Type a task title and press enter"
    />
  </FormGroup>
);
````
````
// import/startup/server/api.js

import '../../api/tasks/methods.js';
````
````
// import/startup/server/index.js

import './api';
````
````
// server/main.js

import '/imports/startup/server';
````
