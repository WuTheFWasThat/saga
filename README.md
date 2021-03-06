# THIS IS A WIP.  DONT USE IT YET

# TODO
- write tests
- make run effect?
- make sagas pause/resumable? (generalizing cancelable)


# saga

[![Join the chat at https://gitter.im/wuthefwasthat/saga](https://badges.gitter.im/wuthefwasthat/saga.svg)](https://gitter.im/wuthefwasthat/saga?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge) [![npm version](https://img.shields.io/npm/v/redux-saga.svg?style=flat-square)](https://www.npmjs.com/package/redux-saga)

`saga` is a library that aims to make side effects (i.e. asynchronous things like data fetching and impure things like accessing the browser cache) easier and better.

The mental model is that a saga is like a separate thread in your application that's solely responsible for side effects.

It uses an ES6 feature called Generators to make those asynchronous flows easy to read, write and test. *(if you're not familiar with them [here are some introductory links](https://yelouafi.github.io/redux-saga/docs/ExternalResources.html))* By doing so, these asynchronous flows look like your standard synchronous JavaScript code. (kind of like `async`/`await`, but generators have a few more awesome features we need)

# Getting started

## Install

```sh
$ npm install --save saga-js
```

Alternatively, you may use the provided UMD builds directly in the `<script>` tag of an HTML page. See [this section](#using-umd-build-in-the-browser).

## Usage Example

Suppose we have an UI to fetch some user data from a remote server when a button is clicked. (For brevity, we'll just show the action triggering code.)

```javascript
class UserComponent extends React.Component {
  ...
  onSomeButtonClicked() {
    const { userId, dispatch } = this.props
    dispatch({type: 'USER_FETCH_REQUESTED', payload: {userId}})
  }
  ...
}
```

The Component dispatches a plain Object action to the Store. We'll create a Saga that watches for all `USER_FETCH_REQUESTED` actions and triggers an API call to fetch the user data.

#### `sagas.js`

```javascript
import { takeEvery, takeLatest } from 'saga'
import { call, put } from 'saga/effects'
import Api from '...'

// worker Saga: will be fired on USER_FETCH_REQUESTED actions
function* fetchUser(action) {
   try {
      const user = yield call(Api.fetchUser, action.payload.userId);
      yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "USER_FETCH_FAILED", message: e.message});
   }
}

/*
  Starts fetchUser on each dispatched `USER_FETCH_REQUESTED` action.
  Allows concurrent fetches of user.
*/
function* mySaga() {
  yield* takeEvery("USER_FETCH_REQUESTED", fetchUser);
}

/*
  Alternatively you may use takeLatest.

  Does not allow concurrent fetches of user. If "USER_FETCH_REQUESTED" gets
  dispatched while a fetch is already pending, that pending fetch is cancelled
  and only the latest one will be run.
*/
function* mySaga() {
  yield* takeLatest("USER_FETCH_REQUESTED", fetchUser);
}

export default mySaga;
```

# Documentation

- [Introduction](http://yelouafi.github.io/redux-saga/docs/introduction/BeginnerTutorial.html)
- [Basic Concepts](http://yelouafi.github.io/redux-saga/docs/basics/index.html)
- [Advanced Concepts](http://yelouafi.github.io/redux-saga/docs/advanced/index.html)
- [Recipes](http://yelouafi.github.io/redux-saga/docs/recipes/index.html)
- [External Resources](http://yelouafi.github.io/redux-saga/docs/ExternalResources.html)
- [Troubleshooting](http://yelouafi.github.io/redux-saga/docs/Troubleshooting.html)
- [Glossary](http://yelouafi.github.io/redux-saga/docs/Glossary.html)
- [API Reference](http://yelouafi.github.io/redux-saga/docs/api/index.html)

# Using umd build in the browser

There is also a **umd** build of `saga` available in the `dist/` folder. When using the umd build `saga` is available as `Saga` in the window object.

The umd version is useful if you don't use Webpack or Browserify. You can access it directly from [unpkg](unpkg.com).

The following builds are available:

- [https://unpkg.com/redux-saga/dist/redux-saga.js](https://unpkg.com/redux-saga/dist/redux-saga.js)
- [https://unpkg.com/redux-saga/dist/redux-saga.min.js](https://unpkg.com/redux-saga/dist/redux-saga.min.js)

**Important!** If the browser you are targeting doesn't support *ES2015 generators*, you must provide a valid polyfill, such as [the one provided by `babel`](https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.25/browser-polyfill.min.js). The polyfill must be imported before **saga**:
