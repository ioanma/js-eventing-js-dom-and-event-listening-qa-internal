# Listening to Nodes

## Problem Statement

We've seen that we can select and manipulate nodes in the DOM using JavaScript.
We've even seen that we can create and remove elements. But wouldn't it be more
interesting if we made those types of thing happen _when_ we did something:
double-clicked on something, pushed a button, shook our mouse in a circle.
Those actions are called "events." In JavaScript we can "listen" for events and
use them to call functions that update the DOM. In this lesson, we'll learn how
to do just that.

## Objectives

1. Demonstrate using  `addEventListener()` to a DOM node
2. Prevent default behavior
3. Explain the difference between bubbling and capturing events
4. Add `stopPropogation()` to a DOM node

#### Instructions for In-Browser Learn IDE Users

If you are using the Learn IDE available in your browser, use `httpserver` to
launch the web server. You will see the message `Your server is running at ...`
followed by a string of numbers and dots.  This string is your temporary IP
address that is hosting your `index.html` file.  Copy this string of numbers,
open a new tab and past the string in to the URL bar.

#### Instructions for Students Using an Stand Alone Text-Editor

If you are using an standalone text editor such as Sublime or Atom, before we
get started, follow [these instructions][instructions] to manually fork and
clone the lesson repository on GitHub. In your forked and cloned copy, you'll
find the `index.html` file, which you can then manually open up in the browser.
(For instructions on opening HTML files in the browser from the Learn IDE, see
[this Help Center article](help-center).)


## Add `addEventListener()` to a DOM Node

To add an event we can call `addEventListener()` on the node.
`addEventListener()` takes two arguments: the name of the event, and a function
to handle the event.

Let's start by adding a listener for `click` events to the `main#main` element
in `index.html`. Once you've opened `index.html` in the browser, copy and paste
the following in the browser's JS console:

```js
const main = document.getElementById('main')

main.addEventListener('click', function(event) {
  alert('I was clicked!')
})
```

Now if you click on the `main` element (you can click its text, "My ID is
'main'!"), you should see an alert: `'I was clicked!'.` How does this work?

The first argument, `'click'`, is the name of the event we're listening for.

Click events make up the majority of events listened you'll use, but other
events you might use include `change`, `'keydown'`, `'keyup'`, `'load'`,
`'mouseover'`, `'mouseout'`.  You can find more possible events on [MDN][MDN].

The second argument is a function that accepts an event object as its argument.

The event has a number of useful properties on it. `keypress`, `keydown`, and
`keyup` events will have a `which` property that tells us which key was
pressed.  Try it out by copying and pasting the following in your js console in
the browser:

```js
const input = document.querySelector('input')
input.addEventListener('keydown', e => console.log(e.which))
```

_Don't forget arrow functions!_

You'll notice that, for example, pressing "enter" logs `13` in console, while
pressing "a" logs `65`. What do other keys log in the console?

## Prevent Default Behavior

Refresh the page. We've got a vendetta against the letter "g" (71), so we're
going to prevent the input from receiving "g"s. Paste the following in your
console:

```js
const input = document.querySelector('input')

input.addEventListener('keydown', e => {
  if (e.which === 71) {
    return e.preventDefault()
  }
})
```

Now try to enter "g" in the input — no can do!

Every DOM `event` comes with a `preventDefault` property. `preventDefault` is a
function that, when called, will prevent the default event from taking place. It
provides us an opportunity to intercept and change user interactions, usually in
more helpful ways than preventing them from typing "g".

You can probably think of a couple of web pages that have prevented you from
ESC-keying their ad away or one that wouldn't let you paste your password on
that "repeat to confirm" entry field. Yep, that was JavaScript.

Another related event property is called `stopPropagation`. Like
`preventDefault`, `stopPropagation` is a function that, when called, interrupts
the event's normal behavior. In this case, it stops the event from triggering
other nodes in the DOM that might be listening for the same event.

Wait. Do we mean one action can trigger multiple events? We sure do.

## Explain the Difference Between Bubbling and Capturing Events

DOM events propagate by bubbling (starting at the target node and moving up the
DOM tree to the root) and capturing (starting from the target node's parent
elements and propagating down the tree until it reaches the target) — by
default. Events nowadays all bubble. We can show this behavior by putting
listeners to those nested `div`s in `index.html`. Paste the following in your
console:

```js
let divs = document.querySelectorAll('div')

function bubble(e) {
  // remember all of those fancy DOM node properties?
  // we're making use of them to get the number
  // in each div here!

  // if `this` is a bit confusing, don't worry —
  // for now, know that it refers to the div that
  // is triggering the current event handler.
  console.log(this.firstChild.nodeValue.trim() + ' bubbled')
}

for (const aDiv of divs) {
  aDiv.addEventListener('click', bubble);
}
```

Now click on the `div` containing "5". You should see

```js
5 bubbled
4 bubbled
3 bubbled
2 bubbled
1 bubbled
```

What just happened? Well, the event starts at `div` 5, and then it bubbles up to
the topmost node. Along the way, it triggers any other nodes that are listening
for the event -- in this case, `'click'`.

Try clicking on a node that's not so deeply nested -- you should still see the
event bubble up, starting at the node that you clicked and hitting every node up
the tree until it reaches the top.

What about capturing? In order to capture, we need to set the third argument to
`addEventListener` to `true`. Let's try it out.

```js
let divs = document.querySelectorAll('div')

function capture(e) {
  console.log(this.firstChild.nodeValue.trim() + ' captured')
}

for (const aDiv of divs) {
  // set the third argument to `true`!
  aDiv.addEventListener('click', capture, true)
}
```

Now click on `div` 5. You should see

```js
1 captured
2 captured
3 captured
4 captured
5 bubbled
5 captured
4 bubbled
3 bubbled
2 bubbled
1 bubbled
```

Now, the event propagates from the top of the page towards the target node,
triggering event handlers as appropriate along the way.

Notice that the target node is the _last node to capture the event_, whereas
it's the _first node to bubble the event up_. This is the most important
takeaway.

**NOTE**: Don't worry if bubbling and capturing seems a bit confusing or weird.
The different event behaviors are the results of the browser wars of the 90s,
but most of the time it's safe just to stick to the default behaviors(which,
for the record, is bubbling). You can read more about bubbling and capture on
[StackOverflow][SO] and [QuirksMode][QM].

## Add `stopPropogation()` to a DOM Node

Now that you've learned a bit about the dangers and behavior of bubbling and
capturing, you understand how events propagate through the DOM. Much of the
time, since we're listening for very specific events, this doesn't matter: our
events can propagate up or down, and they'll only trigger the event handler(s)
that we want them to trigger. But sometimes, as with these `div`s, we have a
fairly generic event that we want to only hit its target. That's where
`stopPropagation` comes in.

Let's rewrite the bubbling example to stop propagation so that only one event is
triggered (remember to reload the page before entering this code!):

```js
const divs = document.querySelectorAll('div')

function bubble(e) {
  // stop! that! propagation!
  e.stopPropagation()

  console.log(this.firstChild.nodeValue.trim() + ' bubbled')
}

for (const aDiv of divs) {
  aDiv.addEventListener('click', bubble)
}
```

Now try clicking on any node — you should only see one log statement!

## Moving On

In order to move on from this lesson enter `learn`. If your code passes the
test, use `learn submit` to close out this lab and move on.

## Conclusion

We covered a lot in this lesson. Feel free to edit `index.html`, to write code
directly in the document (just put it between `<script></script>` tags). Check
out the [MDN][MDN] documentation and play around with the different events -
this stuff might feel intimidating at first, but it's important to practice so
you can get the hang of it!

You should understand how to add an event listener, how different event triggers
work, and how to intercept user interactions with `e.preventDefault()` and
`e.stopPropagation().`


## Resources

- [MDN - Web Events][MDN]
- [StackOverflow - Bubbling and Capturing][SO]
- [QuirksMode - Event order][QM]

[instructions]: http://help.learn.co/workflow-tips/github/how-to-manually-open-a-lab
[help-center]: http://help.learn.co/the-learn-ide/common-ide-questions/viewing-html-pages-in-the-learn-ide
[MDN]: https://developer.mozilla.org/en-US/docs/Web/Events
[SO]: http://stackoverflow.com/questions/4616694/what-is-event-bubbling-and-capturing
[QM]: http://www.quirksmode.org/js/events_order.html
