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

If you notice, the actual filtering is being performed in this line:
`(contactDomain, keyword) => contactDomain.get('contacts').filter((contact) => contact.name.toLowerCase.includes(keyword.toLowerCase()))`

There really isn't any magic happening, just a simple `filter` function and comparing the contact's name to the keyword. Please note the above filter is not case sensitive, to make it case sensitive, just remove the `.toLowerCase()`

## Component

Now we just need to wire up the component. The finished product, barring imports should look something like below:

```

export class Contacts extends React.Component { // eslint-disable-line react/prefer-stateless-function
    render() {
        return (
            <Paper style={{ minHeight: '70vh', padding: '15px 30px 30px 30px' }}>
                <Toolbar style={{ background: "white", width: '100%' }}>
                    <ToolbarGroup >
                        <h3>Contacts</h3>
                    </ToolbarGroup>
                    <ToolbarGroup>
                        <TextField
                            floatingLabelText="Search"
                            id="text-field-controlled"
                            value={this.props.contacts.searchKey}
                            onChange={this.props.updateSearchKey}
                        />
                        <RaisedButton
                            label="Add"
                            primary={true}
                            onClick={(user) => this.setState({ selectedUser: { 'name': '', notes: '', 'email': '', phoneNumber: '', location: '' } })}
                        />
                    </ToolbarGroup>
                </Toolbar>
                <hr />
                <ContactList
                    contacts={this.props.filteredContacts}
                    onClick={(user) => this.setState({ selectedUser: user })}
                />
            </Paper>
        );
    }
}

Contacts.propTypes = {
  dispatch: PropTypes.func.isRequired,
  filteredContacts: PropTypes.array.isRequired,
};

const mapStateToProps = createStructuredSelector({
  filteredContacts: selectFilteredContacts(),
});

function mapDispatchToProps(dispatch) {
  return {
    dispatch,
    updateSearchKey: (e) => dispatch(updateSearchKey(e.target.value)),
  };
}
```

## Voila, enjoy.