# react-aria-announcer

This lightweight package allows you to broadcast `aria-live` messages to screen-readers from anywhere within your React application.

This package was created to try and solve the problems that I faced when building an online accessible editor. Historically, online editors are known for being not very accessible. Using this package I was able to broadcast announcements such as `"text block had been added"` or an `"image block had been deleted"`, which normally would not have been detected by a screenreader, but would have been visible on the screen.

Unlike other packages that also solve this issue, `react-aria-announce` allows you to pre-define acceptable aria-live announcements. It also, takes advantage of HOCs, allowing for cleaner, more readable code.

Using ``react-aria-announcer`` you can broadcast these important announcements from any component in your application.

## Installation
_______________

```
npm install react-aria-announce
```
or
```
yarn add react-aria-announce
```

## Usage
_________

The library exports 3 components.

#### HiddenMessages
This component contains your aria-live div. 

```javascript
import React, { Component } from 'react';
import { HiddenMessages } from 'react-hoc-aria-live';

class MyAccessibleApp extends Component {
  state = {
    message: '',
    timeStamp: '',
  }

  render() {
    return (
      <HiddenMessages 
        message={this.state.message} 
        timeStamp={this.state.timeStamp} 
      />
    )
  }
}
```

The props:
* `message` - required (must receive the state value)
* `timeStamp` - required (must receive the state value)
* `manners` - optional values: 'assertive' or 'polite' (default: `'polite'`)

#### connectAriaAnnouncementProvider
This higher order component receives your app or a section of your app, where any number of it's ancestors might want to make an announcement. 

This new component must have access to the setState method of the component that holds the state, message and timestamp.

```javascript
import React, { Component } from 'react';
import { HiddenMessages, connectAriaAnnouncementProvider } from 'react-hoc-aria-live';
import ariaMessages from './ariaMessages';

function SearchableDashboard(props) {
  return (
    <div>
      ...lots of components within here!
    </div>
  )
}

const AccessibleSearchableDashboard = connectAriaAnnouncementProvider(SearchableDashboard)

class MyAccessibleApp extends Component {
  constructor(props) {
    super(props)
    this.state = {
      message: '',
      timeStamp: '',
    }
  }

  render() {
    return (
      <div>
      <HiddenMessages 
        message={this.state.message} 
        timeStamp={this.state.timeStamp} 
        manners='assertive'
      />
      <AccessibleSearchableDashboard
        announce={(message, timeStamp) => this.setState({ message, timeStamp })} 
        ariaMessages={ariaMessages}
        {...props}
      />
      </div>
    )
  }
}
```
The new component must receive 2 props:
* `announce` - (message, timeStamp) => this.setState({ message, timeStamp });
* `ariaMessages` - {object}
  - keys: actions e.g. addBlock
  - values: {functions} - can take as many arguments as needed, that `return a string`
e.g.

##### ariaMessages:
```javascript
const ariaMessages = {
  onToggleButton: () => 'Press shift enter to open toolbar, and navigate using tab ',
  addBlock: type => `Press shift enter to add a new ${type} block`,
  blockAdded: type => `New ${type} block added`,
  blockDeleted: type => `${type} block deleted`,
  moveBlock: (type, position) => `${type} block has moved to ${position}`,
  entered: (type) => `You are now in the ${type} block`,
  filter: (type, resultNumber) => `Now viewing ${resultNumber} of ${type} blocks`
};

export default ariaMessages;
```

### connectAriaAnnouncementConsumer
A component that contains important visual changes, you would pass to this higher order component.

```javascript
import React from 'react';
import { HiddenMessages, connectAriaAnnouncementConsumer } from 'react-hoc-aria-live';

function FilterByBlockButton({ announce, type }) {
  function handleClick() {
    fnFiltersDataBy(type)
      .then((res) => {
        announce('filter', type, res.length)
      })
    
  }

  return (
    <button onClick={() => handleClick()}>
      filter by {type}
    </button>
  )
}

const AccessibleSearchableDashboard = connectAriaAnnouncementConsumer(FilterByBlockButton)

function MyAccessibleAppSectionChild() {
  return (
    <div>
      other components
      <AccessibleSearchableDashboard />
      even more components
    </div>
  )
}

```
Your component will receive a prop `announce`.
* `announce` - {function} - the first argument will be a key, as defined in your ariaMessages. Any additional arguments will correspond to the keys value and any parameters that you defined that function to take.
