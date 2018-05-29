# Views

- [Creating Views](#creating-views)
- [Passing Data To Views](#passing-data-to-views)
    - [Sharing Data With All Views](#sharing-data-with-all-views)
- [View Composers](#view-composers)

<a name="creating-views"></a>
## Creating Views

> {tip} Sedang mencari informasi mengenai Blade Template? Cek [Dokumentasi Blade](/docs/{{version}}/blade) lengkap untuk memulai. 

*Views* berisikan HTML yang disuplai oleh aplikasi anda dan dipisahkan oleh *controller* dari *presentation logic*. *Views* tersimpan di dalam direktori `resources/views`. Berikut ini contoh *view* yang sederhana:

    <!-- View stored in resources/views/greeting.blade.php -->

    <html>
        <body>
            <h1>Hello, {{ $name }}</h1>
        </body>
    </html>

Karena *view* diatas disimpan di `resources/views/greeting.blade.php`, kita bisa melakukan *return* dengan method `view` yang bersifat global, seperti:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

Seperti yang anda bisa lihat, argumen pertama dikirimkan ke `view` merupakan nama dari berkas *view* yang ada di dalam direktori `resources/views`, yakni `greeting.blade.php`. Lalu pada argumen kedua adalah sebuah array yang berisikan data yang nantinya bisa ditampilkan di dalam *view*. Pada contoh ini, kita mengirimkan variabel nama, yang mana nantinya akan ditampilkan oleh [Blade syntax](/docs/{{version}}/blade).

Tentu saja, *views* juga bisa diletakkan pada sub-direktori dari `resources/views`. Notasi "Dot" akan digunakan untuk mereferensi *views* yang berada pada sub. Untuk contihnya, jika *view* anda terletak pada `resources/views/admin/profile.blade.php`, anda bisa mereferensikannya seperti contoh:

    return view('admin.profile', $data);

#### Determining If A View Exists

Jika anda butuh mengecek apakah suatu *view* sudah ada atau belum, anda bisa menggunakan *facade* `View`. Method `exists` akan mengembalikan nilai `true` apabila *view* tersebut sudah ada. 

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

#### Creating The First Available View

Gunakan method `first`, jika anda ingin membuat sebuah *view* pertama yang sudah ada pada suatu array yang terdiri dari *view-view*. Ini berguna agar aplikasi anda memungkinkan agar *view-view* tersebut untuk dikostumisasi dan ditimpa: 

    return view()->first(['custom.admin', 'admin'], $data);
Dan tentu, jika anda ingin memanggil method tersebut melalui *facade* `View`:

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);

<a name="passing-data-to-views"></a>
## Passing Data To Views

Seperti yang anda lihat pada contoh sebelumnya, anda bisa mengirimkan data berupa array kepada *view*:

    return view('greetings', ['name' => 'Victoria']);
Ketika mengirimkan informasi seperti ini, data harus berupa array dan memiliki *key/value*. Di dalam view, anda bisa mengakses masing-masing nilai dengan key-nya sama seperti `<?php echo $key; ?>`. Adapun alternatif lain untuk mengirimkan data berupa array ke `view` adalah dengan method `with` yang bisa digunakan untuk mengirimkan data individu ke *view*:

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### Sharing Data With All Views

Kadang kala, mungkin anda butuh membagikan atau mengirimkan sejumlah data dengan semua *view* yang ada pada aplikasi anda, sehingga anda tidak perlu mendefinisikannya satu-satu. Anda bisa melakukan hal tersebut dengan method `share` dari *view* facade. Pada dasarnya, anda harus melakukan pemanggilan method `share` di dalam sebuah method `boot` yang ada di service provider. Anda bebas untuk menggunakan service provider apapun, entah itu `AppServiceProvider` ataupun buat service provider lain: 

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            View::share('key', 'value');
        }
    
        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="view-composers"></a>
## View Composers

*View Composer* merupakan sebuah *callback* atau *method kelas* yang akan dipanggil ketika *view* dirender oleh aplikasi. Jika anda memiliki data yang anda ingin hubungkan pada suatu *view* setiap kali *view* tersebut dirender, maka sebuah *view composer* akan sangat membantu anda dalam mengorganisasikan hal tersebut di tempat yang sama.

Untuk contoh, ayo *register* suatu *view composer* di dalam sebuah service provider. Kita akan gunakan facade `View` untuk mengakses informasi kontrak yang berada di `Illuminate\Contracts\View\Factory`. Perlu diingat, Laravel tidak mengatur direktori untuk *view composer*, anda bebas untuk meletakkan *view composer* anda dimana saja. Contoh, anda mungkin akan meletakkan *view composer* di direktori `app/Http/ViewComposers`: 

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;
    
    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            View::composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );
    
            // Using Closure based composers...
            View::composer('dashboard', function ($view) {
                //
            });
        }
    
        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

> {note} Ingat, jika anda membuat sebuah service provider baru yang berisi registrasi *view composer*, anda perlu menambahkan service provider ke dalam array `providers` yang ada di dalam berkas `config/app.php`. 

Sekarang kita telah memiliki *view composer* yang sudah diregistrasi, method `ProfileComposer@compose` akan dieksekusi setiap kali *view* `profile` dirender. Maka, ayo kita definisikan kelas composer:

    <?php

    namespace App\Http\ViewComposers;

    use Illuminate\View\View;
    use App\Repositories\UserRepository;
    
    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;
    
        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Dependencies automatically resolved by service container...
            $this->users = $users;
        }
    
        /**
         * Bind data to the view.
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

Tepat sebelum *view* dirender, method `compose` akan dipanggil di dalam suatu *instance* dari `Illuminate\View\View`. Anda bisa menggunakan method `with` untuk melakukan binding suatu data ke *view*.

> {tip} Semua *view composers* dikenali melalui [service container](/docs/{{version}}/container), jadi anda mungkin harus mengetik setiap ketergantungan yang anda butuhkan di dalam konstruktor pada composer. 

#### Attaching A Composer To Multiple Views

You may attach a view composer to multiple views at once by passing an array of views as the first argument to the `composer` method:

Anda mungkin bisa menyisipkan sebuah *view composer* pada banyak *view* dalam satu waktu dengan mengirimkan *view-view* berupa array sebagai argumen pertama dengan method `composer`: 

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );

Method `composer` juga menerima karakter `*` sebagai suatu *wildcard*, memungkinkan anda untuk menyisipkan sebuah composer ke semua *view* tanpa terkecuali. 

    View::composer('*', function ($view) {
        //
    });

#### View Creators

*View Creators* sangat mirip dengan *view composer*, tetapi, mereka dieksekusi langsung disaat suatu *view* dipanggil ketimbang menunggu saat suatu *view* akan dirender. Untuk meregistrasi suatu *view creator*, gunakan method `creator`: 

    View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
