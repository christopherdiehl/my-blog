---
title: "Create Filter Bar React Redux Reselect"
date: 2018-03-13T07:49:00-04:00
draft: false
---

# Intro
I recently created a filter bar using the popular [Reselect](https://github.com/reactjs/reselect) and [Redux](https://github.com/reactjs/redux) libraries in React. For this post, we're going to create a filter bar in a standard contacts page, where the contacts will be filtered by individual's name.

Here is an example of how the contacts page will look with the filter. 

![Contact Filter](/contact-page.png)

## Reducer Setup
First we need to create the reducer and the initial state of our application, let's use [Immutable.js](https://facebook.github.io/immutable-js/) to ensure our state is..... immutable. Our reducer should look something like this.

```
import { UPDATE_SEARCH_KEY } from './constants';
import { fromJS } from 'immutable';

const initialState = fromJS({
  contacts: [],
  searchKey: '',
});

function ContactsReducer(state = initialState, action) {
  switch (action.type) {
    case UPDATE_SEARCH_KEY:
      return state.set('searchKey', action.data);
    default:
      return state;
  }
}

export default ContactsReducer;
```

## Action Setup

Now time to wire up the function that will trigger the UPDATE_SEARCH_KEY case in the reducer. 
Something like this should work.
```
export function updateSearchKey(key) {
  return {
    type: UPDATE_SEARCH_KEY,
    data: key,
  };
}
```

## Selectors

This is where the real magic of the filtering happens. By using reselect the contacts are only filtered when the search key or the contacts are modified. Which is amazing because we don't have to worry about the filtering function happening when the component updates and re-renders.

Selectors file should look something like this:
```
import { createSelector } from 'reselect';

/**
 * Direct selector to the contacts state domain
 */
const selectContactsDomain = (state) => state.get('contacts');

const getKeyword = (state) => state.get('contacts').get('searchKey')


const selectFilteredContacts = () => createSelector(
  [selectContactsDomain, getKeyword],
  (contactDomain, keyword) => contactDomain.get('contacts').filter((contact) => contact.name.toLowerCase().includes(keyword.toLowerCase()))
);

export {
  selectContactsDomain,
  selectFilteredContacts,
};
```

