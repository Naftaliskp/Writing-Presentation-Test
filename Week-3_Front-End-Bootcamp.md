## React Context
> * __Apa itu Context?__ </br>
> Context menyediakan cara untuk meneruskan data melalui component tree tanpa harus meneruskan prop secara manual di setiap level.</br>

### When to Use Context
* Context dirancang untuk berbagi data yang dapat dianggap “global” untuk tree dari React components, seperti current authenticated user, theme, atau preferred language. Misalnya, dalam kode di bawah ini kita secara manual memasukkan prop "theme" untuk menata style dari Button component
```
class App extends React.Component {
  render() {
    return <Toolbar theme="dark" />;
  }
}

function Toolbar(props) {
  // The Toolbar component must take an extra "theme" prop
  // and pass it to the ThemedButton. This can become painful
  // if every single button in the app needs to know the theme
  // because it would have to be passed through all components.
  return (
    <div>
      <ThemedButton theme={props.theme} />
    </div>
  );
}

class ThemedButton extends React.Component {
  render() {
    return <Button theme={this.props.theme} />;
  }
}
```
* Dengan konteks, kita dapat menghindari passing props melalui intermediate elements :
```
// Context lets us pass a value deep into the component tree
// without explicitly threading it through every component.
// Create a context for the current theme (with "light" as the default).
const ThemeContext = React.createContext('light');

class App extends React.Component {
  render() {
    // Use a Provider to pass the current theme to the tree below.
    // Any component can read it, no matter how deep it is.
    // In this example, we're passing "dark" as the current value.
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// A component in the middle doesn't have to
// pass the theme down explicitly anymore.
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // Assign a contextType to read the current theme context.
  // React will find the closest theme Provider above and use its value.
  // In this example, the current theme is "dark".
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```

### Before You Use Context
* Context terutama digunakan ketika beberapa data harus dapat diakses oleh banyak komponen pada nesting level yang berbeda. Terapkan sparingly karena membuat component reuse lebih sulit.
* Jika Anda hanya ingin menghindari passing beberapa props melalui banyak level, komposisi komponen seringkali merupakan solusi yang lebih sederhana daripada context.
* Misalnya, pertimbangkan komponen Page yang meneruskan user dan avatarSize prop beberapa level ke bawah sehingga deeply nested Link dan Avatar components dapat membacanya:
```
<Page user={user} avatarSize={avatarSize} />
// ... which renders ...
<PageLayout user={user} avatarSize={avatarSize} />
// ... which renders ...
<NavigationBar user={user} avatarSize={avatarSize} />
// ... which renders ...
<Link href={user.permalink}>
  <Avatar user={user} size={avatarSize} />
</Link>
```

* Salah satu cara untuk mengatasi masalah redundant untuk pass down user dan avatarSize props melalui banyak levels tanpa context adalah dengan pass down komponen Avatar itu sendiri sehingga intermediate components tidak perlu mengetahui tentang user atau properti avatarSize:
```
function Page(props) {
  const user = props.user;
  const userLink = (
    <Link href={user.permalink}>
      <Avatar user={user} size={props.avatarSize} />
    </Link>
  );
  return <PageLayout userLink={userLink} />;
}

// Now, we have:
<Page user={user} avatarSize={avatarSize} />
// ... which renders ...
<PageLayout userLink={...} />
// ... which renders ...
<NavigationBar userLink={...} />
// ... which renders ...
{props.userLink}
```

### API
#### React.createContext
> `const MyContext = React.createContext(defaultValue);` </br>
* Creates a Context object. Ketika React me-render sebuah komponen yang subscribe ke objek Context ini, ia akan membaca nilai konteks saat ini dari Provider terdekat yang cocok di atasnya dalam tree.
* Argumen defaultValue hanya digunakan bila komponen tidak memiliki Provider yang cocok di atasnya dalam tree. Nilai default ini dapat berguna untuk menguji komponen secara terpisah tanpa wrapping.

#### Context.Provider
> `<MyContext.Provider value={/* some value */}>` </br>
* Setiap objek Context dilengkapi dengan komponen Provider React yang memungkinkan penggunaan komponen untuk mengikuti perubahan konteks.
* Komponen Provider menerima prop nilai untuk diteruskan ke komponen konsumsi yang merupakan turunan dari Provider ini. Satu Provider dapat terhubung dengan banyak konsumen. Provider dapat nested untuk mengganti nilai lebih dalam di dalam tree.
* Semua konsumen yang merupakan turunan dari Provider akan merender ulang setiap kali prop nilai Provider berubah. Propagation dari Provider ke konsumen turunannya (termasuk .contextType dan useContext) tidak subject pada metode shouldComponentUpdate, sehingga konsumen diperbarui meskipun komponen ancestor melewatkan update.
* Perubahan ditentukan dengan membandingkan nilai baru dan lama menggunakan algoritme yang sama dengan Object.is.

#### Class.contextType
```
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* perform a side-effect at mount using the value of MyContext */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* render something based on the value of MyContext */
  }
}
MyClass.contextType = MyContext;
```
* Properti contextType pada sebuah class dapat diberi objek Context yang dibuat oleh React.createContext(). Menggunakan properti ini memungkinkan kita menggunakan nilai terdekat saat ini dari jenis Konteks tersebut menggunakan this.context. Kita dapat mereferensikan ini di salah satu lifecycle methods termasuk fungsi render.
```
class MyClass extends React.Component {
  static contextType = MyContext;
  render() {
    let value = this.context;
    /* render something based on the value */
  }
}
```

#### Context.Consumer
```
<MyContext.Consumer>
  {value => /* render something based on the context value */}
</MyContext.Consumer>
```
* Komponen React yang mengikuti perubahan konteks. Menggunakan komponen ini memungkinkan kita subscribe konteks di dalam komponen fungsi.
* Membutuhkan fungsi sebagai child. Fungsi menerima nilai konteks saat ini dan mengembalikan node React. Argumen nilai yang diteruskan ke fungsi akan sama dengan prop nilai dari Provider terdekat untuk konteks ini di atas dalam tree. Jika tidak ada Provider untuk konteks ini di atas, argumen value akan sama dengan defaultValue yang diteruskan ke createContext().

#### Context.displayName
* Objek konteks menerima properti string displayName. React DevTools menggunakan string ini untuk menentukan apa yang akan ditampilkan untuk konteksnya.
* Misalnya, komponen berikut akan muncul sebagai MyDisplayName di DevTools:
```
const MyContext = React.createContext(/* some value */);
MyContext.displayName = 'MyDisplayName';

<MyContext.Provider> // "MyDisplayName.Provider" in DevTools
<MyContext.Consumer> // "MyDisplayName.Consumer" in DevTools
```

## React Testing
> * __Apa itu Testing?__ </br>
> dalam tech jargon testing adalah memeriksa apakah kode kami memenuhi beberapa ekspektasi. </br>
> Misalnya: fungsi yang disebut "transformator" harus mengembalikan keluaran yang diharapkan dengan memberikan beberapa input.</br>
* long story short tests terbagi dalam tiga kategori utama:
  + unit testing
  + integration testing
  + UI testing

### Jest Tutorial: what is Jest?
* Jest adalah test runner JavaScript, yaitu library JavaScript untuk membuat, menjalankan, dan menyusun tes.
* Jest dikirimkan sebagai paket NPM, kita dapat menginstalnya di proyek JavaScript apa pun. Jest adalah salah satu test runner terpopuler saat ini, dan pilihan default untuk proyek React.

#### Setting up the project
* Create a new folder and initialize the project with:
```
mkdir getting-started-with-jest && cd $_
npm init -y
```
* install Jest
`npm i jest --save-dev`
* configure an NPM script for running our tests from the command line. Open up package.json and configure a script named test for running Jest:
```
"scripts": {
    "test": "jest"
  },
```
