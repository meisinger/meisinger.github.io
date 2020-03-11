---
layout: single
title:  "Getting the Hook"
date:   2020-03-01 19:04:17 -0500
read_time: true
categories: [Development]
tags: [React, React-Native]
---
If you have been ignoring the Functional Component aspect of React and only working with Class Components, you might have
missed the `Hook` revolution.

One of the reasons why I focused on Class Components, and payed little attention to Fuctional Components,  was the ability 
to "hook" into the `Component Lifecycle`. Most of the time I was living in three methods: `constructor`, `componentDidMount` and 
`componentWillUnmount` methods. Setting up my state in the `constructor`, making all of my subscription calls in 
`componentDidMount` and cleaning everything up in `componentWillUnmount`.

To me, Fuctional Components were only used for stateless and "pure" components. As soon as one of these components needed
to setup a subscription, back it went to the Class Component. I believe there we times I would code the following in my
sleep simply due to the number of times it was written:

{% highlight js %}
export default new class extends React.Component {
  constructor(props) {
    super(props)
  }
}
{% endhighlight %}

When I found the `Hook` paradigm, it changed my perspective on everything React related.

## Let's `use` this

The `Hook` paradigm exposes a few fuctions that really turn things on their head:
* `useState`
* `useEffect`

There are a few others that we can get into later but for now, these are the two that you will likely use the most.
At least initially any way.

## State the Obvious

The `useState` replaces the need to create your state within the `constructor`. Before, we typically would create our state
in the following manner.

{% highlight js %}
constructor(props) {
  super(props)

  this.state = { active: true }
}
{% endhighlight %}

With the `useState` hook, that changes to the following

{% highlight js %}
const [active, setActive] = useState(true)
{% endhighlight %}

It should be fairly easy to gather what is happening in those two code blocks. Both are creating state where "active" will
be tracked and maintained across renderings. Both are also setting "active" to a value of "true" initially.

What may not be obvious at first is what is being returned from the `useState` function. The `useState` function returns an array
where the first item is the the state and the second item is a function use to modify or change the state. Simple.

A few subtle things to cover here. First, I can stop using `this.state` since the state is not a local variable.
You might also miss being able to call a function after the state was updated with the `setState` function but, 
you can simple use an effect to handle that instead.

## Affect the Effect

The `useEffect` hook gives us the ability to call functions and methods that produce side effects. Well that clears everything up
now doesn't it? What the hell is a side effect?

A "side effect" is really anything that changes something else (way over simplified). Consider the variable `foo`. If the value of 
`foo` is changed from `bar` to `baz` then we have created a "side effect." In other words, the value of the variable has changed.

If we take a cue from Functional Programming, a function that requires or accesses data outside of its given parameters is 
consider to have side effects. And while we can debate the validity and practicality of this, it serves to prove our point and
show how `useEffect` can be applied.

{% highlight js %}
const NameBadge = ({ id, onClick }) => {
  const [name, setName] = useState(null)
  useEffect(() => {
    const load = async () => {
      const data = await getNameFromId(id)
      setName(data)
    }

    load()
  }, [id])

  if (!name)
    return null

  return (
    <div className={'badge'}>
      <div onClick={() => onClick(id)}>{name}</div>
    </div>
  )
}
{% endhighlight %}

In this example, `getNameFromId` gets the name from some external data source.

If we look a little deeper into the `useEffect` method, we see that it has two parameters. The first is the method or function
that should be executed. The second is an array but it isn't really clear what it does.

The second parameter to the `useEffect` method as a "dependency list" of sorts with a few special rules that help dictate
when the given function is executed:
* If omitted, execute for every update, state or parameter change
* If an empty array, execute once when the component is mounted
* Otherwise, execute if any of the given dependencies change

In our example, the `id` parameter is a dependency so the function will be executed only when the `id` parameter changes.
When it does change the function is called which calls `setName` which updates the `name` state and causes the component
to re-render.

One thing not shown is the `useEffect` actually expects a function to be returned. This function is called by React only
when the given function is called and after the component has rendered.

Perhaps an example is best served. I am a huge RxJS fan so using subscriptions is the easiest example I can think of:

{% highlight js %}
const NameBadge = ({ id, onClick }) => {
  const [name, setName] = useState(null)
  useEffect(() => {
    const stream$ = getNameFromId(id)
      .subscribe(data => setName(data))

    return () => stream$.unsubscribe()
  }, [id])

  if (!name)
    return null

  return (
    <div className={'badge'}>
      <div onClick={() => onClick(id)}>{name}</div>
    </div>
  )
}
{% endhighlight %}

As you can see we return a function that cleans up the subscription by calling `unsubscribe`.

## `Hooks` Rule

It's interesting to note that "hooks" have two rules for when and where you can use them. The rules are simple to
remember:
* Only call hooks at the Top Level
* Only call hooks from React Functions

The first rule makes sense. Basically, don't call "hooks" within loops or or nested functions. The second rule ensures you don't
try to use "hooks" within the Class paradigm. That's it. 

Next up, we'll cover one of my favorite things about the new "Hook" paradigm, custom hooks.

