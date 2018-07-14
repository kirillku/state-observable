# [wip] state-observable
If you want to use `redux-observable` without `redux`


```javascript
import * as React from 'react'
import { ajax } from 'rxjs/ajax'
import { takeUntil, mergeMap, map } from 'rxjs/operators'

import { combineEpics, withStateObservable, ofType } from 'state-observable'

class SomeComponent extends React.Component {
  componentDidMount() {
    this.props.loadUsers()
  }

  componentWillUnmount() {
    this.props.loadUsersCancel()
  }

  render() {
    const { status, users } = this.props
    return (
      <div>
        {status === 'SUCCESS' && users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </div>
    )
  }
}

const FETCH_USERS = 'FETCH_USERS'
const FETCH_USERS_CANCEL = 'FETCH_USERS_CANCEL'

const loadUsersEpic = action$ => action$.pipe(
  ofType(FETCH_USERS),
  mergeMap(action => ajax.getJSON(`/api/users`).pipe(
    map(users => ({ status: 'SUCCESS', users }))
    takeUntil(action$.pipe(
      ofType(FETCH_USERS_CANCEL)
    ))
  ))
)

const epic = combineEpics(loadUsersEpic, ...someOtherEpics)

const mapProps = (props, state, dispatch) => ({
  loadUsers: () => dispatch({ type: FETCH_USERS }),
  loadUsersCancel: () => dispatch({ type: FETCH_USERS_CANCEL })
  users: state.users
})

export default withStateObservable(epic, mapProps)(SomeComponent)
```
