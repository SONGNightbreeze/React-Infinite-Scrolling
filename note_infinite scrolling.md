1. create custom hook useBookSearch.js and we will use axios

> we will use function component remove the JSX, because we're not actually going to return JSX

### we want to use useEffect,useState
```import {useEffect, useState} from 'react'```


### send to our search, we also want to send in here what page number on 
```export default function useBookSearch(query, pageNumber)```


### set up access inside of useEffect
```js 
    useEffect(()=>{
        // import axios from access
        axios({
            method:'GET',
            //this is the Open Library API and we're using the Search API
            url:'http://openlibrary.org/search.json',
            //these are things you can find in the API documentation for this API
            params: { q: query, page: pageNumber },
            }).then(res => {
                console.log(res.data)
        })
    },[query,pageNumber])
```

### App.js
```js
import React, {useState} from 'react'
import useBookSearch from './useBookSearch'

const [query, setQuery] = useState('')  // default our query is going to be an empty string
const [pageNumber, setPageNumber] = useState(1)
useBookSearch(query,pageNumber)

// now we have actually have these variables being set but we're not updating them anywhere
// the easy method to update is our query, so we can just update that in our input
<input type="text" onChange={handleSearch}> </input>

function handleSearch(e){ 
    setQuery(e.target.value)
    setPageNumber(1)
}

```
> **_NOTE:_** the first problem we meet 
-----------
> in website F12-console we can see a bunch of results because we're not cancelling our previous requests so every character  that we type is actually sending off a request
>
> every character we type cause this function run which updates our query which then updates our search, so we're going to fix  
>
> this data has a docs field which is all of the different books numFound, num_found is useful for what we know when to end our entire scrolling start is a page were currently on 
>
> this cancellation, we dont want to send a query every single time we want to cancel our query if we are typing information 
### in the useBookSearch, Axios is easy way to set up cancellations
```cancelToken: new axios.cancelToken(c => cancel = c)```
### create a variable
```let cancel```

> now we're going to be setting that variable cancel here to the C from our cancel token


### useEffect return
> and what we can do instead of user fact is if you return something from useEffect
> 
> you return a function inside that function we can just called cancel.
>  
> this is going cancel our request every single time it recall a useEffect

```return ()=> cancel()```

> you can see that immediately we're getting uncaught promises because every single time 
> that we cancel something it causes a error to occur instead of our promise
> 
> so actually catch that, we can say inside of here we want to catch our error
```js
}).catch(e =>{
    //we want to check to see if this is an Axios cancellation error
    if(axios.isCancel(e)) return 
})
```
    
> we just want to return essentially we're saying ignore every single time that we can still request because we meant to cancel it 
>
>if we start typing you'll see that only one request is made no matter how many
>
> characters we type its only going to send that one single request and its not going to send a bunch of extra requests because its actually cancelling those for us 
------------------------

2. useBookSearch.js
### set up state
```js
const [loading, setloading] = setState(true)
const [error, setError] = setState(false)
const [books, setBooks] = setState([]) //default is empty array
const [hasMore, setHasMore] = setState(false)  // in case there is no more results
```
### setting the state inside of our application
```js 
useEffect(()=>{
    setLoading(true)
    setError(false)
    axios({
        .....
    }).then(res => {
        //console.log(res.data)
        setBooks(prevBooks => {
            // return previous books combine with our new books
            return [...new Set([...prevBooks, ...res.data.docs.map(b => b.title)])]
        })
```                                      
> return to us a list of variables where we could have multiple titles
> 
> because this data actually dose things beyond just the title of the book
> so it may have multiple titles that are exactly the same and we want to remove those
> 
> Set in JavaScript you can pass it an array, and its going to return just unique values
> we need to do is convert this back to an array by just spreading over our entire Set       

> we have to check here if the res.data.docs.length > 0
```setHasMore(res.data.docs.length > 0)```
> means that we have no more data, because there's no books

> returned to us so we know that we never need to make this query again 
>
> we can set our loading here to false
```setLoading(false)```


> inside of our catch, we want to set our error to be true
```js
    .catch(e => {
        if (axios.isCancel(e)) return
        setError(true)
    })
```

> now that we have all these different variables being set we can return these to our user
```return { loading, error, books, hasMore }```

---------------------------------------------------------------------------
3. App.js
```js
    const {
        books,
        hasMore,
        loading,
        error
    } = useBookSearch(query, pageNumber)
    return (
        <>
        <input type="text" value={query} onChange={handleSearch}></input>
        {books.map(book => {
            return <div key={book}>{book}</div>
        })}
        <div>{loading && 'Loading...'}</div>
        <div>{error && 'Error'}</div>
        </>
    )
```
**_NOTE:_** second problem
> if we scroll back up to the top and lets see we change our query
> our results aren't actually being appended on the end 
>
> we have all of our results as well as our test results

### useBookSearch.js
> inside of  useBookSearch is use another effect and this effect is going to be 
>
> a very small effect and all its going to do is every single time that 
>
> we change our query, we're gonna set here our books, 
>
> we just want to set it back to the empty array
```js
useEffect(()=>{
    setBooks([])
},[query])
```
> so every time we change our query we're gonna reset our book array 
> so that we don't have any of our old books being shown up

------------------------------------------------------
4. setting up pagination -- pageNumber
### App.js
```<input type="text" value={query}>  ```
> in case we change our query somewhere else it'll be updated inside of the input field

```js
import React, { useState, useRef } from 'react'
const observer = useRef()
```

> we alse need to get a reference to the last book element
>
> we actually want to change our page number and add one to it
>
> intersection observer is going to allow us to say when something is on our screen
>
> we need to get an element reference to that last element in our books array in order to know which element is the last one 
>
> useRef is not part of our state so it doesn't update every single time that it changes
> 
> so when our reference changes it doesn't actually rerun our component
### we can use callback 
```js
import React, { useState, useRef, useCallback } from 'react'
const lastBookElementRef = useCallback() 
return <div ref={lastBookElementRef}>
```
> in that what happen is whenever this element is created it's going to call the function
> 
> instead of use callback with the reference to the element that we're using down here
> it's gonna call a function which is going to have node for example as the parameter
> this node corrisponds to this individual element right here ```<div ref={lastBookElementRef}>```
```js
const lastBookElementRef = useCallback(node =>{
    console.log(node)
}) 
```
> we only want to do this for our last book, so put the simple check her
```js
        {books.map((book, index) => {
            if (books.length === index + 1) {
            return <div ref={lastBookElementRef} key={book}>{book}</div>
            } else {
            return <div key={book}>{book}</div>
            }
        })}
```

5. now all we have left to do is actually set up our intersection observer 
    ```js
    const lastBookElementRef = useCallback(node => {
        // we want to check if we are loading then we just want to return 
        // because loading our information dont want to trigger our infinite scrolling
        // otherwise it'll just constantly call the API while we're loading and 
        if (loading) return

        // observer they have a variable property called current 
        // which is whatever the current iteration of that variable is 
        // so we have an observer what we want to do is we want to disconnect
        // that observer because we're going to reconnect it 
        // so we can save our observer current
        if (observer.current) observer.current.disconnect() 
        // is disconnect our observer from the previous element 
        // so that way our new last element will be hooked up correct
        
        //set our current observer 
        // and this is going to take in a function and this function takes all
        // the entries that are available so everything that it's watching 
        // is going to be in this entries array as soon as become visible
        observer.current = new IntersectionObserver(entries => {
        if (entries[0].isIntersecting && hasMore) {
            setPageNumber(prevPageNumber => prevPageNumber + 1)
        })
        // we're going to implement this function if we have a node
        // for example if something is actually our last element
        // we just want to make sure that our observer is observering it 
        // so we can get our current observer and we want to observer our node
        if (node) observer.current.observe(node)
    }, [loading, hasMore])

    ```