
# PELAPORAN_SEKOLAH, InputAspirasi dan Aspirasi
1. membuat folder landing ( index.blade.php , form.blade,php, laporan/history.blade.php
2. buat migration InputAspirasi
Schema::create('inputaspirasi', function (Blueprint $table) {
            $table->bigIncrements('id_pelaporan');
            $table->bigInteger('idsiswa')->unsigned();
            $table->foreign('idsiswa')->references('idsiswa')->on('siswa');
            $table->bigInteger('idkat')->unsigned();
            $table->foreign('idkat')->references('idkat')->on('kategori');
            $table->string('tenggatwaktu');
            $table->string('ket');
            $table->string('photo');
            $table->timestamps();
        });
3. buat migration Aspirasi
Schema::create('aspirasi', function (Blueprint $table) {
            $table->increments('id_aspirasi');
            $table->enum('status', array('menunggu','diproses','disetujui'));
            $table->biginteger('id_pelaporan')->unsigned();
            $table->foreign('id_pelaporan')->references('id_pelaporan')->on('inputaspirasi');
            $table->string('feedback');
            $table->timestamps();
        });

4. php artisan migrate
5. membuat model InputAspirasi
protected $table = 'inputaspirasi';
    protected $primaryKey = 'id_pelaporan';
    protected $fillable = ['id_pelaporan', 'idsiswa', 'idkat', 'tenggatwaktu', 'ket', 'photo'];


    public function siswa()
    {
        return $this->belongsTo(Siswa::class, 'idsiswa', 'idsiswa');
    }
    public function kategori()
    {
        return $this->belongsTo(Kategori::class, 'idkat', 'idkat');
    }



    public function aspirasi()
    {
        return $this->belongsTo(Aspirasi::class, 'id_pelaporan', 'id_pelaporan');
    }

    public function getCreatedAtAtribute()
    {
        return Carbon::parse($this->attributes['created_at'])->translatedFormat('l, d F Y H:i');
    }

6. buat model Aspirasi
protected $table = 'aspirasi';
    protected $primaryKey = 'id_aspirasi';
    protected $fillable = ['id_aspirasi', 'status', 'id_pelaporan', 'feedback'];

    public function input_apirasi()
    {
        return $this->hasOne(InputAspirasi::class, 'id_pelaporan', 'id_pelaporan');
    }

7. buat controller InputAspirasiController
public function index()
    {
        $siswa = Siswa::all();
        $kategori = Kategori::all();
        $ia = InputAspirasi::all();

        return view('layout.master-landing', compact('siswa', 'kategori', 'ia'));
    }

    public function store(Request $request)
    {


        $id_pelaporan = random_int(10000, 100000);
        $validasi_idsiswa = Siswa::select('idsiswa')->where('idsiswa', $request->get('idsiswa'))->value('idsiswa');

        if ($validasi_idsiswa) {
            $filename = time() . $request->photo->getClientOriginalName();
            $request->photo->move(public_path('public/img'), $filename);
            inputaspirasi::create([
                'id_pelaporan' => $id_pelaporan,
                'idsiswa' => $request->idsiswa,
                'idkat' => $request->idkat,
                'tenggatwaktu' => $request->tenggatwaktu,
                'ket' => $request->ket,
                'photo' => $filename
            ]);

            Aspirasi::create([
                'status' => 'menunggu',
                'id_pelaporan' => $id_pelaporan,
                'feedback' => '-'
            ]);

            return redirect('halaman')->with('message-success', 'Di Kirim! Data Anda Sedang Di Proses, Tunggu!');
        } else {
            return redirect('halaman')->with('message-error', 'Error! Data Anda Salah/Tidak Terdaftar, Ulangi!');
        }
    }

8. buat controller AspirasiController
public function index()
    {
        $aspirasi = InputAspirasi::all();
        return view('aspirasi.index-aspi', compact('aspirasi'));
    }

    public function feedback(Request $request, $id)
    {
        $aspirasi = Aspirasi::find($id);

        $aspirasi->update([
            'feedback'=>$request->feedback
        ]);

        return redirect()->back()->with('pesan-feedback', 'terimakasih sudah merespon');

    }

    public function status_proses( Request $request, $id)
    {
        $aspirasi = Aspirasi::find($id);
        $aspirasi->update([
            'status'=>'diproses'
        ]);

        return redirect()->back()->with('pesan-proses', 'status berhasil diproses');
    }

    public function status_selesai(Request $request, $id)
    {
        $aspirasi = Aspirasi::find($id);
        $aspirasi->update([
            'status'=>'disetujui'
        ]);

        return redirect()->back()->with('pesan-selesai', 'status berhasil disetujui');
    }

9. web.php

        
        Route::resource('aspirasi', AspirasiController::class);

        route::put('feedback/{id}',[AspirasiController::class,'feedback'])->name('aspirasi.feedback');
        route::put('statusproses/{id}',[AspirasiController::class,'status_proses'])->name('aspirasi.proses');
        route::put('statusselesai/{id}',[AspirasiController::class,'status_selesai'])->name('aspirasi.selesai');

10. buka folder landing form.blade.php
<div class="mb-12">

    @if (Session::has('message-success'))
        <div class="alert alert-success alert-dismissible fade show" role="alert">
            {{ Session::get('message-success') }}
            <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                <span aria-hidden="true">&times;</span>
            </button>
            <i class="fa fa-check"></i>
        </div>
    @endif

    @if (Session::has('message-error'))
        <div class="alert alert-danger alert-dismissible fade show" role="alert">
            {{ Session::get('message-error') }}
            <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                <span aria-hidden="true">&times;</span>
            </button>
            <i class="fa fa-exclamation-circle"></i>
        </div>
</div>
@endif

<form action="{{ route('halaman.store') }}" method="POST" enctype="multipart/form-data">
    @csrf
    <div class="mb-3">
        <label for="idsiswa" class="form-label">ID Siswa</label>
        <input type="text" class="form-control" name="idsiswa" id="idsiswa" aria-describedby="">
    </div>

    <div class="mb-3">
        <label for="nama_siswa" class="form-label">Nama Siswa</label>
        <input type="text" class="form-control" name="nama_siswa" id="nama_siswa" aria-describedby="">
    </div>

    <div class="mb-3">
        <label for="kategori" class="form-label">Kategori</label>
        <select name="idkat" class="form-control" id="validationCustom04" required>
            <option selected disabled value="">Pilih Kategori</option>
            @foreach ($kategori as $k)
                <option value="{{ $k->idkat }}">{{ $k->namkat }}</option>
            @endforeach
        </select>
    </div>

    <div class="mb-3">
        <label for="tenggatwaktu" class="form-label">Tenggat Waktu</label>
        <input type="text" class="form-control" name="tenggatwaktu" id="tenggatwaktu" aria-describedby="">
    </div>

    <div class="mb-3">
        <label for="ket" class="form-label">Keterangan</label>
        <input type="text" class="form-control" name="ket" id="ket" aria-describedby="">
    </div>

    <div class="mb-3">
        <label for="photo" class="form-label">PHOTO</label>
        <input type="file" class="form-control" id="photo" name="photo" aria-describedby="photo">
    </div>


    <button type="submit" class="btn btn-primary">Submit</button>
</form>
</div>

11. buka file laporan.blade.php
<table class="table table-striped">
    <thead>
        <tr>
            <th scope="col">NO.</th>
            <th scope="col">Tanggal Pelaporan</th>
            <th scope="col">Nama Siswa</th>
            <th scope="col">Kategori</th>
            <th scope="col">Tenggat Waktu</th>
            <th scope="col">Keterangan</th>
            <th scope="col">Photo</th>
            <th scope="col">Feedback</th>
            <th scope="col">Status</th>
        </tr>
    </thead>
    <tbody>
        @foreach ($ia as $k => $v)
            <tr>
                <td>{{ $k += 1 }}</td>
                <td>{{ $v->created_at }}</td>
                <td>{{ $v->siswa->nama_siswa }}</td>
                <td>{{ $v->kategori->namkat }}</td>
                <td>{{ $v->tenggatwaktu }}</td>
                <td>{{ $v->ket }}</td>
                <td>
                    <img height="60" src="{{ asset('public/img/' . $v->photo) }}">
                </td>
                <td>
                    @if ($v->aspirasi->status == 'disetujui')
                        @if ($v->aspirasi->feedback != '-')
                            {{ $v->aspirasi->feedback }}
                        @else
                            <form action="{{ route('aspirasi.feedback', $v->aspirasi->id_aspirasi) }}" method="POST">
                                @csrf
                                @method('PUT')
                                <button type="submit" name="feedback" class="btn btn-success btn-sm"
                                    value="puas">Puas</button>
                                <button type="submit" name="feedback" class="btn btn-warning btn-sm"
                                    value="cukup puas">Cukup
                                    Puas</button>
                                <button type="submit" name="feedback" class="btn btn-danger btn-sm"
                                    value="tidak puas">Tidak
                                    Puas</button>
                            </form>
                        @endif
                    @else
                        -
                    @endif
                </td>
                <td>{{ $v->aspirasi->status }}</td>
            </tr>
        @endforeach
    </tbody>
</table>

12. buka folder aspirasi index-aspirasi.blade.php
@extends('layout.master')

@section('title', 'History')

@section('content')
    <div class="col-lg-12 col-12">
        <div class="card">
            <div class="card-header">
                <h4 class="card-title">History</h4>
            </div>
            <div class="card-body">
                <table class="table">
                    <thead>
                        <tr>
                            <th scope="col">No.</th>
                            <th scope="col">Tanggal Pelaporan</th>
                            <th scope="col">Nama Siswa</th>
                            <th scope="col">Kategori</th>
                            <th scope="col">Tenggat Waktu</th>
                            <th scope="col">Keterangan</th>
                            <th scope="col">Photo</th>
                            <th scope="col">Feedback</th>
                            <th scope="col">Status</th>
                            <th scope="col">Action</th>
                        </tr>
                    </thead>
                    <tbody>
                        @foreach ($aspirasi as $k => $v)
                            <tr>
                                <td>{{ $k += 1 }}</td>
                                <td>{{ $v->created_at }}</td>
                                <td>{{ $v->siswa->nama_siswa }}</td>
                                <td>{{ $v->kategori->namkat }}</td>
                                <td>{{ $v->tenggatwaktu }}</td>
                                <td>{{ $v->ket }}</td>
                                <td>
                                    <img height="60" src="{{ asset('public/img/' . $v->photo) }}">
                                </td>
                                <td>{{ $v->aspirasi->feedback }}</td>
                                <td>{{ $v->aspirasi->status }}</td>

                                <td>
                                    @if ($v->aspirasi->status == 'menunggu')
                                        <form action="{{ route('aspirasi.proses', $v->aspirasi->id_aspirasi) }}"
                                            method="POST">
                                            @csrf
                                            @method('PUT')
                                            <button class="btn btn-primary" type="submit">Proses</button>
                                        </form>
                                    @elseif($v->aspirasi->status == 'diproses')
                                        <form action="{{ route('aspirasi.selesai', $v->aspirasi->id_aspirasi) }}"
                                            method="POST">
                                            @csrf
                                            @method('PUT')
                                            <button class="btn btn-success" type="submit">Selesai</button>
                                        </form>
                                    @elseif($v->aspirasi->status == 'disetujui')
                                        <form action="{{ route('aspirasi.feedback', $v->aspirasi->id_aspirasi) }}"
                                            method="POST">
                                            @csrf
                                            @method('PUT')
                                            <button class="btn btn-secondary" type="submit" disabled>Done</button>
                                        </form>
                                    @endif
                                </td>

                            </tr>
                        @endforeach
                    </tbody>
                </table>
            </div>
        </div>
    </div>
@endsection

# MEMBUAT LOGIN PETUGAS/USER TANPA REGISTER
    1. buat migration dan model petugas
    2. MIGRATION PETUGAS
        Schema::create('petugas', function (Blueprint $table) {
            $table->id('id_petugas');
            $table->string('nama_petugas', 35);
            $table->string('username', 25)->unique();
            $table->enum('level', ['admin', 'petugas']);
            $table->string('password');
            $table->string('notelp', 13);
            $table->timestamps();
        });
        3. migrate
            php artisan migrate
        4. MODEL PETUGAS
            use Illuminate\Foundation\Auth\User as Authenticatable;

class Petugas extends Authenticatable
{
    use HasFactory;
            protected $table = 'petugas';
            protected $primaryKey = 'id_petugas';
            protected $fillable = ['nama_petugas', 'username', 'level', 'password', 'notelp'];
}

        5. ke folder config auth.php
            'admin' => [
            'driver' => 'session',
            'provider' => 'admins'
            ],

---------------------
'admins' => [
            'driver' => 'eloquent',
            'model' => App\Models\Petugas::class,
        ],

        6. ke folder seeders
            public function run()
    {
        \App\Models\Petugas::create([
            'nama_petugas' => 'Administrator',
            'username' => 'admin',
            'level' => 'admin',
            'password' => Hash::make('password'),
            'notelp' => '081918244230'
        ]);

        7. lalu, php artisan migrate:fresh --seed
        8. jika folder/file yang sudah di migrate:fresh maka buat AdminController
        9. setelah dibuat,
            public function FormLogin()
    {
        return view('admin.login');
    }

    public function login ( Request $request)
    {
        $username = Petugas::where('username', $request->username)->first();

        if (!$username) {
            return redirect ()->back()->with(['pesan' => 'Username tidak terdaftar!' ]);
        }

        $password = Hash::check($request->password, $username->password);

        if (!$password){
            return redirect ()->back()->with(['pesan' => 'Username tidak sesuai!']);
        }

        $auth = Auth::guard('admin')->attempt(['username' => $request->username, 'password' => $request->password]);

        if ($auth){
            return redirect ()->route('dashboard');
        } else {
            return redirect ()->back()->with(['pesan' => 'Akun tidak terdaftar!']);
        }
    }

    public function logout()
    {
        Auth::guard('admin')->logout();

        return redirect ()->route('admin.FormLogin');
    }

        10. web.php
              
Route::resource('petugas', PetugasController::class);
Route::resource('siswa', SiswaController::class);
Route::resource('ekskul', EkskulController::class);

route::prefix('admin')->group(function (){
route::get('/', [AdminController::class, 'formLogin'])->name('admin.formLogin');
route::post('/login', [AdminController::class, 'login'])->name('admin.login');
route::get('/logout', [AdminController::class, 'logout'])->name('admin.logout');

route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
});

11. buat folder admin login.blade.php
      <!doctype html>
<html lang="en">

<head>
    <title>Login | Admin</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <link href="https://fonts.googleapis.com/css?family=Lato:300,400,700&display=swap" rel="stylesheet">

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css">

    <link rel="stylesheet" href="{{ asset('loginuser/css/style.css') }}">

</head>

<body>
    <section class="ftco-section">
        <div class="container">
            <div class="row justify-content-center">
                <div class="col-md-6 text-center mb-5">
                    <h2 class="heading-section">Login</h2>
                </div>
            </div>
            <div class="row justify-content-center">
                <div class="col-md-6 col-lg-5">
                    <div class="login-wrap p-4 p-md-5">
                        <div class="icon d-flex align-items-center justify-content-center">
                            <span class="fa fa-user-o"></span>
                        </div>
                        <h3 class="text-center mb-4"></h3>
                        <form action="{{ route('admin.login') }}" method="POST" class="login-form">
                            @csrf

                            <div class="form-group">
                                <input type="text" class="form-control" name="username" id="username"
                                    placeholder="Username" value="">
                            </div>
                            <div class="form-group d-flex">
                                <input type="password" class="form-control" name="password" id="password"
                                    placeholder="Password">
                            </div>
                            <div class="form-group d-md-flex">
                                <div class="w-50">
                                    <label class="checkbox-wrap checkbox-primary">Remember Me
                                        <input type="checkbox" checked>
                                        <span class="checkmark"></span>
                                    </label>
                                </div>
                                <div class="w-50 text-md-right">
                                    <a href="#">Forgot Password</a>
                                </div>
                            </div>
                            <div class="form-group">
                                <button type="submit" class="btn btn-primary rounded submit p-3 px-5">Get
                                    Started</button>
                            </div>
                        </form>
                    </div>
                </div>


            </div>
        </div>
    </section>

    <script src="{{ asset('loginuser/js/jquery.min.js') }}"></script>
    <script src="{{ asset('loginuser/js/popper.js') }}"></script>
    <script src="{{ asset('loginuser/js/bootstrap.min.js') }}"></script>
    <script src="{{ asset('loginuser/js/main.js') }}"></script>

</body>

</html>

