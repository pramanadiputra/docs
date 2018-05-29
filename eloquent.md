# Eloquent: Getting Started

- [Introduction](#introduction)
- [Defining Models](#defining-models)
    - [Eloquent Model Conventions](#eloquent-model-conventions)
- [Retrieving Models](#retrieving-models)
    - [Collections](#collections)
    - [Chunking Results](#chunking-results)
- [Retrieving Single Models / Aggregates](#retrieving-single-models)
    - [Retrieving Aggregates](#retrieving-aggregates)
- [Inserting & Updating Models](#inserting-and-updating-models)
    - [Inserts](#inserts)
    - [Updates](#updates)
    - [Mass Assignment](#mass-assignment)
    - [Other Creation Methods](#other-creation-methods)
- [Deleting Models](#deleting-models)
    - [Soft Deleting](#soft-deleting)
    - [Querying Soft Deleted Models](#querying-soft-deleted-models)
- [Query Scopes](#query-scopes)
    - [Global Scopes](#global-scopes)
    - [Local Scopes](#local-scopes)
- [Events](#events)
    - [Observers](#observers)

<a name="introduction"></a>
## Introduction

ORM Eloquent memberikan pengembang Laravel kemudahan dan kerapian dalam pengimplementasian ActiveRecord yang simpel saat anda berkerja dengan database. Setiap tabel pada database memiliki representasi sebuah "Model" yang digunakan untuk berinteraksi dengan tabel tersebut. Model - model memungkinkanmu untuk melakukan kueri data pada tabel - tabel, serta memasukkan data baru ke tabel - tabel. 

Sebelum memulai, ada baiknya untuk mengkonfigurasi koneksi database di `config/database.php`. Untuk informasi lebih lanjut, cek [dokumentasi](/docs/{{version}}/database#configuration).

<a name="defining-models"></a>
## Defining Models

Untuk memulai, ayo buat sebuah model Eloquent. Biasanya model - model akan diletakkan di dalam direktori `app`, namun tentu saja anda bebas untuk meletakkannya dimana saja asalkan dapat di *loading* oleh pengaturan `composer.json` anda. Semua model - model Eloquent menurunkan sifat dari kelas `Illuminate\Database\Eloquent\Model`. 

Cara paling simpel dalam membuat sebuah model adalah dengan menjalankan  [perintah artisan](/docs/{{version}}/artisan) : `make:model`

    php artisan make:model Flight
Jika anda langsung ingin meng-generasi sebuah [migrasi database](/docs/{{version}}/migrations) disaat anda membuat sebuah model, anda bisa gunakan opsi `--migration` atau cukup `-m`. 

    php artisan make:model Flight --migration

    php artisan make:model Flight -m

<a name="eloquent-model-conventions"></a>
### Eloquent Model Conventions

Sekarang, ayo lihat contoh model `Flight`, yang akan kita gunakan untuk menerima dan mengirimkan data ke tabel `flights` di database.

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }

#### Table Names

Perhatikan bahwa kita tidak memberitahu Eloquent tabel database yang mana yang kita maksud untuk model `Flight`. Secara bawaan, Eloquent akan langsung otomatis mengetahui tabel database mana yang dimaksud dengan melakukan "snake case", pada setiap nama tabel di database karena pada umumnya nama tabel akan selalu bersifat *plural* yakni memiliki akhiran **s**. Pada contoh ini Eloquent akan berasumsi bahwa model `Flight` merupakan representasi tabel database `Flights`.  Tentu saja anda juga bisa mengkostumisasi nama tabel yang berepresentasi di database dengan model tersebut. 

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The table associated with the model.
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

#### Primary Keys

Eloquent akan berasumsi bahwa setiap tabel memiliki sebuah *primary key* yang dinamakan `id`. Anda bisa mengubahnya dengan mendefinisikan `protected $primaryKey`. 

Dan juga, Eloquent berasumsi bahwa sebuah primary key akan bersifat *increment* atau selalu bertambah nilainya, yang mana secara bawaan Eloquent akan menganggap sebuah primary key akan bertipe `int`. Jika anda ingin Eloquent  menganggap bahwa primary key anda merupakan *non increment* ataupun bukan bertipe `int`, anda dapat mengubahnya dengan mendefinisikan `public $incrementing = false`. Dan untuk primary key yang bukan bertipe `int`, anda harus mendefinisikan `protected $keyType` menjadi `string` pada model anda. 

#### Timestamps

Secara bawaan, Eloquent menganggap kolom `created_at` dan `updated_at` sudah pasti ada pada tabel - tabel di database anda. Jika anda tidak ingin kolom - kolom tersebut diatur oleh Eloquent, set `public $timestamps = false` di model anda.

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Indicates if the model should be timestamped.
         *
         * @var bool
         */
        public $timestamps = false;
    }

Jika anda ingin mengkostumisasi format pada timestamps anda, set `$dateFormat` pada model anda. Atribut ini menentukan bagaimana atribut tanggal akan disimpan ke database, begitu juga dengan format yang diserialisasi menjadi array atau JSON.

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

Jika anda mengkostumisasi nama dari kolom - kolom untuk menyimpan timestamps, anda mungkin harus mendefinisikan `CREATED_AT` dan `UPDATED_AT` sebagai konstanta pada model anda. 

    <?php

    class Flight extends Model
    {
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'last_update';
    }

#### Database Connection

Secara bawaan, semua model Eloquent akan menggunakan koneksi database yang sudah dikonfigurasikan di aplikasi anda. Jika anda ingin mengkonfigurasikan koneksi yang berbeda untuk sebuah model, set atribut `$connection`.

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The connection name for the model.
         *
         * @var string
         */
        protected $connection = 'connection-name';
    }

<a name="retrieving-models"></a>
## Retrieving Models

Seketika anda sudah membuat sebuah model dan mengasosiasikannya dengan tabel database, model anda sudah siap untuk menerima data dari tabel database anda. Setiap model Eloquent memiliki fitur yang sama powerfulnya dengan [query builder](/docs/{{version}}/queries) yang memungkinkan manajemen kueri dengan asosiasi model. Contohnya: 

    <?php

    use App\Flight;

    $flights = App\Flight::all();

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### Adding Additional Constraints

Setiap method `all` pada Eloquent akan memberikan semua data yang ada pada sebuah tabel. Dan juga karena model Eloquent turut mengindikasikan penggunaan [query builder](/docs/{{version}}/queries), maka anda bisa menambahkan suatu kosntrain pada kueri - kueri anda, dan gunakan `get` untuk menerima pengambilan data:

    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> {tip} Karena model - model Eloquent juga merupakan query builders, anda bisa saja mengecek semua method - method yang ada pada [query builder](/docs/{{version}}/queries). Anda bisa menggunakan salah satu dari keseluruhan method - method tersebut pada kueri Eloquent anda. 

<a name="collections"></a>
### Collections

Untuk semua method - method seperti `all` dan `get` yang menerima hasil data secara jamak, merupakan sebuah *instance* dari kelas `Illuminate\Database\Eloquent\Collection`. Maka itu [method - method yang berguna](/docs/{{version}}/eloquent-collections#available-methods) dari kelas `Collection` dapat anda gunakan pada model Eloquent anda:

    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });

Dan tentunya, anda juga bisa melakukan perulangan pada koleksi seperti sebuah array: 

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="chunking-results"></a>
### Chunking Results

Jika anda butuh melakukan pemrosesan pengambilan data yang berupa ribuan, gunakanlah perintah `chunk`. Method `chunk` akan menerima data - data tersebut untuk model Eloquent, dan mengirimkannya pada `Closure` yang terkait untuk melakukan pemrosesan. Penggunaan `chunk` untuk menambil data yang banyaknya ribuan akan lebih menghemat penggunaan memori dan kinerja server.

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

Argumen pertama yang dikirimkan ke method adalah angka data yang anda harapkan untuk terima per "chunk". Dan *Closure* yang bersangkutan mengirimkan argumen kedua yang akan memanggil masing-masing chunk untuk diterima dari database. Sebuah kueri database akan diseksekusi untuk menerima data chunk tersebut.

#### Using Cursors

Menggunakan method `cursor` akan memungkinkan anda untuk melakukan iterasi pada barisan data di database anda, yang mana kemudian akan mengeksekusi satu kueri. Ketika memproses data dalam jumlah banyak, method `cursor` akan sangat membantu dalam mengurangi penggunaan memori anda:

    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }

<a name="retrieving-single-models"></a>
## Retrieving Single Models / Aggregates

Tentu saja, selain pengambilan data dalam jumlah lebih dari satu, anda bisa menerima data tunggal menggunakan `find` atau `first`. Kedua method ini akan mengembalikan *instance* dari model, daripada koleksi model - model. 

    // Retrieve a model by its primary key...
    $flight = App\Flight::find(1);
    
    // Retrieve the first model matching the query constraints...
    $flight = App\Flight::where('active', 1)->first();

Anda bisa juga menggunakan method `find` untuk array dari beberapa primary key, yang akan mengembalikan koleksi dari record-record tersebut.

    $flights = App\Flight::find([1, 2, 3]);

#### Not Found Exceptions

Kadang-kala anda mungkin ingin melemparkan sebuah eksepsi jika suatu data tidak ditemukan. Biasanya berguna untuk route dan kontroller. Method - method `findOrFail` dan `firstOrFail` akan mengambil data dari database; tetapi, jika tidak ditemukan maka akan melemparkan eksepsi dari kelas `Illuminate\Database\Eloquent\ModelNotFoundException`: 

    $model = App\Flight::findOrFail(1);

    $model = App\Flight::where('legs', '>', 100)->firstOrFail();
Jika eksepsi tidak tertangkap, maka respon HTTP `404` yang akan dikirimkan otomatis kepada pengguna. 

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### Retrieving Aggregates

Jika anda menggunakan method - method `count`, `sum`, `max`, dan [method agregat](/docs/{{version}}/queries#aggregates) yang disediakan oleh [query builder](/docs/{{version}}/queries). Akan mengembalikan nilai skalar ketimbang *instance* sebuah model:

    $count = App\Flight::where('active', 1)->count();

    $max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## Inserting & Updating Models

<a name="inserts"></a>
### Inserts

Untuk membuat sebuah record baru di database, buatlah sebuah *instance* baru dari suatu model, dan set atribut-atribut pada model, lalu panggil method `save`: 

    <?php

    namespace App\Http\Controllers;

    use App\Flight;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class FlightController extends Controller
    {
        /**
         * Create a new flight instance.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate the request...
    
            $flight = new Flight;
    
            $flight->name = $request->name;
    
            $flight->save();
        }
    }

Pada contoh ini, kita kaitkan parameter `name` dari request HTTP yang datang pada atribut `name` di dalam model `App\Flight`. Ketika kita memanggil method `save`, sebuah record akan dikirimkan ke database. timestamps `created_at` dan `updated_at` akan otomatis diset saat `save` dipanggil, jadi tidak perlu set secara manual. 

<a name="updates"></a>
### Updates

Method `save` juga dapat digunakan untuk melakukan update pada model - model yang sudah ada di database. Untuk mengupdate sebuah model, anda harus mencari record tersebut terlebih dulu, lalu set atribut apa yang anda ingin update, dan panggil method `save` untuk menyimpan update tersebut. Dan juga timestamp `updated_at` akan otomatis diperbarui, jadi tidak perlu secara manual mengubah nilainya.

    $flight = App\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

#### Mass Updates

Updates can also be performed against any number of models that match a given query. In this example, all flights that are `active` and have a `destination` of `San Diego` will be marked as delayed:

Update juga bisa dilakukan secara masal, pada contoh ini, semua penerbangan yang `active` dan untuk `destination` pada daerah `San Diego` akan diubah menjadi `delayed`:

    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

Method `update` mengarapkan suatu kolom dan juga nilainya untuk diupdate.

> {note} Ketika melakukan update masal melalui Eloquent, event `saved` dan `updated` tidak akan dijalankan untuk memperbarui model - model. Ini karena setiap model sebenarnya tidak terpanggil saat update masal dilakukan.

<a name="mass-assignment"></a>
### Mass Assignment

Anda bisa menggunakan method `create` untuk menyimpan atau membuat model baru hanya dengan satu baris. Model yang disisipkan akan dikembalikan dari method tersebut. Tetapi, sebelum melakukannya, anda perlu menentukan apakah atribut pada model bersifat `fillable` atau `guarded`, karena pada dasarnya semua model - model Eloquent mencegah pengisian-data secara masal.

Pemberian-data secara masal atau *mass-assignment* memiliki kerentanan disaat suatu pengguna mengirimkan paramter HTTP yang tidak diinginkan melalui sebuah request, dan parameter tersebut mengubah suatu kolom di database anda. Sebagai contoh, pengguna jahat mungkin akan mengirimkan parameter `is_admin` melalui request HTTP, yang akan terlampau pada model anda dengan method `create`, memungkinkan pengguna jahat tersebut membuat akunnya menjadi admin. 

Lantas, untuk memulai, anda harus mendefisinikan atau menentukan atribut-atribut apa saja yang boleh untuk dilakukan *mass assignment*. Dengan menyisipkan `$fillable` pada model. Untuk contoh, ayo jadikan atribut `name` pada model `Flight` sebagai atribut yang bisa *mass assignment*:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

Ketika kita sudah mendefinisikan atribut sebagai *mass assigment*, kita bisa gunakan method `create` untuk menyimpan sebuah record baru di database. Method `create` akan mengembalikan *instance* dari model. 

    $flight = App\Flight::create(['name' => 'Flight 10']);
Jika anda sudah punya *instance* dari suatu model, anda bisa gunakan method `fill` untuk mempolasikan atribut-atribut sebagai array:

    $flight->fill(['name' => 'Flight 22']);

#### Guarding Attributes

Ketika `$fillable` merupakan atribut seperti `white list` atau atribut yang boleh dianggap sebagai `mass assignment`, anda juga bisa memilih suatu atribut sebagai kebalikannya dengan menggunakan `$guarded`. Atribut yang anda ingin lindungi harus anda isi dengan `$guarded`, dan hanya atribut tersebut saja yang tidak `mass assignment`, atribut lain akan menjadi `mass assignment`. Tentu saja anda harus memilih menggunakan `fillable` atau `guarded`, tidak boleh menggunakan keduanya disaat bersamaan. Seperti pada contoh di bawah, semua atribut **terkecuali untuk** **`price`** akan  bersifat *mass assignment*.

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that aren't mass assignable.
         *
         * @var array
         */
        protected $guarded = ['price'];
    }

Jika anda ingin membuat semua atribut sebagai *mass assignment*, silahkan berikan properti `$guarded` dengan array kosong:

    /**
     * The attributes that aren't mass assignable.
     *
     * @var array
     */
    protected $guarded = [];

<a name="other-creation-methods"></a>
### Other Creation Methods

#### `firstOrCreate`/ `firstOrNew`

Ada dua method yang anda mungkin gunakan untuk membuat data pada atribut *mass assignment*: `firstOrCreate` dan `firstOrNew`. Method `firstOrCreate` akan mencoba untuk mencari data pada suatu database, jika tidak ditemukan maka data tersebut akan dimasukkan ke database. Sebuah data akan dimasukkan dengan parameter pertama, dan yang lainnya pada parameter kedua. 

Method `firstOrNew` sama seperti `firstOrCreate` yang akan mencoba untuk mencari data terlebih dahulu, jika tidak ditemukan, maka method ini akan mengembalikan sebuah *instance* baru dari model tersebut. Perhatikan bahwa model yang dikembalikan oleh `firstOrNew` tidak ada di database, untuk itu harus method `save` harus dipanggil:

    // Retrieve flight by name, or create it if it doesn't exist...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);
    
    // Retrieve flight by name, or create it with the name and delayed attributes...
    $flight = App\Flight::firstOrCreate(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );
    
    // Retrieve by name, or instantiate...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);
    
    // Retrieve by name, or instantiate with the name and delayed attributes...
    $flight = App\Flight::firstOrNew(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );

#### `updateOrCreate`

Anda mungkin akan berhadapan pada situasi dimana anda ingin memperbarui model yang sudah ada atau membuatnya jika belum ada. Laravel memiliki method `updateOrCreate` untuk hal ini. Method ini sama seperti `firstOrCreate`, jadi tidak perlu memanggil `save`:

    // If there's a flight from Oakland to San Diego, set the price to $99.
    // If no matching model exists, create one.
    $flight = App\Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99]
    );

<a name="deleting-models"></a>
## Deleting Models

Untuk menghapus sebuah model, panggil method `delete`:

    $flight = App\Flight::find(1);

    $flight->delete();

#### Deleting An Existing Model By Key

Pada contoh diatas, kita menerima data dari database sebelum memanggil method `delete`. Tetapi, jika anda tahu primary key dari suatu model, anda tidak perlu menerima atau mencari datanya terlebih dahulu, tinggal hapus saja seperti kode di bawah:

    App\Flight::destroy(1);

    App\Flight::destroy([1, 2, 3]);

    App\Flight::destroy(1, 2, 3);

#### Deleting Models By Query

Of course, you may also run a delete statement on a set of models. In this example, we will delete all flights that are marked as inactive. Like mass updates, mass deletes will not fire any model events for the models that are deleted:

    $deletedRows = App\Flight::where('active', 0)->delete();

> {note} When executing a mass delete statement via Eloquent, the `deleting` and `deleted` model events will not be fired for the deleted models. This is because the models are never actually retrieved when executing the delete statement.

<a name="soft-deleting"></a>
### Soft Deleting

Dalam rangka untuk menghapus data dari database anda, Eloquent memiliki fitur "soft delete". Yang mana ketika suatu data dihapus menggunakan *soft delete*, maka sebenarnya data tersebut tidaklah benar-benar dihapus, melainkan data tersebut tidak akan lagi ditampilkan, dan atribut `deleted_at` akan diisi dengan nilai tanggal kapan data tersebut dihapus. Untuk memberikan fitur *soft delete* pada model anda, gunakan trait `lluminate\Database\Eloquent\SoftDeletes`, dan tambahkan kolom `deleted_at` serta properti `$dates`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;
    
    class Flight extends Model
    {
        use SoftDeletes;
    
        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = ['deleted_at'];
    }

Tentu saja, anda harus punya kolom `deleted_at` di tabel database anda. Laravel [schema builder](/docs/{{version}}/migrations) memiliki method - method berguna untuk membantu anda membuatkan kolom:

    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });

Sekarang, saat anda memanggil method `delete` pada suatu model, kolom `deleted_at` akan diisi oleh tanggal dan waktu disaat method tersebut dipanggil, seperti yang sudah dijelaskan diatas. Eloquent nantinya akan mengecualikan data-data yang sudah dihapus secara *soft* agar tidak diambil.

Untuk mengetahui apakah suatu data merupakan data yang *soft deleted*, gunakan method `trashed`:

    if ($flight->trashed()) {
        //
    }

<a name="querying-soft-deleted-models"></a>
### Querying Soft Deleted Models

#### Including Soft Deleted Models

Seperti yang disebutkan diatas, data yang *soft deleted* akan dikecualikan dari pengambilan data. Akan tetapi, jika anda ingin data yang *soft deleted* juga diambil dan ditampilkan, gunakan method `withTrashed`: 

    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

The `withTrashed` method may also be used on a [relationship](/docs/{{version}}/eloquent-relationships) query:

Method `withTrashed` juga bisa digunakan pada suatu kueri [relationship](/docs/{{version}}/eloquent-relationships) : 

    $flight->history()->withTrashed()->get();

#### Retrieving Only Soft Deleted Models

Method `onlyTrashed` akan menerima data yang sudah di *soft delete*:

    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

#### Restoring Soft Deleted Models

Kadang mungkin anda ingin membatalkan *soft delete* pada suatu data. Untuk melakukannya, gunakan method `restore`

    $flight->restore();
Anda juga mungkin menggunakan method `restore` pada suatu kueri seperti contoh di bawah.

    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

Seperti model `withTrashed`, method `restore` bisa digunakan untuk [relationships](/docs/{{version}}/eloquent-relationships)

    $flight->history()->restore();

#### Permanently Deleting Models

Kadang anda ingin menghapus suatu data secara permanen, walaupun data tersebut merupakan milik suatu model yang memiliki properti *soft delete*. Untuk menghapus data secara permanen, gunakan method `forceDelete`: 

    // Force deleting a single model instance...
    $flight->forceDelete();
    
    // Force deleting all related models...
    $flight->history()->forceDelete();

<a name="query-scopes"></a>
## Query Scopes

<a name="global-scopes"></a>
### Global Scopes

*Global scopes* memungkinkan anda untuk menambahkan konstrain kepada semua kueri pada suatu model. Fitur Laravel seperti [soft delete](#soft-deleting) juga menggunakan *global scopes* untuk menarik model "non-deleted" dari database. Anda bisa membuat *global scope* anda sendiri secara mudah.

#### Writing Global Scopes

Membuat *global scope* sangat simpel. Definisikan sebuah kelas yang mengimplementasi interface `Illuminate\Database\Eloquent\Scope`. Interface yang anda definisikan memiliki satu method `apply`. Method ini bisa digunakan dengan konstrain `where` ketika diperlukan.

    <?php

    namespace App\Scopes;

    use Illuminate\Database\Eloquent\Scope;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;
    
    class AgeScope implements Scope
    {
        /**
         * Apply the scope to a given Eloquent query builder.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $builder
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @return void
         */
        public function apply(Builder $builder, Model $model)
        {
            $builder->where('age', '>', 200);
        }
    }

> {tip} Jika global scope menambahkan kolom - kolom pada kalusa *select* dari suatu kueri, anda harus menggunakan method `addSelect` ketimbang `select`. Hal ini untuk mencegah pergantian suatu klausa select yang tidak diinginkan.

#### Applying Global Scopes

Untuk menetapkan *global scope* pada suatu model, anda harus menindih model tersebut dengan method `boot` dan gunakan method `addGlobalScope`: 

    <?php

    namespace App;

    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();
    
            static::addGlobalScope(new AgeScope);
        }
    }

Setelah menambahkan scope, tambahkan kueri `User::all()` yang akan memproduksi SQL sebagai berikut:

    select * from `users` where `age` > 200

#### Anonymous Global Scopes

Eloquent memungkinkan anda untuk mendefinisikan *global scope* menggunakan Closures, yang mana berguna untuk suatu *scope* yang tidak terpisah dari suatu class:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;
    
    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();
    
            static::addGlobalScope('age', function (Builder $builder) {
                $builder->where('age', '>', 200);
            });
        }
    }

#### Removing Global Scopes

Jika anda ingin menghapus suatu *global scope* pada suatu kueri, anda bisa menggunakan method `withoutGlobalScope`. Method ini menerima nama kelas pada *global scope* sebagai argumen:

    User::withoutGlobalScope(AgeScope::class)->get();
Atau, jika anda mendefinisikan sebuah *global scope* dengan Closure:

    User::withoutGlobalScope('age')->get();
Jika anda ingin menghapus beebrapa atau semua *global scope*, anda bisa gunakan method `withoutGlobalScopes`: 

    // Remove all of the global scopes...
    User::withoutGlobalScopes()->get();
    
    // Remove some of the global scopes...
    User::withoutGlobalScopes([
        FirstScope::class, SecondScope::class
    ])->get();

<a name="local-scopes"></a>
### Local Scopes

Local scopes allow you to define common sets of constraints that you may easily re-use throughout your application. For efinisikan konstrain - konstrain yang anda mungkin akan gunakan berkali-kali kembali pada aplikasi anda, seperti anda mungkin harus mengambil semua pengguna yang "populer". Untuk mendefinisikan sebuah scope, Eloquent menggunakan method `scope`. 

Scope selalu mengembalikan *instance* dari builder:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include popular users.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }
    
        /**
         * Scope a query to only include active users.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

#### Utilizing A Local Scope

Setelah *scope* didefinisikan, anda mungkin dapat memanggil method *scope* ketika melakukan kueri suatu model. Tetapi, anda tidak harus menyisipkan prefix `scope` ketika memanggil method. Anda bahkan dapat merantai atau menyambungkannya pada scope-scope lainnya, contoh: 

    $users = App\User::popular()->active()->orderBy('created_at')->get();

#### Dynamic Scopes

Kadang anda ingin mendefinisikan sebuah scope yang menerima parameter. Untuk memulai, tambahkan parameter pada scope anda. Parameter scope tersebut harus didefinisikan setelah parameter `$query`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include users of a given type.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @param mixed $type
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

Sekarang, anda bisa kirimkan parameter-parameter ketika memanggil scope:

    $users = App\User::ofType('admin')->get();

<a name="events"></a>
## Events

Model Eloquent turut memanggil atau menjalankan beberapa event-event, memungkinkan anda untuk turut memanggil beberapa dari model lifecycle seperti: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`. Event-event memudahkan anda untuk mengeksekusi kode setiap kali suatu kelas model disimpan atau diperbarui di dalam database.

The `retrieved` event will fire when an existing model is retrieved from the database. When a new model is saved for the first time, the `creating` and `created` events will fire. If a model already existed in the database and the `save` method is called, the `updating` / `updated` events will fire. However, in both cases, the `saving` / `saved` events will fire.

Event yang di `retrieved` akan terpanggil ketika sebuah model yang sudah ada di database dipanggil dari database. Ketika model baru disimpan untuk pertama kali, maka event `creating` dan `created` yang akan terpanggil. Jika sebuah model sudah ada di database dan method `save` dipanggil, maka event `updating` / `updated` yang akan terpanggil. Jika ada skenario keduanya maka event `saving` / `saved` yang akan terpanggil 

Untuk memulai, definisikan `$dispatchesEvents` pada model Eloquent anda yang menuju pada beberapa lifecycle tadi: 

    <?php

    namespace App;

    use App\Events\UserSaved;
    use App\Events\UserDeleted;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    
    class User extends Authenticatable
    {
        use Notifiable;
    
        /**
         * The event map for the model.
         *
         * @var array
         */
        protected $dispatchesEvents = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }

<a name="observers"></a>
### Observers

Jika anda mendengarkan banyak event pada model tertentu, anda bisa menggunakan *observers* untuk mengrupkan *listener - listener* anda ke dalam suatu class. Kelas *Observer* memiliki nama method yang berefleksi pada event Eloquent yang anda dengarkan. Setiap method - method tersebut menerima model sebagai argumen satu-satunya. Laravel tidak memasukkan direktori bawaan untuk suatu Observer, jadi anda bebas untuk membuatnya dan meletakkannya dimana saja:

    <?php

    namespace App\Observers;

    use App\User;

    class UserObserver
    {
        /**
         * Listen to the User created event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function created(User $user)
        {
            //
        }
    
        /**
         * Listen to the User deleting event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function deleting(User $user)
        {
            //
        }
    }

Untuk register suatu observer, gunakan method `observe` pada model yang ingin anda pantau. anda mungkin perlu register observer pada method `boot` di salah satu *service provider* yang anda miliki. Contoh, kita akan meregister observer pada `AppServiceProvider`:

    <?php

    namespace App\Providers;

    use App\User;
    use App\Observers\UserObserver;
    use Illuminate\Support\ServiceProvider;
    
    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            User::observe(UserObserver::class);
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
