## JavaScript Intermediate - Asynchronous - Fetch
### Fetching data dari server
> * __Apa itu Fetch?__ </br>
> Task lain yang sangat umum di situs web dan aplikasi modern adalah mengambil item data individual dari server untuk memperbarui bagian halaman web tanpa harus memuat seluruh halaman baru. </br>

### What is the problem here?
* Halaman web terdiri dari halaman HTML dan (biasanya) berbagai file lain, seperti `stylesheet`, `script`, dan `image`. Model dasar pemuatan halaman di Web adalah browser membuat satu atau lebih permintaan HTTP ke server untuk file yang diperlukan untuk menampilkan halaman, dan server merespons dengan file yang diminta. Jika mengunjungi halaman lain, browser meminta file baru, dan server meresponsnya.</br>
![traditional-loading](img/traditional_loading.jpg) </br>
* Cara ini bekerja dengan sangat baik untuk banyak situs. Tetapi pertimbangkan situs web yang sangat didorong oleh data. </br>
* > Misalnya, situs web perpustakaan. Kita bisa menganggap situs seperti ini sebagai interface pengguna ke database. Ini mungkin memungkinkan untuk mencari genre buku tertentu, atau mungkin menunjukkan rekomendasi untuk buku yang mungkin disukai, berdasarkan buku yang dipinjam sebelumnya. Saat melakukan ini, halaman perlu diperbarui dengan kumpulan buku baru untuk ditampilkan. Namun perhatikan bahwa sebagian besar konten halaman — termasuk item seperti `header` halaman, `sidebar`, dan `footer` — tetap sama. </br>
* Masalah di sini adalah kita harus mengambil dan memuat seluruh halaman, bahkan ketika kita hanya perlu memperbarui satu bagian saja. Ini tidak efisien dan dapat mengakibatkan pengalaman pengguna yang buruk. </br>
* Jadi, alih-alih model tradisional, banyak situs web menggunakan `API JavaScript` untuk meminta data dari server dan memperbarui konten halaman tanpa memuat halaman. Jadi ketika pengguna mencari produk baru, browser hanya meminta data yang diperlukan untuk memperbarui halaman — kumpulan buku baru untuk ditampilkan, misalnya.</br>
![fetch-update](img/fetch_update.jpg) </br>
* API utama di sini adalah `Fetch API`. Ini memungkinkan JavaScript yang berjalan di halaman untuk membuat permintaan HTTP ke server untuk mengambil sumber daya tertentu. Saat server menyediakannya, JavaScript dapat menggunakan data untuk memperbarui halaman, biasanya dengan menggunakan API manipulasi DOM. Data yang diminta seringkali berupa `JSON`, yang merupakan format yang baik untuk mentransfer data terstruktur, tetapi bisa juga berupa HTML atau hanya teks. </br>
* Ini adalah pola umum untuk situs berbasis data seperti `Amazon`, `YouTube`, `eBay`, dan sebagainya. Dengan model ini: </br>
  + Pembaruan halaman jauh lebih cepat dan kita tidak perlu menunggu halaman untuk refresh,, artinya situs terasa lebih cepat dan lebih responsif. </br>
  + Lebih sedikit data yang diunduh pada setiap pembaruan, yang berarti lebih sedikit `bandwidth` yang terbuang. Ini mungkin bukan masalah besar pada desktop pada koneksi broadband, tetapi ini adalah masalah besar pada perangkat seluler dan di negara-negara yang tidak memiliki layanan internet cepat di mana-mana. </br>
* Untuk mempercepat lebih jauh, beberapa situs juga menyimpan aset dan data di komputer pengguna saat pertama kali diminta, yang berarti bahwa pada kunjungan berikutnya mereka menggunakan versi lokal alih-alih mengunduh salinan baru setiap kali halaman pertama kali dimuat. Konten hanya dimuat ulang dari server ketika telah diperbarui. </br>

### The Fetch API
* __Fetching text content__ </br>
* Untuk contoh ini, kami akan meminta data dari beberapa file teks yang berbeda dan menggunakannya untuk mengisi area konten. </br>
* Serangkaian file ini akan bertindak sebagai database palsu kami; dalam aplikasi nyata, kita akan lebih cenderung menggunakan bahasa sisi server seperti `PHP`, `Python`, atau `Node` untuk meminta data kita dari database. Di sini, bagaimanapun, kami ingin tetap sederhana dan berkonsentrasi pada bagian sisi klien ini. </br>
* Untuk memulai contoh ini, buat salinan lokal dari `fetch-start.html` dan empat file teks — `verse1.txt`, `verse2.txt`, `verse3.txt`, dan `verse4.txt` — di direktori baru di komputer. Dalam contoh ini, kami akan mengambil bait puisi yang berbeda (yang mungkin Anda kenali) saat dipilih di `drop-down menu`. </br>
* Tepat di dalam elemen `<script>`, tambahkan kode berikut. Ini menyimpan referensi ke elemen `<select>` dan `<pre>` dan menambahkan listener ke elemen `<select>`, sehingga ketika pengguna memilih nilai baru, nilai baru diteruskan ke fungsi bernama `updateDisplay()` sebagai parameter. </br>
```
const verseChoose = document.querySelector('select');
const poemDisplay = document.querySelector('pre');

verseChoose.addEventListener('change', () => {
  const verse = verseChoose.value;
  updateDisplay(verse);
});
```

* Definisikan fungsi `updateDisplay()`. Pertama-tama, letakkan kode berikut di bawah blok kode sebelumnya, ini adalah cangkang kosong dari fungsi tersebut.</br>
```
function updateDisplay(verse) {

}
```
* Mulai function dengan membuat URL relatif yang menunjuk ke file teks yang ingin dimuat, karena akan dibutuhkan nanti. Nilai elemen `<select>` setiap saat sama dengan teks di dalam `<option>` yang dipilih (kecuali jika ditentukan nilai yang berbeda dalam atribut value) — jadi misalnya `Verse 1`. File verse text  yang sesuai adalah `verse1.txt`, dan berada di direktori yang sama dengan file HTML, oleh karena itu hanya nama file yang akan digunakan.
* Server web cenderung peka terhadap huruf besar/kecil, dan nama file tidak memiliki spasi di dalamnya. Untuk mengonversi `"Verse 1"` menjadi `"verse1.txt"` kita perlu mengonversi V ke huruf kecil, menghapus spasi, dan menambahkan `.txt` di bagian akhir. Ini dapat dilakukan dengan `replace()`, `toLowerCase()`, dan rangkaian string. 
* Tambahkan baris berikut di dalam fungsi `updateDisplay()` : </br>
```
verse = verse.replace(' ', '').toLowerCase();
const url = `${verse}.txt`;
```  
* Hasil akhir `Fetch API`: </br>
```
// Call `fetch()`, passing in the URL.
fetch(url)
  // fetch() returns a promise. When we have received a response from the server,
  // the promise's `then()` handler is called with the response.
  .then((response) => {
    // Our handler throws an error if the request did not succeed.
    if (!response.ok) {
      throw new Error(`HTTP error: ${response.status}`);
    }
    // Otherwise (if the response succeeded), our handler fetches the response
    // as text by calling response.text(), and immediately returns the promise
    // returned by `response.text()`.
    return response.text();
  })
  // When response.text() has succeeded, the `then()` handler is called with
  // the text, and we copy it into the `poemDisplay` box.
  .then((text) => poemDisplay.textContent = text)
  // Catch any errors that might happen, and display a message
  // in the `poemDisplay` box.
  .catch((error) => poemDisplay.textContent = `Could not fetch verse: ${error}`);

```
### The XMLHttpRequest API
* Terkadang, terutama dalam kode yang lebih lama, kita akan melihat API lain yang disebut `XMLHttpRequest` (sering disingkat sebagai "XHR") yang digunakan untuk membuat permintaan HTTP. 
* Ini mendahului Fetch, dan benar-benar merupakan API pertama yang digunakan secara luas untuk mengimplementasikan AJAX. ini adalah API yang lebih sederhana dan memiliki lebih banyak fitur daripada XMLHttpRequest.
```
  const request = new XMLHttpRequest();

try {
  request.open('GET', 'products.json');

  request.responseType = 'json';

  request.addEventListener('load', () => initialize(request.response));
  request.addEventListener('error', () => console.error('XHR error'));

  request.send();

} catch (error) {
  console.error(`XHR error ${request.status}`);
}

```
* Ada lima tahap untuk ini:
  + Buat objek `XMLHttpRequest` baru.
  + Panggil metode `open()` untuk menginisialisasinya.
  + Tambahkan `event listener` ke `load event`, yang diaktifkan saat respons berhasil diselesaikan. Di listener kita memanggil `initialize()` dengan data.
  + Tambahkan `event listener` ke error event, yang diaktifkan saat request mengalami kesalahan
  + Kirim request.
* Kita juga harus wrap semuanya dalam blok `try...catch`, untuk menangani kesalahan yang dilontarkan oleh `open()` atau `send()`.

## JavaScript Intermdiate - Asynchronous - Async Await
### Resolving JavaScript Promises
> Saat menggunakan JavaScript `async...await`, beberapa operasi asinkron dapat berjalan secara bersamaan. Jika nilai yang diselesaikan diperlukan untuk setiap promise yang dimulai, `Promise.all()` dapat digunakan untuk mengambil nilai yang diselesaikan, menghindari pemblokiran yang tidak perlu. </br>
```
let promise1 = Promise.resolve(5);
let promise2 = 44;
let promise3 = new Promise(function(resolve, reject) {
  setTimeout(resolve, 100, 'foo');
});

Promise.all([promise1, promise2, promise3]).then(function(values) {
  console.log(values);
});
// expected output: Array [5, 44, "foo"]
```

### Creating async Function
> Fungsi JavaScript asinkron dapat dibuat dengan kata kunci async sebelum nama fungsi, atau `before ()` saat menggunakan sintaks arrow function. Fungsi async return promise. </br>
```
function helloWorld() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('Hello World!');
    }, 2000);
  });
}

const msg = async function() { //Async Function Expression
  const msg = await helloWorld();
  console.log('Message:', msg);
}

const msg1 = async () => { //Async Arrow Function
  const msg = await helloWorld();
  console.log('Message:', msg);
}

msg(); // Message: Hello World! <-- after 2 seconds
msg1(); // Message: Hello World! <-- after 2 seconds
```


### Async Await Promises
> Sintaks `async...await` di ES6 menawarkan cara baru menulis kode yang lebih mudah dibaca dan terukur untuk handle promises. Ini menggunakan fitur yang sama yang sudah ada di JavaScript. </br>
```
function helloWorld() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('Hello World!');
    }, 2000);
  });
}

async function msg() {
  const msg = await helloWorld();
  console.log('Message:', msg);
}

msg(); // Message: Hello World! <-- after 2 seconds
```

### Using async await syntax
> Membangun satu atau lebih `promises` atau `calls` tanpa `await` dapat memungkinkan beberapa fungsi asinkron dijalankan secara bersamaan. Melalui pendekatan ini, sebuah program dapat memanfaatkan konkurensi, dan tindakan asinkron dapat dimulai dalam fungsi asinkron. Karena menggunakan kata kunci `await` menghentikan eksekusi fungsi asinkron, setiap fungsi asinkron dapat ditunggu setelah nilainya diperlukan oleh logika program. </br>

### JavaScript async…await advantage
> Sintaks `async...await` JavaScript memungkinkan beberapa promise untuk dimulai dan kemudian diselesaikan untuk nilai bila diperlukan selama eksekusi program. Sebagai alternatif untuk fungsi chaining `.then()`, ia menawarkan pemeliharaan kode yang lebih baik dan kode sinkron yang sangat mirip. </br>

### Async Function Error Handling
> Fungsi asinkron JavaScript menggunakan pernyataan `try...catch` untuk penanganan kesalahan. Metode ini memungkinkan penanganan kesalahan bersama untuk kode sinkron dan asinkron. </br>
```
let json = '{ "age": 30 }'; // incomplete data

try {
  let user = JSON.parse(json); // <-- no errors
  alert( user.name ); // no name!
} catch (e) {
  alert( "Invalid JSON data!" );
}
```

### The aysnc and await Keywords
> Sintaks JavaScript `async…await` ES6 menawarkan cara baru untuk menulis kode yang lebih mudah dibaca dan skalabel untuk `handle promises`. Fungsi asinkron JavaScript dapat berisi pernyataan yang didahului oleh operator `await`. Operan await adalah sebuah promise. Pada ekspresi await, eksekusi fungsi async dijeda dan await promise operan untuk diselesaikan. Operator await mengembalikan nilai promise yang diselesaikan. Operand await hanya dapat digunakan di dalam fungsi async. </br>
```
function helloWorld() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('Hello World!');
    }, 2000);
  });
}

async function msg() {
  const msg = await helloWorld();
  console.log('Message:', msg);
}

msg(); // Message: Hello World! <-- after 2 seconds
```
  
## Git & Github Lanjutan
> * __Apa itu GIT?__ </br>
> GIT adalah Tools untuk programmer. GIT sebagai Version Control System </br>
> > * __Apa itu Version Control System?__ </br>
> > Tugasnya adalah mencatat setiap perubahan pada File (termasuk code yang kita buat) pada suatu proyek baik dikerjakan secara individu maupun tim. </br>
> Git adalah aplikasi yang dapat melacak setiap perubahan yang terjadi pada suatu folder atau file. </br>
> Git biasanya digunakan oleh para programmer sebagai tempat penyimpanan file pemrograman mereka, karena lebih efektif. </br>
> File-file yg disimpan menggunakan git akan terlacak seluruh perubahannya, termasuk siapa yang mengubah. </br>

### ‘WHY’ should use GIT and Github?
* Dengan menggunakan GIT dan Github, kamu akan bisa bekerja dalam sebuat tim. Tujuan besarnya adalah kamu bisa berkolaborasi mengerjakan proyek yang sama tanpa harus repot copy paste folder aplikasi yang terupdate.
* Kamu juga tidak perlu menunggu rekan dalam satu tim kamu menyelesaikan suatu program dahulu untuk berkolaborasi. Kamu bisa membuat file didalam projek yang sama atau membuat code di file yang sama dan menyatukannya saat sudah selesai.

### Instalasi GIT
* Download dan jalankan hasil download GIT kamu seperti instal aplikasi pada umumnya
  + Setup Awal
    - `git config --global user.name "username"`
    - `git config --global user.email contoh@email.com`
  + Cek apakah instalasi berhasil
    - `git --version`
  + Cek apakah setup berhasil
    - `git config --list`
> __WARNING:__ email yang disetup HARUS SAMA dengan yang digunakan pada GITHUB </br>

### Repository GIT
* Repository adalah direktori projek yang kita buat.
  + Membuat Repository
    - `git init nama-projek`
  + jika folder sudah ada sebelumnya
    - `git init .`

#### GIT STATUS
* 3 Kondisi File pada GIT
  + __Modified__ adalah kondisi dimana revisi atau perubahan sudah dilakukan, tetapi belum ditandai (untracked) dan belum disimpan dalam version control.
  + __Staged__ adalah kondisi dimana revisi sudah ditandai (modified) namun belum disimpan di version control.
  + __Commit/committed__ adalah kondisi dimana revisi sudah disimpan pada version control.

#### GIT ADD
* Setelah cek status dengan ‘git status’, selanjutnya kita ubah status ‘untrackted file’ dan ‘unmodified’ menjadi modified. Gunakan git add
  + `git add index.html` atau `git add .`
  + Jika untracked file nya dalam jumlah besar gunakan `git add .`

#### GIT COMMIT
* Lakukan ‘git commit’ untuk save perubahan pada version control
  + `git commit -m "your comment"`

#### Revisi Kedua pada GIT
* __Perhatikan!__ Kondisi sudah tidak di ‘untrackted files’ tapi ‘modified files’

#### GIT LOG
* Dari dua revisi yang sudah dilakukan kita dapat melihat catatal log dari revisi - revisi tersebut dengan menggunakan perintah berikut ini:
  + `git log`
  + Untuk git log yang lebih pendek, bisa menggunakan perintah berikut ini: `git log --oneline`
* Melihat log dari berbagai sisi.
  + Melihat log dari nomor version
  + Melihat log file tertentu
  + Melihat log berdasarkan author

#### GIT REVERT
* GIT Revert akan membatalkan semua perubahan yang ada tanpa menghapus commit terakhir. Jika menggunakan GIT Reset, commit terakhir akan hilang.
  + `git revert -n <nomor commit>`

#### GIT CHECKOUT
* Kondisi file sudah pada kondisi ‘Modified’. Selanjutnya kita lakukan proses yg sama sebelumnya. Menggunakan ‘git checkout’
* Kita bisa mengembalikan commit untuk semua file
* Membatalkan Perubahan - Sudah Stagged namun Belum Commited
  + `git checkout nama.file`
* Jika ingin mengembalikan commit jauh ke bawah. Misal kita ingin kembali pada 3 commit sebelumnya
  + `git checkout HEAD-3 nama.file`

#### GIT RESET
* Kita bisa mengembalikan commit hanya pada file tertentu  
  + `git reset nama.file`

#### GIT BRANCH
* Fitur yang WAJIB digunakan jika berkolaborasi dengan developer atau dalam tim
* Untuk menghindari conflict code yang dikembangkan. Kita tidak boleh berkolaborasi dalam project di satu branch yang sama!
* Tidak boleh mengganggu branch ‘master’ yang sudah terupdate
* Untuk membuat branch, gunakan perintah berikut ini
  + `git branch <branch>`
* Melihat List Branch
  + `git branch`
* Pindah ke branch tertentu
  + `git checkout <branch tertentu>`
* Delete Branch
  + `git branch -d <branch>`

#### GIT MERGE
* Setelah membuat branch baru, lalu lakukan commit. Saatnya kita menyatukan pekerjaan ke master file/branch utama yaitu branch MASTER
* Untuk menyatukan branch cabang fitur yang telah kita kembangkan. Gunakan perintah seperti berikut ini:
  + Kita harus checkout dahulu ke branch master : `git checkout master`
  + Lalu lakukan merge : `git merge nama_file`

## Responsive Web Design
> * __Apa itu Responsive Web Design?__ </br>
> Responsive Web Design adalah bertujuan membuat website kita dapat diakses dalam device apapun </br>

### Setting up Chrome Dev Tools
* Setiap developer website wajib menggunakan tools bawaan dari setiap browser yang memudahkan proses development website </br>
* Pada browser chrome biasa disebut dengan Chrome Dev Tools. </br>

### Add Viewport in HTML
`<meta name="viewport" content="width=device-width, initial-scale=1.0">`

### Use Max-width element
`<img style="max-width: 100%;" src="image.jpg" alt="image">`

### Media Query
* Jenis Media Query
  + Media query untuk responsive web design umumnya hanya menggunakan 2 jenis media query.
    - min-width
    - max-width
* Media Query digunakan untuk membuat beberapa style tergantung pada jenis device.
* Ada 2 cara/pattern dalam menggunakan media query
  + Membuat file css berbeda untuk masing-masing device
  + Menggabungkan 1 file CSS untuk styling berbagai device

### Breakpoint
* Perubahan yang terjadi pada tampilan saat berganti device atau ukuran width disebut breakpoint

### Complex Breakpoint Media Query
* Jika kita menginginkan tampilan yang ingin diterapkan pada range ukuran device tertentu, kita bisa membuatnya menjadi range media query.

### Important Notes
* Tidak ada ukuran baku besaran width dan berapa banyak breakpoint yang harus dilakukan
* Responsive web design dilakukan sesuai kebutuhan konten kita. Jika konten yang ditampilkan sudah tidak bisa diakses atau sulit dilihat pada width tertentu, sudah saatnya kita menggunakan media query.

## Bootstrap 5
### Get started with Bootstrap
* Bootstrap adalah toolkit frontend yang kuat dan penuh fitur. Bangun apa pun—dari prototipe hingga produksi—dalam hitungan menit.

### Quick start
* Mulailah dengan menyertakan CSS dan JavaScript siap produksi Bootstrap melalui CDN tanpa perlu langkah pembuatan apa pun
  + Buat file index.html baru di root proyek Anda. Sertakan tag `<meta name="viewport">` juga untuk perilaku responsif yang tepat di perangkat seluler.
  ```
  <!doctype html>
  <html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Bootstrap demo</title>
  </head>
  <body>
    <h1>Hello, world!</h1>
  </body>
  </html>
  ```
  + Sertakan CSS dan JS Bootstrap. Tempatkan tag `<link>` di `<head>` untuk CSS, dan tag `<script>` untuk bundel JavaScript (termasuk Popper untuk memposisikan dropdown, popper, dan tooltips) sebelum penutup `</body>`.
  ```
  <!doctype html>
  <html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Bootstrap demo</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-Zenh87qX5JnK2Jl0vWa8Ck2rdkQ2Bzep5IDxbcnCeuOxjzrPF/et3URy9Bv1WTRi" crossorigin="anonymous">
  </head>
    <body>
      <h1>Hello, world!</h1>
      <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-OERcA2EqjJCMA+/3y+gxIOqMEjwtxJY7qPCqsdltbNJuaOe923+mo//f6V8Qbsw3" crossorigin="anonymous"></script>
 
    </body>
  </html>
  ```
  Atau juga dapat memasukkan Popper dan JS secara terpisah. Jika tidak berencana menggunakan dropdown, popover, atau tooltips, hemat beberapa kilobyte dengan tidak menyertakan Popper.
  ```
  <script src="https://cdn.jsdelivr.net/npm/@popperjs/core@2.11.6/dist/umd/popper.min.js" integrity="sha384-oBqDVmMz9ATKxIep9tiCxS/Z9fNfEXiDAYTujMAeBAsjFuCZSmKbSSUnQlmh/jp3" crossorigin="anonymous"></script>
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/js/bootstrap.min.js" integrity="sha384-IDwe1+LCz02ROU9k972gdyvl+AESN10+x7tBKgc9I5HFtuNz0wWnPclzo6p9vxnk" crossorigin="anonymous"></script>
  ```
  + __Hello, world!__ Buka halaman di browser untuk melihat halaman Bootstrapped. Sekarang dapat mulai membangun dengan Bootstrap dengan membuat layout sendiri, menambahkan komponen, dan memanfaatkan contoh resmi bootstrap.

### CDN links 
* Sebagai referensi, primary CDN links :
  + CSS : (https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/css/bootstrap.min.css)
  + JS : (https://cdn.jsdelivr.net/npm/bootstrap@5.2.2/dist/js/bootstrap.bundle.min.js)

### JS components
* Alerts for dismissing
* Buttons for toggling states and checkbox/radio functionality
* Carousel for all slide behaviors, controls, and indicatorsCollapse for toggling visibility of content
* Dropdowns for displaying and positioning (also requires Popper)
* Modals for displaying, positioning, and scroll behavior
* Navbar for extending our Collapse and Offcanvas plugins to implement responsive behaviors
* Navs with the Tab plugin for toggling content panes
* Offcanvases for displaying, positioning, and scroll behavior
* Scrollspy for scroll behavior and navigation updates
* Toasts for displaying and dismissing
* Tooltips and popovers for displaying and positioning (also requires Popper)]

### HTML5 doctype
* Bootstrap membutuhkan penggunaan doctype HTML5. Tanpa itu, akan terlihat beberapa gaya yang funky dan tidak lengkap.
```
<!doctype html>
<html lang="en">
  ...
</html>
```

### Responsive meta tag
* Bootstrap dikembangkan terlebih dahulu untuk seluler, sebuah strategi di mana mengoptimalkan kode untuk perangkat seluler terlebih dahulu dan kemudian meningkatkan komponen seperlunya menggunakan kueri media CSS. Untuk memastikan rendering yang tepat dan zoom sentuh untuk semua perangkat, tambahkan tag meta area pandang responsif ke `<head>`.
`<meta name="viewport" content="width=device-width, initial-scale=1">`

### Box-sizing 
* Untuk ukuran yang lebih mudah di CSS, kami mengganti nilai `box-sizing` global dari `content-box` ke `border-box`. Ini memastikan padding tidak memengaruhi lebar akhir yang dihitung dari suatu elemen, tetapi dapat menyebabkan masalah dengan beberapa perangkat lunak pihak ketiga seperti Google Maps dan Google Custom Search Engine.
* Pada kesempatan langka Anda perlu menimpanya, gunakan sesuatu seperti berikut:
```
.selector-for-some-widget {
  box-sizing: content-box;
}
```
* Dengan cuplikan di atas, nested elements-termasuk konten yang dihasilkan melalui `::before` dan `::after`—semuanya akan mewarisi `box-sizing` yang ditentukan untuk `.selector-for-some-widget` tersebut.

### Reboot
* Untuk rendering lintas-browser yang lebih baik, digunakan Reboot untuk memperbaiki ketidakkonsistenan di seluruh browser dan perangkat sambil memberikan pengaturan ulang yang sedikit lebih sesuai untuk elemen HTML umum.
