# Sample Mobile Application with Mithril and Cordova

[Mithril](mithril.js.org) is my favorite Javascript framework. It has a simple API, it's lightweight, it's very fast and it's just Javascript.
I believe it deserves a few more tutorials and sample applications to illustrate how great it is.

Inspired by what Jonny Buchanan (@insin) did for [large datasets](https://insin.github.io/ui-lib-samples/large-datasets/), this tutorial is unashamedly copying a sample mobile app developped by Christophe Coenraets (@ccoenraets), a developer evangelist working at Salesforce, who regularly publishes awesome tutorials, using the latest javascript technologies. His blog can be found [here](http://coenraets.org/blog/) and the model for the following tutorial can be found [here](http://coenraets.org/blog/2014/12/sample-mobile-application-with-react-and-cordova/).

The focus of this tutorial will be to highlight the similarities and differences between React and Mithril, when developing this sample application.

* No JSX, no tools, only javascript
* No templates: Components are the building blocks. By keeping the markup and the corresponding UI logic together, you have the full power of Javascript to express the UI.
* Virtual DOM. In Mithril, rendering a component means creating a lightweight description of the component UI. Mithril diffs this new description with the old one (if any), and generates a minimal set of changes to apply to the DOM.

## Introduction

As [@ccoenraets](https://twitter.com/ccoenraets) says, there is nothing like building an app to get familiar with a new language or framework. “App” is the application container that will transition pages in and out of the UI. Only one page is displayed at any given time.


In this article, we will build the application going through several iterations, from a simple static prototype to the final product. The source code for the application, including the different iterations and a Cordova version, is available in [this repository](https://github.com/Bondifrench/mithril-employee-directory).

## Iteration 1: Static Version

In this first version, we create and render the HomePage component with some hardcoded (static) sample data.

[View source](https://github.com/Bondifrench/mithril-employee-directory/blob/master/iteration1/js/app.js) | [Run it](http://bondifrench.github.io/mithril-employee-directory/iteration1/)

**Code Highlights:**

We create 3 components for our App: a `Header`, a `SearchBar`, and an `EmployeeList`.

Creating components is easy: you create a JavaScript object with a key called 'view': 

```Javascript
var MyComponent = {
	//the controller is optional
    controller: function() {
    	return {
    		greeting: "Hello"
    		};
    },
    view: function(ctrl) {
        return m("h1", ctrl.greeting);
    }
}

m.mount(document.body, MyComponent);
```
This `view` is a function that returns the UI description. The element returned by the view function is not an actual DOM node: it’s a description of the UI that will be diffed with the current description to perform the minimum set of changes to the DOM. To actually render the view, we use here `m.mount()`

Using a component can be done in two ways:  
- either by referring to it directly if it is a static component, like `SearchBar` or `EmployeeList` in the source code of Iteration 1
- or with the `m.component()` function, like in the following example: `m.component(Header, {text: 'Employee Directory'})`. Its first argument is the name of the component, the second is a plain javascript object which contains what you want to pass down.

Side Note: If you don't like the syntax of `m('input[type=search]')` you can use [MSX](https://github.com/insin/msx) (which is similar to [JSX](https://facebook.github.io/jsx/) in React).

Another side note: the default of `m('')` is a div, so you can ommit `div` if you want.

## Iteration 2: Data Flow

In this second version, we define an array of employees in the HomePage component, and we make the data flow down the component hierarchy to EmployeeList and EmployeeListItem. In this version, the list of employees is hardcoded: we’ll work with dynamic data in iteration 4.

[View source](https://github.com/Bondifrench/mithril-employee-directory/blob/master/iteration2/js/app.js) | [Run it](http://bondifrench.github.io/mithril-employee-directory/iteration2/)

**Code Highlights:**

In Mithril, you can add a plain javascript object to your custom component tags to pass properties to the component instance.

m.component(EmployeeList, {employees: employees})

Properties passed by the parent are available in this.props in the child. For example EmployeeList can access the list of employees provided by HomePage in this.props.employees.
In a list component (like EmployeeList), it’s a common pattern to programmatically create an array of child components (like EmployeeListItem) and include that array in the Mithril description of the component.

```Javascript
var EmployeeList = {
	view: function(ctrl, args) {
		var items = args.employees.map(function(employee) {
			return m.component(EmployeeListItem, {
				key: employee.id,
				employee: employee
			})
		})
		return m('ul', items);
	}
}
```
The key attribute (like in EmployeeListItem above) is used to uniquely identify an instance of a component (useful in the diff process).

Unlike React, which can have some surprising [behaviours]http://www.bennadel.com/blog/2880-a-quick-look-at-rendering-white-space-using-jsx-in-reactjs.htm) when translating JSX to HTML, there is no whitespace created by a `m()` function so if you render the following
```Javascript
m('span', args.employee.firstName),
m('span', args.employee.lastName)
```
Unless you have inserted a whitespace in your strings, the first name won't be separated from the last name. 
Some might prefer with, some without, I prefer without because I like the concept of pure functions, m() is to create a virtual DOM element, with no side effects


## Iteration 3: Inverse Data Flow

In the previous version, the data flew down the component hierarchy from HomePage to EmployeeListItem. In this version, we make data (the search key to be specific) flow upstream, from the SearchBar to the HomePage where it is used to find the corresponding employees.

[View source](https://github.com/Bondifrench/mithril-employee-directory/blob/master/iteration3/js/app.js) | [Run it](http://bondifrench.github.io/mithril-employee-directory/iteration3/)

**Code Highlights:**

In this version, the inverse data flow is implemented by providing the child (SearchBar) with a handler to call back the parent (HomePage) when the search key value changes.

```Javascript
m.component(SearchBar, {searchHandler: ctrl.searchHandler })
```
## Iteration 4: Async Data and State

In this version, we implement employee search using an async service. In this sample app, we use a mock in-memory service (defined in data.js) that uses promises so you can easily replace the implementation with Ajax calls. We keep track of the search key and the list of employees in the HomePage state.

[View source](https://github.com/Bondifrench/mithril-employee-directory/blob/master/iteration4/js/app.js) | [Run it](http://bondifrench.github.io/mithril-employee-directory/iteration4/)

**Code Highlights:**
In React: The state (this.state) is private to the component and is changed using this.setState().
In Mithril: 

The UI is automatically updated when the user types a new value in SearchBar. This is because when the state of a React component is changed, the component automatically re-renders itself (by executing its `render()` function in React, the `view` function in Mithril).

In React: getInitialState() executes once during the component lifecycle and is used to set up the initial state of the component.
Im Mithril: 

###Iteration 5: Routing

In this version, we add an employee details page. Because the application now has more than one view, we add a simple view routing mechanism.

[View source](https://github.com/Bondifrench/mithril-employee-directory/blob/master/iteration5/js/app.js) | [Run it](http://bondifrench.github.io/mithril-employee-directory/iteration5/)

**Code Highlights:**


There are many options to implement routing. Some routing libraries are specific to React (check out react-router), but you can also use other existing routing libraries. Because the routing requirements of this sample app are very simple, I used a simple script (router.js) that I have been using in other sample apps.

componentDidMount() is called when a component is rendered.

## Iteration 6: Styling

Time to make the app look good. 
In Mithril, we can use CSS selectors to specify attributes. We can aslo use the `.` syntax to add CSS classes and the `#` to add an id. 
In this version, we use the Ratchet CSS library to provide the app with a mobile skin.

[View source](https://github.com/Bondifrench/mithril-employee-directory/blob/master/iteration6/js/app.js) | [Run it](http://bondifrench.github.io/mithril-employee-directory/iteration6)

**Code Highlights:**

In React: Notice the use of className instead of class
In Mithril: When having static Notice how we can easily 

## Iteration 7: Maintaining State

If you run iteration 6, you’ll notice that when you navigate back to HomePage from EmployeePage, HomePage has lost its state (search key and employee list). In this version, we fix this problem and make sure the state is preserved.

[View source](https://github.com/Bondifrench/mithril-employee-directory/blob/master/iteration7/js/app.js) | [Run it](http://bondifrench.github.io/mithril-employee-directory/iteration7)

**Code Highlights:**

To keep track of the state, we introduce a new parent container (App) that maintains the state (searchKey and employee list) of the Home page and reloads it when the user navigates back to the Home Page.

## Iteration 8: Adding animations
We will use the Mithril-page-slider designed by @ArthurClemens

Coming soon!

Checkout also the modular implementation of Material Design for Mithril developed by @ArthurClemens [Polythene](http://polythene.js.org/#polythene)

## Iteration 9: Running in Cordova

Coming soon!

##Additional resources:

My other tutorial using Mithril: [Mithril trader](https://github.com/Bondifrench/mithril-trader)  
My Sublime package for easy autocompletion of most Mithril methods in Sublime Text: [Mithrilizer](https://github.com/Bondifrench/Mithrilizer)

Other tutorials on Mithril by Gilbert @mindeavor [here](http://gilbert.ghost.io/mithril-js-tutorial-1/) and [here](http://gilbert.ghost.io/mithril-js-tutorial-2/)

[Polythene](http://polythene.js.org/#polythene) is a library by Arthur Clemens using Mithril Components

Two open-source non-trivial applications using Mithril in production:
- [Lichobile](https://github.com/veloce/lichobile) with source code using Mithril [here](https://github.com/veloce/lichobile/tree/2.3.x/project/src/js)
- [Flarum](http://flarum.org/) with source code using Mithril [here](https://github.com/flarum/core/tree/master/js)

[Here](http://tobyzerner.com/mithril/) you can read on how Toby Zerner made the transition from Ember.js to Mithril

Further resources can be found on the Mithril [wiki](https://github.com/lhorie/mithril.js/wiki)
