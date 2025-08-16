---
categories:
- Development
date: "2020-03-06T17:12:51Z"
excerpt_separator: <!--nice-->
read_time: true
tags:
- React
- React-Native
title: The Ref
---
No. Not the movie. The `hook`.<!--more-->
If you've never seen the movie "The Ref" then you owe it to yourself (and Denis Leary) to give it a watch sometime. It has its moments.

But we're not here to talk about the movie. Instead we are going to talk about the `useRef` hook.

## This Is Not The Reference You're Looking For

When I think of a reference in React I think about using _(what React calls)_ a "Callback Ref" like this:

```js
<div ref={(ref) => this.component = ref}>Hello World!</div>
```

Where basically you are grabbing the reference to an existing component and setting it a variable for later use. It should go without saying but, this only works within the Class paradigm of React.

But clearly, this is not the reference we are looking for.

Instead the `useRef` hook operates in the reverse by creating a reference that you can then assign to a component. Like as follows:

```js
const ref = useRef()

return (
  <div ref={ref}>Hello World!</div>
)
```

## Making The Wrong Calls

When using the "Callback Ref" approach the reference has already been created and is well known. In the previous example, React knows that the reference is a DOM element. Therefore we can access the reference directly (e.g. `this.component.focus()`).

When using the `useRef` hook however, we don't know what the underlying reference is yet. We have to wait until we assign it before React knows what the reference points to. 

Because we don't yet know until the assignment, when we use the `useRef` hook we can't access the reference directly like we normally would. Instead we have to go through the `current` property. Like this:

```js
export default () => {
  const ref = useRef()
  
  const clicked = () => {
    ref.current.focus()
  }
  
  return (
    <div>
      <div>
        <input ref={ref} type="text" />
      </div>
      <button onClick={() => clicked()}>Focus Up</button>
    </div>
  )
}
```

From this example we can see what is really happening under the covers. 

In reality, when the `ref` is assigned to the `<input />` element, React sets the `current` property for our `ref` to the DOM element. This means that our `ref` is mutable through the `current` property

## Wait, What?

With the understanding that the `useRef` hook produces a mutable value via the `current` property, this opens up more options to us then before with the simple "Callback Ref" approach.

With `useRef` we can store timers, intervals, DOM elements or even _(wait for it...)_ `Animated.Value` in React Native.

```js
export default ({ active, activeTint = '#900', inactiveTint = '#000' }) => {
  const animatedColor = Animated.Value(0)
  const refColor = useRef(animatedColor)
  const textColor = refColor.current.interpolate({
    inputRange: [0, 1],
    outputRange: [
      inactiveTint,
      activeTint
    ]
  })
  
  const [lastActive, setLastActive] = useState(false)
  useEffect(() => {
    if (active !== lastActive) {
      Animated.timing(refColor.current, {
        toValue: (active) ? 1 : 0,
        duration: 175
      })
    }
    
    setLastActive(active)
  }, [active])
  
  const text_style = {
    fontSize: 16,
    color: textColor
  }
  
  return (
    <Animated.View>
      <Animated.Text style={text_style}>Hello World</Animted.Text>
    </Animated.View>
  )
}
```

Sure, this is a silly example but it shows the potential.

Basically what we have a created a component that when `active` shows the text in "red" and when not `active` whos the text in "black"
When the property changes we animate the color change.

Does anyone else smell a custom `hook` in our future? Nah, probably not.

## Silver Reference?

As cool as `useRef` is, it should really be used sparingly. Having a mutable variable might sound like a solution to a lot of things
but it does come with some caveats.

For one thing, changes to the `ref` is a side effect so it must be done within a `useEffect` function. 
Along those same lines, changing (or mutating) the `current` property doesn't trigger a re-render.

And then we get to the whole "sharing ref" conversation which should honestly be avoided as much as possible.
If you run into a situation where you think "sharing a ref" will solve your problem, give the problem a second look. There may be a larger
underlying problem lurking around some where.

## Setting That Native Props

Unfortunatley, if you need access to `setNativeProps` within React Native, you can't help but use `useRef`.

There is no way around it as far as I know.

```js
export default ({ subject, ...props }) => {
  const component = useRef()

  useEffect(() => {
    const $stream = subject
      .subscribe(data => {
        component.current.setNativeProps({ text: data })
      })

    return () => $stream.unsubscribe()
  }, [])

  return (
    <TextInput ref={component} {...props} onChangeText={subject.changed} />
  )
}
```

Covering everything going on here is a little beyond the scope of this conversation but, it is important to note that
in order to set the `text` property of the `TextInput` component you need to be able to call the `setNativeProps`.

That can only be accessed with a reference to the component, hence... `useRef`


Perhaps we can cover what is happening in this example in the next post... stay tuned.
