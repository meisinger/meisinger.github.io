---
layout: single
title:  "Bending Hooks to Your Will"
date:   2020-03-04 18:14:28 -0500
read_time: true
categories: [Development]
tags: [React, React-Native]
excerpt_separator: <!--nice-->
---
So the `Hook` revolution is upon us and we have learned a little bit about `useState` and `useEffect`.
With these two hooks alone we can get a lot of stuff done.<!--nice-->
Sure, there are a few other hooks that we should learn about like the `useReducer`, `useContext` and `useMemo` but we
can get to those in the near future.

Right now, with just the `useState` and `useEffect` we can actually create our own hooks.

## The Example

Let's start off with a basic example where a class provides us with the ability to load (what appears to be)
some profile information.

```js
// Profile.js
import { BehaviorSubject } from 'rxjs'

export default new class {
  constructor() {
    this._controller = new BehaviorSubject({
      firstName: null,
      lastName: null,
      age: null,
      active: false
    })
  }

  get data() {
    return this._controller
  }

  load = async () => {
    // get data from some external source
    const data = await pull_data()
    this._controller.next(data)
  }
}
```

Nothing special. Just a little RxJS `BehaviorSubject` being exposed from the `data` property and a `load` method
which pulls data from an external source and feeds it to the `BehaviorSubject` for broadcasting or emitting.

Now, let's move on to the presentation...

```js
import React, { useState, useEffect } from 'react'
import Profile from './Profile'

export default () => {
  const [profile, setProfile] = useState(null)
  useEffect(() => {
    const stream$ = Profile.data
      .subscribe(data => {
        setProfile(data)
      })

    return () => stream$.unsubscribe()
  }, [])

  if (!profile) {
    return (
      <div onClick={() => Profile.load()}>Load Profile</div>
    )
  }

  return (
    <div className={'profile'}>
      <div>{`${profile.firstName} ${profile.lastName}`}</div>
      <div>{profile.active ? 'Active' : 'Not Active'}</div>
    </div>
  )
}
```

We are using the `Profile` class defined earlier and subscribe to the `data` property.
Once a value is received we update and re-render by calling `setProfile` with the given data.

Obviously we are using `useState` for the profile data and using `useEffect` to handle
the subscription as well as the cleanup. And because we are passing an empty array (`[]`) to 
the dependency list of `useEffect` we want this to run when the component is mounted.

If we don't have any profile data (e.g. the initial state) then a click is provided to load the data by
calling the `load` method on the `Profile` class.

See... simple.

## We Can Build It Better, Stronger Than Before

So what can we improve here?

Not much but, this pattern of "subscribe and update state" will more than likely be something we will repeat many times.
What if we could wrap that whole block of logic into a function?
What if we called that function a `hook`?

Let's try it...

```js
// Hooks.js
import React, { useState, useEffect } from 'react'

const useSubscription = (subject) => {
  const [data, setData] = useState(null)
  useEffect(() => {
    const stream$ = subject()
      .subscribe(value => setData(value))

    return () => stream$.unsubscribe()
  }, [])

  return data
}

export useSubscription
```

Look Mom! My first custom `hook`! 

Clearly there is nothing special here that we haven't already seen before. Just wraps everything into a nice little
function that we called `useSubscription`.

One difference, however, is adding the `subject` parameter to the `useSubscription` function. That parameter takes the place
of what we had earlier with `Profile.data`. If we didn't do that then this function could only be used for the `Profile` class.
By allowing the caller to pass the "subject" (to be subscribed too), this `hook` can be used for any similar scenario.

Let's see how it changes our presentation now...

```js
import React from 'react'
import Profile from './Profile'
import { useSubscription } from './Hooks'

export default () => {
  const profile = useSubscription(() => Profile.data)

  if (!profile) {
    return (
      <div onClick={() => Profile.load()}>Load Profile</div>
    )
  }

  return (
    <div>
      <div>{`${profile.firstName} ${profile.lastName}`}</div>
      <div>{profile.active ? 'Active' : 'Not Active'}</div>
    </div>
  )
}
```

Here we can see now that we're telling `useSubscription` the subject to be used. We aren't limited to just the `Profile.data`
class. Nice.

Clearly we can change this to be a little better too. For instance:
* Add an `initState` parameter to populate the `useState` with something other than `null`
* Add a return value to indicate whether or not data is present

## That Can't Be It... Can It?

Yup. That's it.

A custom `Hook` is simply wrapping an existng `hook` with your own logic.

