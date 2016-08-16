# Meteor Style Guide
Meteor Style Guide with best practices (in progress)

## Table of Contents

1. [Settings File](#settings file)
2. [Methods](#methods)
3. [Publications / Subscriptions](#publications / subscriptions)

## Settings File

- Always create at the root of your application 2 files: settings-development.json and settings-production.json to store API keys and other infos. Ignore these files in your version control !

````json
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

````js
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

````js
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

````js
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
````js
// import/startup/server/api.js
import '../../api/tasks/methods.js';
````
````js
// import/startup/server/index.js
import './api';
````
````js
// server/main.js
import '/imports/startup/server';
````

## Publications / Subscriptions

- Always remove the autopublish package with ``meteor remove autopublish``:

````js
// imports/api/tasks/server/publications.js
import { Meteor } from 'meteor/meteor';
import { Tasks } from '../tasks';

Meteor.publish('tasks', () => Tasks.find());
````

````js
// imports/startup/server/api.js
import '../../api/tasks/server/publications.js';
````

- Use react-komposer package with ``npm i --save react-komposer`` to create container:

````js
// imports/ui/containers/tasks-list.js
import { composeWithTracker } from 'react-komposer';
import { Tasks } from '../../api/tasks/tasks.js';
import { TasksList } from '../components/tasks-list.js';
import { Loading } from '../components/loading.js';
import { Meteor } from 'meteor/meteor';

const composer = (params, onData) => {
  const subscription = Meteor.subscribe('tasks');
  if (subscription.ready()) {
    const tasks = Tasks.find().fetch();
    onData(null, { tasks });
  }
};

export default composeWithTracker(composer, Loading)(TasksList);
````

````js
// imports/ui/pages/tasks.js

import React from 'react';
import { Row, Col } from 'react-bootstrap';
import TasksList from '../containers/tasks-list.js';

export const Tasks = () => (
  <Row>
    <Col xs={ 12 }>
      <TasksList />
    </Col>
  </Row>
);
````
````js
// imports/ui/components/task.js

import React from 'react';
import { Row, Col, ListGroupItem, FormControl, Button } from 'react-bootstrap';

export const Task = ({ task }) => (
  <ListGroupItem key={ task._id }>
    <Row>
      <Col xs={ 8 } sm={ 10 }>
        <FormControl
          type="text"
          defaultValue={ task.title }
        />
      </Col>
    </Row>
  </ListGroupItem>
);
````

````js
// imports/ui/components/tasks-list.js

import React from 'react';
import { ListGroup, Alert } from 'react-bootstrap';
import { Task } from './task.js';

export const TasksList = ({ tasks }) => (
  tasks.length > 0 ? <ListGroup className="tasks-list">
    {tasks.map((tas) => (
      <Task key={ tas._id } task={ tas } />
    ))}
  </ListGroup> :
  <Alert bsStyle="warning">No task yet.</Alert>
);

TasksList.propTypes = {
  tasks: React.PropTypes.array,
};
````
