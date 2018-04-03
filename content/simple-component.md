---
title: "Build a simple component"
---

# Build a simple component

We’ll build our UI following a [Component-Driven Development](https://blog.hichroma.com/component-driven-development-ce1109d56c8e) (CDD) methodology. It’s a process that builds UIs from the “bottom up” starting with components and ending with screens. CDD helps you scale the amount of complexity you’re faced with as you build out the UI.

## Task

![Task component in three states](/task-states-learnstorybook.png)

`Task` is the core component in our app. Each task displays slightly differently depending on exactly what state it’s in. We display a checked (or unchecked) checkbox, some information about the task (and a link out), and a “pin” button, allowing us move tasks up and down the list. Putting this together, we’ll need these props:

* `title` – a string describing the task
* `subtitle` – a string describing the context of the task in the external system
* `url` – a link to the external resource (clicking on the task will take us there).
* `state` - which list is the task currently in and is it checked off?

As we start to build `Task`, we first write our test states that correspond to the different types of tasks sketch above. Then we use Storybook to build the component in isolation using mocked data. We’ll “visual test” the component’s appearance given each state as we go.

This process is similar to [Test-driven development](https://en.wikipedia.org/wiki/Test-driven_development) (TDD) that we can call “[Visual TDD](https://blog.hichroma.com/visual-test-driven-development-aec1c98bed87)”.

## Get setup

First, let’s create the task component and its accompanying story file: `src/components/Task.js` and `src/components/Task.story.js`.

We’ll begin with a basic implementation of the `Task`, simply taking in the attributes we know we’ll need and the two actions you can take on a task (to move it between lists):

```javascript
import React from 'react';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  return (
    <div className="list-item">
      <input type="text" value={title} readOnly={true} />
    </div>
  );
}
```

Above we we render straightforward markup for `Task` based on the existing HTML structure of the Todos app.

Below we build out task’s three test states in the story file:

```javascript
import React from 'react';
import { storiesOf } from '@storybook/react';
import { action } from '@storybook/addon-actions';

import Task from './Task';

export function createTask(attrs) {
  return {
    id: Math.round(Math.random() * 1000000).toString(),
    title: 'Test Task',
    state: 'TASK_INBOX',
    updatedAt: Date.now(),
    ...attrs,
  };
}

export const actions = {
  onPinTask: action('onPinTask'),
  onArchiveTask: action('onArchiveTask'),
};

storiesOf('Task', module)
  .add('default', () => <Task task={createTask({ state: 'TASK_INBOX' })} {...actions} />)
  .add('pinned', () => <Task task={createTask({ state: 'TASK_PINNED' })} {...actions} />)
  .add('archived', () => <Task task={createTask({ state: 'TASK_ARCHIVED' })} {...actions} />);
```

There are two basic levels of organization in Storybook. The component and it’s child stories. Think of each story as a permutation of a component. You can have as many stories per component as you need.

| **Component** |
| ------------- |
| Story         |
| Story         |
| Story         |

To initiate Storybook we first call the `storiesOf()` function to register the component. We add a display name for the component –the name that appears on the sidebar in the Storybook app.

`action()` allows us to create a callback that appears in the _actions_ panel of the Storybook UI when clicked. So when we build a pin button, we’ll be able to determine in the test UI if a button click is successful.

As we need to pass the same set of actions to all permutations of our component, it is convenient to bundle them up into a single `actions` variable and use React's `{...actions}` props expansion to pass them all at once. (`<Task {..actions}>` is equivalent to `<Task onPinTask={actions.onPinTask} onArchiveTask={actions.onArchiveTask}>`).

Another nice thing about bundling the `actions` that a component needs is that you can `export` them and use them in later stories, as we'll see later.

To define our stories, we call `add()` once for each of our test states to generate a story. The action story is a function that returns a rendered element (i.e. a component class with a set of props) in a given state---exactly like a React [Stateless Functional Component](https://reactjs.org/docs/components-and-props.html).

When creating a story we use a helper function (`createTask()`) to build out the shape of the task the component expects. This is typically modelled off what the true data looks like. Again, `export`-ing this function will enable us to reuse it in later stories, as we'll see.

<div class="aside">
<b>Actions</b> help you verify interactions when building UI components in isolation. Oftentimes you won't have access to the functions and state you have in context of the app. Use `actions()` to stub them in. 
</div>

## Config

We also have to make one small change to the Storybook setup so it notices our `.story.js` files and uses our CSS file. By default Storybook looks for stories in a `/stories` directory; this tutorial uses a naming scheme that mirrors the `.test.js` naming scheme favoured by CRA for automated tests.

```javascript
import { configure } from '@storybooks/storybook';
import '../src/index.css';

const req = require.context('../src', true, /.story.js$/);

function loadStories() {
  req.keys().forEach(filename => req(filename));
}

configure(loadStories, module);
```

Once we’ve done this, restarting the Storybook server should yield test cases for the three Task states:

<video autoPlay muted playsInline controls >
  <source
    src="/inprogress-task-states.mp4"
    type="video/mp4"
  />
</video>

## Build out the states

Now we have Storybook setup, styles imported, and test cases built out, we can quickly start the work of implementing the HTML of the component to match the design.

The component is still basic at the moment. First write the code that achieves the design without going into too much detail:

```javascript
import React from 'react';

const alignStyles = {
  fontSize: '14px',
  lineHeight: '1.5rem',
  padding: '0.75em 0.25em',
};

export default function Task({
  task: { id, title, url, state, subtitle },
  onSnoozeTask,
  onPinTask,
}) {
  return (
    <div className="list-item">
      <label className="checkbox">
        <input
          type="checkbox"
          defaultChecked={state === 'TASK_ARCHIVED'}
          disabled={true}
          name="checked"
        />
        <span className="checkbox-custom" />
      </label>
      <input
        type="text"
        value={title}
        readOnly={true}
        placeholder="Input title"
        onClick={() => open(url, '_new')}
      />
      {subtitle && <p style={{ flex: 1, color: '#666', ...alignStyles }}>{subtitle}</p>}
      {state !== 'TASK_SNOOZED' &&
        state !== 'TASK_ARCHIVED' && (
          <a style={alignStyles} onClick={() => onSnoozeTask(id)}>
            <span className="icon-link icon-sync" />
          </a>
        )}
      {state !== 'TASK_PINNED' && (
        <a style={alignStyles} onClick={() => onPinTask(id)}>
          <span className="icon-link icon-arrow-down" />
        </a>
      )}
    </div>
  );
}
```

The additional markup from above combined with the CSS we imported earlier yields the following UI:

<video autoPlay muted playsInline loop>
  <source
    src="/finished-task-states.mp4"
    type="video/mp4"
  />
</video>

## Specify data requirements

It’s best practice to use `propTypes` in React to specify the shape of data that a component expects. Not only is it self documenting, it also helps catch problems early.

```javascript
import React from 'react';
import PropTypes from 'prop-types';

function Task() {
  ...
}

Task.propTypes = {
  task: PropTypes.shape({
    id: PropTypes.number.isRequired,
    title: PropTypes.string.isRequired,
    subtitle: PropTypes.string.isRequired,
    url: PropTypes.string.isRequired,
    state: PropTypes.string.isRequired,
  }),
  onPinTask: PropTypes.func,
};

export default Task;
```

Now a warning in development will appear if the Task component is misused.

<div class="aside">
An alternative way to achieve the same purpose is to use a JavaScript type system like TypeScript to create a type for the component properties.
</div>

## Component built!

We’ve now successfully built out a component without needing a server or running the entire frontend application. The next step is to build out the remaining Taskbox components one by one in a similar fashion.

As you can see, getting started building components in in isolation is easy and fast. We can expect to produce a higher-quality UI with less bugs and more polish because it’s possible to dig in and test every possible state.

## Automated Testing

Storybook gave us a great way to visually test our application during construction. The ‘stories’ will help ensure we don’t break our Task visually as we continue to develop the app. However, it is a completely manual process at this stage, and someone has to go to the effort of clicking through each test state and ensuring it renders well and without errors or warnings. Can’t we do that automatically?

### Snapshot testing

Snapshot testing refers to the practice of recording the “known good” output of a component for a given input and then flagging the component whenever the output changes in future. This complements Storybook, because it’s a quick way to view the new version of a component and check out the changes.

With the [Storyshots addon](https://github.com/storybooks/storybook/tree/master/addons/storyshots) a snapshot test is created for each of the stories. Use it by adding a development dependency on the package:

```bash
yarn add --dev @storybook/addon-storyshots react-test-renderer
```

Then create an `src/storybook.test.js` file with the following in it:

```javascript
import initStoryshots from '@storybook/addon-storyshots';
initStoryshots();
```

Once the above is done, we can run yarn test and see the following output:

[image testrunner]

We now have a snapshot test for each of our `Task` stories. If we change the implementation of `Task`, we’ll be prompted to verify the changes.