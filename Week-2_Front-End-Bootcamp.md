## React. JS Lanjutan - Proptypes
> * __Apa itu Prop types?__ </br>
> Prop Types merupakan sebuah lib yang dapat mmebantu kita untuk memeriksa data props yang dikirim agar sesuai dengan ekspektasi </br>
> > Jika tidak sesuai, akan muncul pesan error</br>

### Install Prop Types
* `npm install prop-types`

### Penggunaan Prop-types
* props yang akan dikirim harus sesuai dengan tipe data yang diinginkan.
* dan apabila tidak, maka akan didapatkan pesan error jika tidak sesuai ekspektasi.
* sehingga kesalahan dapat langsung diketahui dan segera diperbaiki.

## React Router
### React Router Basics
> install React Router DOM version : `npm i react-router-dom` </br>
> install React Router jika menggunakan React Native : `react-router-native` </br>
> Setelah Anda memiliki library ini, ada tiga hal yang perlu Anda lakukan untuk menggunakan React Router : </br>
> > * Setup your router </br>
> > * Define your routes </br>
> > * Handle navigation</br>

#### Configuring The Router
* import spesifik router yang dibutuhkan (BrowserRouter untuk web dan NativeRouter untuk mobile) dan wrap seluruh application di router itu.
```
import React from "react"
import ReactDOM from "react-dom/client"
import App from "./App"
import { BrowserRouter } from "react-router-dom"

const root = ReactDOM.createRoot(document.getElementById("root"))
root.render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
)
```

#### Defining Routes
* Ini biasanya dilakukan di top level, seperti di komponen App, tetapi dapat dilakukan di mana pun.
```
import { Route, Routes } from "react-router-dom"
import { Home } from "./Home"
import { BookList } from "./BookList"

export function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/books" element={<BookList />} />
    </Routes>
  )
}
```

#### Handling Navigation
* Langkah terakhir untuk React Router adalah handling navigation. Biasanya dalam sebuah aplikasi akan menavigasi dengan anchor tag, tetapi React Router menggunakan komponen custom `Link` sendiri untuk handle navigation. Komponen Tautan ini hanyalah wrapper di sekitar anchor tag yang membantu memastikan semua perutean dan rendering ulang bersyarat ditangani dengan benar.
```
import { Route, Routes, Link } from "react-router-dom"
import { Home } from "./Home"
import { BookList } from "./BookList"

export function App() {
  return (
    <>
      <nav>
        <ul>
          <li><Link to="/">Home</Link></li>
          <li><Link to="/books">Books</Link></li>
        </ul>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/books" element={<BookList />} />
      </Routes>
    </>
  )
}
```

### Advanced Route Definitions
> Ada banyak yang dapat dilakukan dengan perutean untuk membuat rute yang lebih kompleks, lebih mudah dibaca, dan secara keseluruhan jauh lebih fungsional. Hal ini dapat dilakukan melalui 5 teknik utama. </br>
> > * Dynamic Routing </br>
> > * Routing Priority </br>
> > * Nested Routes </br>
> > * Multiple Routes </br>
> > * useRoutes Hook </br>

#### Dynamic Routing
* Mari kita asumsikan bahwa kita ingin merender komponen untuk masing-masing buku dalam aplikasi kita. Kami dapat membuat hardcode setiap rute tersebut, tetapi jika kami memiliki ratusan buku atau ability bagi user untuk membuat buku, maka tidak mungkin untuk membuat hardcode semua rute ini. Sebaliknya kita membutuhkan rute yang dinamis.
```
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/books" element={<BookList />} />
  <Route path="/books/:id" element={<Book />} />
</Routes>
```

* Hampir selalu ketika memiliki rute dinamis seperti ini, kita ingin mengakses nilai dinamis dalam komponen khusus yang merupakan tempat hook `useParams` masuk.
```
import { useParams } from "react-router-dom"

export function Book() {
  const { id } = useParams()

  return (
    <h1>Book {id}</h1>
  )
}
```

#### Routing Priority
* Ketika hanya berurusan dengan rute hardcore, cukup mudah untuk mengetahui rute mana yang akan dirender, tetapi ketika berhadapan dengan rute dinamis itu bisa menjadi sedikit lebih rumit.
```
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/books" element={<BookList />} />
  <Route path="/books/:id" element={<Book />} />
  <Route path="/books/new" element={<NewBook />} />
</Routes>
```

#### Nested Routes
* Yang dilakukan dalam nesting adalah membuat `parent Route` yang memiliki path prop yang disetel ke shared path untuk semua komponen `child Route`.
```
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/books">
    <Route index element={<BookList />} />
    <Route path=":id" element={<Book />} />
    <Route path="new" element={<NewBook />} />
  </Route>
  <Route path="*" element={<NotFound />} />
</Routes>
```

#### Multiple Routes
Hal luar biasa lain yang dapat Anda lakukan dengan React Router adalah menggunakan beberapa komponen Routes secara bersamaan. Ini dapat dilakukan sebagai dua komponen Rute yang separate atau sebagai nested Routes.
* __Separate Routes__ : Jika ingin merender 2 bagian konten yang berbeda yang keduanya bergantung pada URL aplikasi, maka memerlukan beberapa komponen Routes.
```
 <aside>
        <Routes>
          <Route path="/books" element={<BookSidebar />}>
        </Routes>
  </aside>
```
* __Nested Routes__ : Cara lain untuk menggunakan beberapa komponen Routes adalah dengan nesting di dalam satu sama lain. Ini cukup umum jika memiliki banyak rute dan ingin membersihkan kode dengan memindahkan rute serupa ke file mereka sendiri.
```
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/books/*" element={<BookRoutes />} />
  <Route path="*" element={<NotFound />} />
</Routes>
```

#### useRoutes Hook
* Hal terakhir yang perlu diketahui tentang mendefinisikan rute di React Router adalah Anda dapat menggunakan objek JavaScript untuk menentukan rute alih-alih JSX.
```
import { Route, Routes } from "react-router-dom"
import { Home } from "./Home"
import { BookList } from "./BookList"
import { Book } from "./Book"

export function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/books">
        <Route index element={<BookList />} />
        <Route path=":id" element={<Book />} />
      </Route>
    </Routes>
  )
}
```

jika menggunakan useRoutes :
```
import { Route, Routes } from "react-router-dom"
import { Home } from "./Home"
import { BookList } from "./BookList"
import { Book } from "./Book"

export function App() {
  const element = useRoutes([
    {
      path: "/",
      element: <Home />
    },
    {
      path: "/books",
      children: [
        { index: true, element: <BookList /> },
        { path: ":id", element: <Book /> }
      ]
    }
  ])

  return element
}
```

## State Management React Redux 
* Redux adalah predictable state container untuk aplikasi JavaScript, dan tool yang sangat valuable untuk organizing application state. Ini adalah library populer untuk memanage state di aplikasi React, tetapi dapat digunakan juga dengan Angular, Vue.js atau sekadar JavaScript vanilla lama.
* main concepts:
  + reducers
  + actions
  + action creators 
  + store

### Reducer
* Reducer adalah pure function yang mengambil status sebelumnya dan action sebagai argumen dan mengembalikan state baru. Actions adalah objek dengan tipe dan optional payload
* Reducer menentukan bagaimana state aplikasi berubah sebagai respons terhadap actions yang dispatched ke store.
```
function myReducer(previousState, action) => {
  // use the action type and payload to create a new state based on
  // the previous state.
  return newState;
}
```

### Action
* Actions adalah plain JavaScript objects yang mewakili payload informasi yang mengirimkan data dari aplikasi ke store. Actions memiliki type dan optional payload.

### Action Creator
* Action creator adalah fungsi yang mengembalikan action object. 
* Action creators mungkin tampak seperti langkah yang berlebihan, tetapi mereka membuat segalanya lebih portabel dan mudah diuji. Action object yang dikembalikan dari action creators dikirim ke semua reducer berbeda di aplikasi.
* Conton action creator
```
export function addTodo({ task }) {
  return {
    type: 'ADD_TODO',
    payload: {
      task,
      completed: false
    },
  }
}

// example returned value:
// {
//   type: 'ADD_TODO',
//   todo: { task: '🛒 get some milk', completed: false },
// }
```
dan berikut contoh simple reducer yang berhubungan dengan action type ADD_TODO:
```
export default function(state = initialState, action) {
  switch (action.type) {
    case 'ADD_TODO':
      const newState = [...state, action.payload];
      return newState;

    // Deal with more cases like 'TOGGLE_TODO', 'DELETE_TODO',...

    default:
      return state;
  }
}
```

#### Combining Reducers
* Redux memberikan fungsi bernama `combineReducers` yang melakukan 2 task:
  + It generates a function that calls our reducers with the slice of state selected according to their key.
  + It then it combines the results into a single object once again.

### Store
* Di Redux, store mengacu pada objek yang menyatukan actions (yang mewakili apa yang terjadi) dan reducer (yang memperbarui state sesuai dengan action tersebut).
* Hanya ada satu store dalam aplikasi Redux.
* Store memiliki beberapa task:
  + Allow access to state via `getState()`.
  + Allow state to be updated via `dispatch(action)`.
  + Holds the whole application state.
  + Registers listeners using `subscribe(listener)`.
  + Unregisters listeners via the function returned by `subscribe(listener)`.

### Data Flow in Redux
> Salah satu dari banyak manfaat Redux adalah semua data dalam aplikasi mengikuti lifecycle pattern yang sama. Logika aplikasi lebih dapat diprediksi dan lebih mudah dipahami, karena arsitektur Redux mengikuti data flow searah yang ketat. </br>
* 4 Main Steps of the Data Lifecycle in Redux:
  + An event inside your app triggers a call to store.dispatch(actionCreator(payload)).
  + The Redux store calls the root reducer with the current state and the action.
  + The root reducer combines the output of multiple reducers into a single state tree.
  + The Redux store saves the complete state tree returned by the root reducer. The new state tree is now the nextState of your app.

## Async Actions with Middleware and Thunk
### Redux Middleware and Side Effects
* Redux Store tidak tahu apa-apa tentang logika async. Dan hanya tahu cara mengirimkan action secara sinkron, memperbarui state dengan memanggil fungsi root reducer, dan memberi tahu UI bahwa ada sesuatu yang berubah. Setiap asinkronisitas harus terjadi di luar store.
* "side effect" adalah perubahan apa pun pada state atau behavior yang dapat dilihat di luar returning value dari suatu fungsi. Beberapa jenis "side effect" yang umum adalah hal-hal seperti:
  + Logging a value to the console
  + Saving a file
  + Setting an async timer
  + Making an AJAX HTTP request
  + Modifying some state that exists outside of a function, or mutating arguments to a function
  + Generating random numbers or unique random IDs (such as Math.random() or Date.now())
* Middleware redux dirancang untuk memungkinkan penulisan logika yang memiliki side effects.
* Middleware Redux dapat melakukan apa saja saat melihat dispatched action: log something, modify the action, delay the action, make an async call, dan banyak lagi. Selain itu, karena middleware membentuk pipeline di sekitar fungsi `store.dispatch` yang sebenarnya, ini juga berarti bahwa kita benar-benar dapat melewatkan sesuatu yang bukan plain action object untuk di-dispatch, selama middleware memotong nilai tersebut dan tidak membiarkannya mencapai reducer.
* Middleware juga memiliki akses ke `dispatch` dan `getState`. Itu berarti kita dapat menulis beberapa logika asinkron di middleware, dan masih memiliki kemampuan untuk berinteraksi dengan store Redux dengan mengirimkan action.

#### Using Middleware to Enable Async Logic
Salah satu kemungkinan adalah menulis middleware yang mencari jenis action tertentu, dan menjalankan logika asinkron ketika melihat action tersebut, seperti contoh berikut:
```
import { client } from '../api/client'

const delayedActionMiddleware = storeAPI => next => action => {
  if (action.type === 'todos/todoAdded') {
    setTimeout(() => {
      // Delay this action by one second
      next(action)
    }, 1000)
    return
  }

  return next(action)
}

const fetchTodosMiddleware = storeAPI => next => action => {
  if (action.type === 'todos/fetchTodos') {
    // Make an API call to fetch todos from the server
    client.get('todos').then(todos => {
      // Dispatch an action with the todos we received
      storeAPI.dispatch({ type: 'todos/todosLoaded', payload: todos })
    })
  }

  return next(action)
}
```

#### Writing an Async Function Middleware
* Bagaimana jika kita menulis middleware yang memungkinkan kita pass fungsi untuk dispatch, alih-alih objek action? Kita dapat meminta middleware untuk memeriksa apakah "action" sebenarnya adalah sebuah fungsi, dan jika itu adalah sebuah fungsi, panggil fungsi tersebut segera. Itu akan memungkinkan kita menulis logika async dalam fungsi terpisah, di luar definisi middleware.
* Berikut tampilan middleware tersebut:
```
const asyncFunctionMiddleware = storeAPI => next => action => {
  // If the "action" is actually a function instead...
  if (typeof action === 'function') {
    // then call the function and pass `dispatch` and `getState` as arguments
    return action(storeAPI.dispatch, storeAPI.getState)
  }

  // Otherwise, it's a normal action - send it onwards
  return next(action)
}
```
* Kemudian kita dapat menggunakan middleware seperti ini :
```
const middlewareEnhancer = applyMiddleware(asyncFunctionMiddleware)
const store = createStore(rootReducer, middlewareEnhancer)

// Write a function that has `dispatch` and `getState` as arguments
const fetchSomeData = (dispatch, getState) => {
  // Make an async HTTP request
  client.get('todos').then(todos => {
    // Dispatch an action with the todos we received
    dispatch({ type: 'todos/todosLoaded', payload: todos })
    // Check the updated store state after dispatching
    const allTodos = getState().todos
    console.log('Number of todos after loading: ', allTodos.length)
  })
}

// Pass the _function_ we wrote to `dispatch`
store.dispatch(fetchSomeData)
// logs: 'Number of todos after loading: ###'
```

### Redux Async Data Flow
* Sama seperti normal action, pertama-tama kita harus handle a user event di application, seperti click on a button. Kemudian call dispatch(), dan pass in something, entah itu plain action object, a function, atau some other value yang dapat dicari oleh middleware.
* Setelah dispatched value mencapai middleware, dapat membuat async call, dan kemudian dispatch a real action object ketika async call completes.
* Ketika kita add async logic to a Redux app, add an extra step dimana middleware dapat run logic seperti AJAX requests, lalu dispatch actions. 

### Using the Redux Thunk Middleware
* Redux already has an official version of that "async function middleware", called the Redux "Thunk" middleware. The thunk middleware allows us to write functions that get dispatch and getState as arguments. The thunk functions can have any async logic we want inside, and that logic can dispatch actions and read the store state as needed.
> Writing async logic as thunk functions allows us to reuse that logic without knowing what Redux store we're using ahead of time.

#### Configuring the Store
* install redux-thunk : `npm install redux-thunk`
* setelah diinstall, kita dapat update redux-store di todo-app untuk menggunakan middlewarenya :
```
import { createStore, applyMiddleware } from 'redux'
import thunkMiddleware from 'redux-thunk'
import { composeWithDevTools } from 'redux-devtools-extension'
import rootReducer from './reducer'

const composedEnhancer = composeWithDevTools(applyMiddleware(thunkMiddleware))

// The store now has the ability to accept thunk functions in `dispatch`
const store = createStore(rootReducer, composedEnhancer)
export default store
```

#### Fetching Todos from a Server

#### Saving Todo Items
