
# Basics

we'll learn nodejs. nodejs is a js-engine built on v8(top jsengine) primarily to build backend services using js.

### console.log

- `console.log("hello")` == `print("hello")`
- `console.log(20+3)` >> `23`
- single quotes can also be used.

### comments

```js
//this is a comment
console.log("hello")
/*
this is a 
multiline 
comment.
*/
```

### variables

```js
var name = "deepam"   //string
var age = 25   // int
var ismale = true  // bool
var dob = new Date(2000,12,17)
var hobbies = {"music", "coding", "movies"} // object type
var undefined_var = undefined   // this is a undefined type (~None)

console.log(name, age, ismale, dob)
consolelog(typeof name, typeof age, typeof ismale, typeof dob)
```
- js is a dynamic typed variable
- must beign with letter, underscore(\_) or dollar sign($).

### strings

```js
var brand="nestle"
brand.length //6
brand.toUpperCase() // NESTLE
brand.substring(0,4) // nest
console.log(brand+brand) //nestlenestle

//string formatitng
console.log(`${brand} is a household brand.`)  // >>nestle is a household brand
```

### objects
anagolous to dicts in python
```js
person = {
name: "deepam",
age: 25,
ismale: true,
hobbies: {"music", "coding", "movies"} 
}

//access via dot notation
concole.log(person.age)    //>>25
console.log(Object.values(person))  // same as person.values() in python
console.log(Object.keys(person))  // same as person.keys() in python

```

### conditionals
```js
if (person.ismale) {
	console.log(`${person.name} is male`)
}
else {
	console.log(`${person.name} is non male`)
}

console.log(!person.ismale)   //>>false
```


### arrays
```js
names = ['deepam', 'minda', 'nayna', 'tuni']
console.log(names[0])
console.log(names[1])
console.log(names[2])
console.log(names.length)

```


### loops
```js
//for i 
for (var i=0; i < names.length; i++){
	console.log(names[i])
}
// for of
for (const name of names){
	console.log(name)
}

//for each
names.foreach(function(name)){
	  console.log(name)
}

//while

var i=0
while(i<4){
	console.log(i);
	i = i+1
}
```


### break and continue
```js
var i=0
while(i<4){
	console.log(i);
	i = i+1
	if (i>2) {
		break
	}
	
}
```

### functions normal arrow and named.

### events and event listeners
In web development, events and event listeners are crucial for making web pages interactive and responsive to user actions. Let’s break down what these concepts mean and why they are important.

1. What is an Event?

An event in JavaScript refers to a specific action or occurrence that happens in the browser, typically in response to user interaction or other changes in the environment. Common examples of events include:
	•	User Actions:
	•	Clicking a button (click)
	•	Hovering over an element (mouseover)
	•	Pressing a key (keydown)
	•	Submitting a form (submit)
	•	Scrolling (scroll)
	•	Document/Window Actions:
	•	Page load (load)
	•	Resizing the browser window (resize)
	•	Changing the input value (change)
	•	Other Events:
	•	Network requests (fetch, load, error)

2. What is an Event Listener?

An event listener is a function that listens for a specific event to occur and then runs when that event is triggered. The event listener is attached to an element, and when the event happens (e.g., a button is clicked), the listener executes the code associated with it.

How to Add an Event Listener

You can add an event listener to an element using the addEventListener method:

const button = document.getElementById('myButton');
button.addEventListener('click', function() {
    alert('Button clicked!');
});

Here:
	•	addEventListener attaches the listener to the button element.
	•	'click' specifies the event to listen for.
	•	The function is the callback that gets executed when the event occurs.

3. Why Are Events and Event Listeners Important in Web Development?

Events and event listeners are critical for creating interactive user interfaces (UI). They allow developers to:
	•	Respond to user actions: Reacting to clicks, form submissions, or other user inputs is the foundation of interactivity.
	•	Improve user experience (UX): Event-driven interactions (like hover effects, dynamic updates, etc.) make websites feel more responsive and engaging.
	•	Implement dynamic content updates: Through events, content can be updated without needing to reload the page (e.g., via AJAX or single-page applications).
	•	Decouple actions from elements: By attaching event listeners, you can define logic separately from HTML structure, making the code more maintainable and modular.

4. Types of Event Listeners

There are two primary ways to add event listeners:
	•	Inline Event Handlers (Not Recommended)
You can define an event directly within HTML elements (e.g., <button onclick="alert('Clicked!')">Click me</button>). However, this is often discouraged due to maintainability and separation of concerns.
	•	JavaScript Event Listeners (Recommended)
Attaching event listeners in JavaScript allows more flexibility, modularity, and easier management, especially when dealing with multiple events or elements.

5. Event Propagation (Bubbling and Capturing)

Events in JavaScript can propagate through the DOM in two phases:
	•	Bubbling: The event starts from the target element and bubbles up to the root of the DOM.
	•	Capturing (Trickling): The event starts from the root of the DOM and trickles down to the target element.

document.getElementById('parent').addEventListener('click', function() {
    alert('Parent clicked!');
}, true);  // true for capturing phase

document.getElementById('child').addEventListener('click', function() {
    alert('Child clicked!');
}, false);  // false for bubbling phase

	•	In bubbling, events are first triggered on the target element, then propagate up the DOM.
	•	In capturing, events are first triggered on the root element and then trickle down to the target.

You can choose which phase to listen for by passing true (capturing) or false (bubbling) as the third argument in addEventListener.

6. Event Delegation

Instead of attaching individual event listeners to each child element, you can use event delegation. This is when you attach a single listener to a parent element and let events propagate down to the target child element.

document.getElementById('parent').addEventListener('click', function(event) {
    if (event.target && event.target.matches('button.classname')) {
        alert('Button clicked!');
    }
});

This technique is more efficient, especially when dynamically adding/removing child elements.

7. Common Event Types
	•	User Interaction:
	•	click, dblclick (double click)
	•	keydown, keyup, keypress
	•	mouseover, mouseout
	•	focus, blur
	•	Form Events:
	•	submit, change, input
	•	focus, blur
	•	Window/Document Events:
	•	load, resize, scroll
	•	beforeunload, unload
	•	Touch and Gesture Events (for mobile):
	•	touchstart, touchmove, touchend

8. Conclusion

Events and event listeners are essential for making interactive and dynamic web pages. They provide the means to detect and respond to user actions (like clicks, keystrokes, and form submissions), enabling richer web applications. Understanding how to use them effectively is key to mastering client-side JavaScript and creating modern, responsive websites.

### states components and props


# JS for python devs

https://www.youtube.com/watch?v=pdRTlDLMEww

![[Screenshot 2025-01-16 at 12.58.48 PM.png]]

- package manager: 
	- npm is equivalent to pip.
	- yarn is the other option.
- compiler: 
	- converts modern js into basic old school js to be able to render it in legacy browser.
	- babel is the most propular one
- the bundler:
	- makes code more performant by bundling all files it into a small number of js files.
	- webpack is the most standard one.

# React
https://www.youtube.com/watch?v=b9eMGE7QtTk



# Questions

- wtf is jquery?
- wtf is es6: msking js like python; classes, lmbdas, string templating, default args.
- jsx?: templatisez html?



