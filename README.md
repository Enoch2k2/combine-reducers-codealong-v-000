# Redux Combine Reducers Lab

## Objectives

1. Write action creators and reducers to modify different pieces of application state
2. Build Redux's `combineReducers` function
3. Use the `combineReducers` function to delegate different pieces of state to each reducer

## Introduction
So far we have been using a single reducer to return a new state when an action is dispatched.  This works great for a small application where we only need our reducer to manage the state of one resource.  However, as you will see, when working with multiple resources, placing all of this logic in one reducer function can quickly become unwieldy.

Enter combineReducers to save the day! In this lab, we'll see how Redux's combineReducers function lets us delegate different pieces of state to separate reducer functions.

We'll do this in the context of a book application that we'll use to keep track of programming books that we've read.

We want our app to do two things:
1. Keep track of all the books we've read: title, author, description.
2. Keep track of the authors who wrote these books.

### Determine Application State Structure

Our app will need a state object that stores two types of information:

1. All our books, in an array
2. Our authors, also in an array

Each of these types of information--all our books, and the authors--should be represented on our store's state object. We want to think of our store's state structure as a database.  We will represent this as a belongsTo/hasMany relationship, in that a book belongsTo an author and an author hasMany books.  So this means each author would have its own id, and each book would have an authorId as a foreign key.

With that, we can set the application state as:

```
{
  books: // array of books,
  authors: //array of favorite books
}
```
So our state object will have two top-level keys, each pointing to an array.  For now, let's write a single reducer to manage both of these resources.  Inside the `src/reducers/index.js` file we write the following.

```javascript
  export function bookApp(state = {authors: [], books: []}, action){
    switch (action.type) {
    case "ADD_BOOK":
      // need to add bookid
      return Object.assign(state, {books: state.books.concat(action.payload)})
    case "REMOVE_BOOK":
      let idx = state.books.indexOf(action.payload)
      let books = [].concat(state.slice(0, idx), state.slice(idx + 1, state.length))
      return Object.assign(state, {books: books})
    case "ADD_AUTHOR":
        return Object.assign(state, {authors: state.authors.concat(action.payload)})
    case "REMOVE_AUTHOR":
    let idx = state.authors.indexOf(action.payload)
    let authors = [].concat(state.slice(0, idx), state.slice(idx + 1, state.length))
    return Object.assign(state, {authors: authors})
    default:
      return state
    }
  }

  export const store = createStore(bookApp)
```

As you can see, just working with two resources increases the size of our reducer to almost twenty lines of code.  Moreover, by placing each resource in the same reducer, we are coupling these resources together, where we would prefer to maintain their separation.  

## Refactor by using combineReducers

The combineReducers function allows us to write two separate reducers, then pass each reducer to the combineReducers function to produce the reducer we wrote above.  Then we pass that combined reducer to the store.  Let's write some code, and then we'll walk through it below.  
Open up the src/reducers/index.js function and write the following:

`src/reducers/bookApp.js`

```javascript
  let rootReducer = combineReducers({books: booksReducer, authors: authorsReducer})

  export const store = createStore(rootReducer)

  function booksReducer(state = [], action){
    switch (action.type) {
    case "ADD_BOOK":
      // need to add bookid
      return state.concat(action.payload)
    case "REMOVE_BOOK":
      let idx = state.indexOf(action.payload)
      let books = [].concat(state.slice(0, idx), state.slice(idx + 1, state.length))
      return books
    default:
      return state;
  }

  function authorsReducer(state = [], action){
    switch (action.type) {
      case "ADD_AUTHOR":
        return state.concat(action.payload)
      case "REMOVE_AUTHOR":
        let idx = state.indexOf(action.payload)
        let authors = [].concat(state.slice(0, idx), state.slice(idx + 1, state.length))
      return authors
      default:
        return state
    }
  }
```

There's a lot of code there, so let's unpack it a bit.  At the very top you see the following line:

```javascript

	  let rootReducer = combineReducers({books: booksReducer, authors: authorsReducer})

```

We're telling redux to produce a reducer which will return a state that has a key of books with a value equal to the return value of the booksReducer and a key of authors with a value equal to the return value of the authorsReducer.  Now if you look at the booksReducer and the authorsReducer you will see that each returns a default state of an empty array.  So by passing our rootReducer to the createStore method, the application maintains its initial state of `{books: [], authors: []}`.  

### Examining Our New Reducers

Now if you examine the authorsReducer, notice that this reducer only concerns itself with its own slice of the state.  This makes sense.  Remember that ultimately the array that the authorsReducer returns will be the value to the key of authors.  Similarly the authorsReducer only receives as it's state argument the value of state.authors, in other words the authors array.  

So examining the authorsReducer, we see that we no longer retrieve the list of authors with a call to `state.authors`, but can access the list of authors simply by calling `state`.

```javascript
  function authorsReducer(state = [], action){
    switch (action.type) {
      case "ADD_AUTHOR":
        return state.concat(action.payload)
      case "REMOVE_AUTHOR":
        let idx = state.indexOf(action.payload)
        let authors = [].concat(state.slice(0, idx), state.slice(idx + 1, state.length))
      return authors
      default:
        return state
    }
  }
```

### Dispatching Actions

The combineReducer function returns to us one large reducer looks like the following:

```javascript

	function reducer(state = {authors: [], books: []}, action){
		    switch (action.type) {
		    	case "ADD_AUTHOR"
		    		return state.concat(action.payload)
		    	case 'REMOVE_AUTHOR'
		    ....
		    }
	}

```

Because of this, it means that we can dispatch actions the same way we always did.  `store.dispatch({type: 'ADD_AUTHOR', {title: 'huck finn'}})` will hit our switch statement in the reducer and add a new author.  

One thing to note, is that if you want to have more than one reducer respond to the same action, you can.  For example, let's say that when a user inputs information about a book, the user also inputs the author's name.  The action dispatched may look like the following: `store.dispatch({action: 'ADD_BOOK', payload: {title: 'huck finn', authorName: 'Mark Twain'}})`.  Our reducers may look like the following:

```javascript
  function booksReducer(state = [], action){
    switch (action.type) {
    case "ADD_BOOK":
      // need to add bookid
      return state.concat({title: action.payload.title})
    case "REMOVE_BOOK":
      let idx = state.indexOf(action.payload)
      let books = [].concat(state.slice(0, idx), state.slice(idx + 1, state.length))
      return books
    default:
      return state;
  }

    function authorsReducer(state = [], action){
	    switch (action.type) {
	      case "ADD_AUTHOR":
	        return state.concat(action.payload)
	      case "REMOVE_AUTHOR":
	        let idx = state.indexOf(action.payload)
	        let authors = [].concat(state.slice(0, idx), state.slice(idx + 1, state.length))
	      return authors
	     	case "ADD_BOOK":
	     		let existingAuthor = state.filter(function(author){
	     			return author.name === action.payload.authorName
	     		})
	     		if(existingAuthor.length > 1){
	     			state.concat({name: action.payload.authorName})
	     		}else {
	     			return state
	     		}
	      default:
	        return state
	    }
  }

```

So you can see that both the booksReducer and the authorsReducer will be modifying the store's state when an action of type ADD_BOOK is dispatched.  The booksReducer will provide the standard behavior of adding the new book.  The author's reducer will look to see if there is an existing author of that type in the store, and if not add a new author.  

> Note: The above code has a bug in that we would not be able to add a foreign key of the authorId on a book, because when the books reducer receives the action, the related author would not exist yet.  Therefore, we would take a different approach, and likely dispatch two sequential actions of types 'ADD_AUTHOR' and then 'ADD_BOOK'.  However, the above example is to show that two separate reducers can respond to the same dispatched action.   

### Resources

+ [Implementing Combine Reducers from Scratch](https://egghead.io/lessons/javascript-redux-implementing-combinereducers-from-scratch)
