# Update diskon transaksi UKK 

## Langkah - langkah
### Tambah migrations untuk paket
```php
php artisan make:migration add_diskon_and_harga_akhir_to_paket --table=pakets
```

### edit pada bagian function up
```php
public function up()
    {
        Schema::table('pakets', function (Blueprint $table) {
            $table->integer('diskon')->nullable()->after('jenis');
            $table->integer('harga_akhir')->nullable()->after('harga');
        });
    }
```
### Tambahkan *diskon* dan *harga_akhir* pada fillable di model **Paket.php**
```php
class Paket extends Model
{

    use HasFactory;

    public $timestamps = false;

    protected $fillable = [
        'nama_paket',
        'harga',
        'jenis',
        'diskon',
        'harga_akhir',
        'outlet_id',
    ];
}
```
### Edit *PaketController*
```php
    public function index(Request $request)
    {
        // ...............
            ->select(
                'pakets.id as id',
                'nama_paket',
                'harga',
                'jenis',
                'diskon', // tambahkan ini
                'harga_akhir', // tambahkan ini
                'outlets.nama as outlet'
            )
            ->paginate();
        // ...............
        $pakets->map(function ($row) use ($jenis) {
            $row->jenis = $jenis[$row->jenis];
            $row->harga = number_format($row->harga, 0, ',', '.');
            $row->diskon = number_format($row->diskon, 0, ',', '.'); // tambahkan ini
                $row->harga_akhir = number_format($row->harga_akhir, 0, ',', '.'); // tambahkan ini
            return $row;
        });

        // ...............
    }

    public function store(Request $request)
    {
        $request->validate([
            'nama_paket' => 'required|max:100',
            'harga' => 'required|numeric|min:0',
            'jenis' => 'required|in:kiloan,kaos,bed_cover,selimut,lain',
            'diskon' => 'nullable|numeric|min:0|', // tambahkan ini
            'harga_akhir' => 'required|numeric|min:0|', // tambahkan ini
            'outlet_id' => 'required|exists:outlets,id',
        ], [], [
            'outlet_id' => 'Outlet',
        ]);

        // ...............
    }

    public function update(Request $request, Paket $paket)
    {
        $request->validate([
            'nama_paket' => 'required|max:100',
            'harga' => 'required|numeric|min:0',
            'jenis' => 'required|in:kiloan,kaos,bed_cover,selimut,lain',
            'diskon' => 'nullable|numeric|min:0|', // tambahkan ini
            'harga_akhir' => 'required|numeric|min:0|', // tambahkan ini
            'outlet_id' => 'required|exists:outlets,id',
        ], [], [
            'outlet_id' => 'Outlet',
        ]);

        // ...............
    }
```

### Merubah bagian view yaitu index,create,edit

### index.blade.php
```html
<!-- index.blade.php -->
<!-- ................ -->
<table class="table table-hover table-striped m-0">
    <thead>
        <tr>
            <th>No</th>
            <th>Nama Paket</th>
            <th>Harga</th>
            <th>Jenis</th>
            <th>Diskon</th> <!-- tambahkan ini -->
            <th>Harga Akhir</th> <!-- tambahkan ini -->
            <th>Outlet</th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        <?php
            $no = $pakets->firstItem()
        ?>
        @foreach($pakets as $paket)
            <tr>
                <td>{{ $no++ }}</td>
                <td>{{ $paket->nama_paket }}</td>
                <td>{{ $paket->harga }}</td>
                <td>{{ $paket->jenis }}</td>
                <td>{{ $paket->diskon }}</td> <!-- tambahkan ini -->
                <td>{{ $paket->harga_akhir }}</td> <!-- tambahkan ini -->
                <td>{{ $paket->outlet }}</td>
                <td class="text-right">
                    <x-edit :href="route('paket.edit', ['paket' => $paket->id])"/>
                    <x-delete :data-url="route('paket.destroy', ['paket' => $paket->id])"/>
                </td>
            </tr>
        @endforeach
    </tbody>
</table>
<!-- ................ -->
```
### create.blade.php
```html
<!-- create.blade.php -->
@extends('layouts.main', ['title' => 'Paket'])
@section('content')
    <x-content :title="['name' => 'Paket','icon' => 'fas fa-cubes']">
        <div class="row">
            <div class="col-md-6">
                <form action="{{ route('paket.store') }}" method="post" class="card card-primary">
                    <div class="card-header">
                        Buat Paket
                    </div>
                    <div class="card-body">
                        @csrf
                        <x-input label="Nama Paket" name="nama_paket"/>
                        <x-input label="Harga" name="harga" id="harga" min="0" />
                        <x-input label="Diskon" name="diskon" type="number"/>
                        <x-select label="Jenis" name="jenis" :data-option="[
                            ['option' => 'Kiloan', 'value' => 'kiloan'],
                            ['option' => 'T-Shirt/Kaos', 'value' => 'kaos'],
                            ['option' => 'Bed Cover', 'value' => 'bed_cover'],
                            ['option' => 'Selimut', 'value' => 'selimut'],
                            ['option' => 'Lainnya', 'value' => 'lain'],
                        ]" />
                        <x-select label="Outlet" name="outlet_id" :data-option="$outlets" />
                        <x-input label="Harga Akhir" name="harga_akhir" id="harga_akhir" readonly />
                    </div>
                    <div class="card-footer">
                        <x-btn-submit />
                    </div>
                </form>
            </div>
        </div>
    </x-content>
@endsection


@push('js')
    <script>
        $(document).ready(function() {
            // Function to calculate final price
            function calculateFinalPrice() {
                let harga = parseInt($('#harga').val());
                let diskon = parseInt($('input[name="diskon"]').val());
                if (isNaN(diskon)) {
                    diskon = 0;
                }
                let harga_akhir = harga - diskon;
                if (harga_akhir < 0) {
                    $('#harga_akhir').val('');
                    alert('Diskon tidak boleh melebihi harga.');
                    $('button[type="submit"]').attr('disabled', true);
                    return;
                }
                $('#harga_akhir').val(harga_akhir);
                $('button[type="submit"]').attr('disabled', false);
            }

            // Calculate final price on input change
            $('#harga').on('input', function() {
                calculateFinalPrice();
            });

            $('input[name="diskon"]').on('input', function() {
                calculateFinalPrice();
            });
        });
    </script>
@endpush
```

### edit.blade.php
```html
@extends('layouts.main', ['title' => 'Paket'])
@section('content')
    <x-content :title="[
        'name' => 'Paket',
        'icon' => 'fas fa-store-alt'
    ]">
        <div class="row">
            <div class="col-lg-4 col-md-6">
                <form action="{{ route('paket.update', ['paket' => $paket->id]) }}" method="post" class="card card-primary">
                    <div class="card-header">
                        Edit Paket
                    </div>
                    <div class="card-body">
                        @csrf
                        @method('PUT')
                        <x-input label="Nama Paket" name="nama_paket" :value="$paket->nama_paket"/>
                        <x-input label="Harga" name="harga" id="harga" :value="$paket->harga"/>
                        <x-input label="Diskon" name="diskon" :value="$paket->diskon" type="number"/>
                        <x-select label="Jenis" name="jenis" :data-option="[
                            ['option' => 'Kiloan', 'value' => 'kiloan'],
                            ['option' => 'T-Shirt/Kaos', 'value' => 'kaos'],
                            ['option' => 'Bed Cover', 'value' => 'bed_cover'],
                            ['option' => 'Selimut', 'value' => 'selimut'],
                            ['option' => 'Lainnya', 'value' => 'lain'],
                        ]" :value="$paket->jenis"/>
                        <x-select label="Outlet" name="outlet_id" :data-option="$outlets" :value="$paket->outlet_id" />
                        <x-input label="Harga Akhir" name="harga_akhir" id="harga_akhir" :value="$paket->harga_akhir" readonly />
                    </div>
                    <div class="card-footer">
                        <x-btn-update />
                    </div>
                </form>
            </div>
        </div>
    </x-content>
@endsection


@push('js')
    <script>
        $(document).ready(function() {
            // Function to calculate final price
            function calculateFinalPrice() {
                let harga = parseInt($('#harga').val());
                let diskon = parseInt($('input[name="diskon"]').val());
                if (isNaN(diskon)) {
                    diskon = 0;
                }
                let harga_akhir = harga - diskon;
                if (harga_akhir < 0) {
                    $('#harga_akhir').val('');
                    alert('Diskon tidak boleh melebihi harga.');
                    $('button').attr('disabled', true);
                    return;
                }
                $('#harga_akhir').val(harga_akhir);
                $('button').attr('disabled', false);
            }

            // Calculate final price on input change
            $('#harga').on('input', function() {
                calculateFinalPrice();
            });

            $('input[name="diskon"]').on('input', function() {
                calculateFinalPrice();
            });
        });
    </script>
@endpush
```

### edit TransaksiController
```php
    public function add(Request $request, Member $member)
    {
        $request->validate([
            'paket' => 'required|exists:pakets,id',
            'quantity' => 'required|numeric|min:1',
            'keterangan' => 'nullable|max:200',
        ]);

        $paket = Paket::find($request->paket);

        Cart::session($member->id)->add(array(
            'id' => $paket->id,
            'name' => $paket->nama_paket,
            'price' => $paket->harga_akhir,
            'quantity' => $request->quantity,
            'attributes' => [
                'harga_awal' => $paket->harga,
                'keterangan' => $request->keterangan,
                'diskon' => $paket->diskon,
            ]
        ));

        return back();
    }


    public function store(Request $request, Member $member)
    {
        // .............

        foreach ($items as $item) {
            TransaksiDetail::create([
                'transaksi_id' => $transaksi->id,
                'paket_id' => $item->id,
                'harga' => $item->attributes->harga_awal,
                'diskon_paket' => $item->attributes->diskon,
                'qty' => $item->quantity,
                'sub_total' => $item->price * $item->quantity,
                'keterangan' => $item->attributes->keterangan,
            ]);
        }

        // ...............
    }

    public function detail(Transaksi $transaksi)
    {
        $user = User::find($transaksi->user_id);
        $member = Member::find($transaksi->member_id);
        $outlet = Outlet::find($transaksi->outlet_id);
        $items = TransaksiDetail::join('pakets', 'pakets.id', 'transaksi_details.paket_id')
            ->where('transaksi_id', $transaksi->id)
            ->select(
                'pakets.id as id',
                'nama_paket',
                'qty',
                'transaksi_details.harga as harga',
                'sub_total',
                'diskon_paket', // tambahkan ini
                'keterangan',
            )
            ->get();

        return view('transaksi.detail', [
            'items' => $items,
            'member' => $member,
            'user' => $user,
            'outlet' => $outlet,
            'transaksi' => $transaksi,
        ]);
    }
```

### edit view transaksi

#### create.blade.php
```html
@extends('layouts.main', ['title' => 'Transaksi'])
@section('content')
    <x-content :title="['name' => 'Transaksi','icon' => 'fas fa-cash-register']">
        <div class="card card-primary card-outline">
            <div class="card-header">
                <div class="row">
                    <div class="col">
                        <div class="form-group">Nama Member : {{ $member->nama }}</div>
                        <div class="form-group">No. Telepon : {{ $member->tlp}}</div>
                        <div class="form-group">Alamat : {{ $member->alamat }}</div>
                    </div>
                    <div class="col-2"></div>
                    <div class="col">
                        <form action="{{ route('transaksi.add', ['member' => $member->id]) }}" method="post">
                            @csrf
                            <div class="form-group row">
                                <label class="col">Outlet</label>
                                <div class="col">
                                    <input type="text" value="{{ $outlet->nama }}" class="form-control" disabled>
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col">Pilih Paket </label>
                                <div class="col">
                                    <x-select-transaksi
                                        name="paket"
                                        :data-option="$pakets" required
                                    />
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col">Quantity </label>
                                <div class="col">
                                    <x-input-transaksi
                                        name="quantity" type="number" required
                                    />
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col">Keterangan </label>
                                <div class="col">
                                    <x-textarea-transaksi
                                        name="keterangan" required
                                    />
                                </div>
                            </div>
                            <div class="form-group row">
                                <div class="offset-6 col">
                                    <button class="btn btn-primary btn-block" type="submit">
                                        Tambah
                                    </button>
                                </div>
                            </div>
                        </form>
                    </div>
                </div>
            </div>
            <div class="card-body p-0">
                <table class="table table-striped table-hover m-0">
                    <thead>
                        <tr>
                            <th>No</th>
                            <th>Nama Paket</th>
                            <th>Qty</th>
                            <th>Harga</th>
                            <th>Diskon</th>
                            <th>Sub Total</th>
                            <th>Keterangan</th>
                            <th></th>
                        </tr>
                    </thead>
                    <tbody>
                        <?php $no = 1; ?>
                        @forelse ($items as $item)
                            <tr>
                                <td>{{ $no++ }}</td>
                                <td>{{ $item->name }}</td>
                                <td>
                                    {{ $item->quantity }} x {{ number_format($item->attributes->harga_awal,0,',','.') }}
                                </td>
                                <td>
                                    {{ number_format($item->quantity * $item->attributes->harga_awal,0,',','.') }}
                                </td>
                                <td>
                                    {{ number_format($item->quantity * $item->attributes->diskon,0,',','.') }}
                                </td>
                                <td>
                                    {{ number_format($item->quantity * $item->price,0,',','.') }}
                                </td>
                                <td>{{ $item->attributes->keterangan }}</td>
                                <td>
                                    <a href="{{ route('transaksi.qUpdate', ['member' => $member->id ,'paket' => $item->id, 'type' => 'plus']) }}"
                                        class="btn p-0 text-primary">
                                        <i class="fas fa-plus-square"></i>
                                    </a>
                                    <a href="{{ route('transaksi.qUpdate', ['member' => $member->id ,'paket' => $item->id, 'type' => 'min']) }}"
                                        class="btn p-0 text-warning">
                                        <i class="fas fa-minus-square"></i>
                                    </a>
                                    <a href="{{ route('transaksi.delete', ['member' => $member->id, 'paket' => $item->id]) }}"
                                        class="btn p-0 text-danger">
                                        <i class="fas fa-trash"></i>
                                    </a>
                                </td>
                            </tr>
                        @empty
                            <tr>
                                <td colspan="8" class="text-center">
                                    Tidak ada paket dipilih.
                                </td>
                            </tr>
                        @endforelse
                    </tbody>
                </table>
            </div>
            <form action="{{ route('transaksi.store', ['member' => $member->id]) }}" method="post" class="card-body border-top">
                @csrf
                <div class="row">
                    <div class="col">
                        <div class="form-group row">
                            <label class="col">Tanggal</label>
                            <div class="col">
                                <x-input-transaksi name="tanggal" :value="date('d/m/Y H:i:s')" disabled />
                            </div>
                        </div>
                        <div class="form-group row">
                            <label class="col">Batas Waktu</label>
                            <div class="col">
                                <x-input-transaksi name="batas_waktu" type="datetime-local" required />
                            </div>
                        </div>
                    </div>
                    <div class="col-2"></div>
                    <div class="col">
                        <div class="form-group row">
                            <label class="col">Total</label>
                            <div class="col">
                                <x-input-transaksi name="total" id="total" :value="$total" disabled />
                            </div>
                        </div>
                        <div class="form-group row">
                            <label class="col">Diskon Tambahan (Optional)</label>
                            <div class="col">
                                <x-input-transaksi name="diskon" id="diskon"  />
                            </div>
                        </div>
                        <div class="form-group row">
                            <label class="col">Biaya Tambahan (Optional)</label>
                            <div class="col">
                                <x-input-transaksi name="biaya_tambahan" id="biaya_tambahan"/>
                            </div>
                        </div>
                        <div class="form-group row">
                            <label class="col">Pajak (10%)</label>
                            <div class="col">
                                <x-input-transaksi name="pajak" id="pajak" :value="$total * 10 / 100" disabled/>
                            </div>
                        </div>
                        <div class="form-group row">
                            <label class="col">Total Bayar</label>
                            <div class="col">
                                <x-input-transaksi name="total_bayar" id="total_bayar" :value="$total + ($total * 10 / 100)" disabled/>
                            </div>
                        </div>
                        <div class="form-group row">
                            <label class="col">Uang Tunai / Cash (Optional)</label>
                            <div class="col">
                                <x-input-transaksi name="uang_tunai"/>
                            </div>
                        </div>
                        <div class="form-group row">
                            <div class="col">
                                <a href="{{ route('transaksi.index') }}" class="btn btn-default">Kembali</a>
                                <a href="{{ route('transaksi.clear', ['member' => $member->id]) }}" class="btn btn-danger">Clear</a>
                            </div>
                            <div class="col">
                                <button type="submit" class="btn btn-primary btn-block">
                                    <i class="fas fa-database mr-2"></i> Simpan / Proses
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </x-content>
@endsection

@push('js')
    <script>
        $('#diskon, #biaya_tambahan').keyup(function (e) {
            let t = parseInt($('#total').val());
            let d = parseInt($('#diskon').val());
            let bt = parseInt($('#biaya_tambahan').val());
            d = isNaN(d) ? 0 : d;
            bt = isNaN(bt) ? 0 : bt;
            let total = t - d + bt ;
            let pajak = Math.round( total * 10 / 100 );
            $("#pajak").val(pajak);
            $("#total_bayar").val(total + pajak);
        });
    </script>
@endpush
```
#### detai.blade.php

```html

@extends('layouts.main', ['title' => 'Transaksi'])
@section('content')
    <x-content :title="['name' => 'Transaksi','icon' => 'fas fa-cash-register']">

        @if (session('message') == 'success update')
            <x-alert-success type="update" />
        @endif
        @if (session('message') == 'fail store')
            <x-alert-danger />
        @endif
        <div class="card card-outline card-primary">
            <div class="card-header">
                <div class="row">
                    <div class="col">
                        <div class="form-group">Nama Member : {{ $member->nama }}</div>
                        <div class="form-group">No. Telepon : {{ $member->tlp }}</div>
                        <div class="form-group">Alamat : {{ $member->alamat }}</div>
                    </div>
                    <div class="col-2"></div>
                    <div class="col">
                        <div class="form-group">Outlet : {{ $outlet->nama }}</div>
                        <div class="form-group">Kasir : {{ $user->nama }}</div>
                        <div class="form-group">
                            <a href="{{ route('transaksi.invoice', ['transaksi' => $transaksi->id]) }}" target="_blank" class="btn btn-primary">
                                Cetak Invoice
                            </a>
                        </div>
                    </div>
                </div>
            </div>
            <div class="card-body p-0">
                <table class="table table-hover table-striped m-0">
                    <thead>
                        <tr>
                            <th>No</th>
                            <th>Nama Paket</th>
                            <th>Qty</th>
                            <th>Harga</th>
                            <th>Diskon</th>
                            <th>Sub Total</th>
                            <th>Keterangan</th>
                        </tr>
                    </thead>
                    <tbody>
                        <?php
                          $no = 1 ;
                        ?>
                        @forelse ($items as $item)
                            <tr>
                                <td>{{ $no++ }}</td>
                                <td>{{ $item->nama_paket }}</td>
                                <td>{{ $item->qty }} x {{ number_format($item->harga,0,',','.') }}</td>
                                <td>{{ number_format($item->qty * $item->harga,0,',','.') }}</td>
                                <td>{{ number_format($item->qty * $item->diskon_paket,0,',','.') }}</td>
                                <td>{{ number_format($item->sub_total,0,',','.') }}</td>
                                <td>{{ $item->keterangan }}</td>
                            </tr>
                        @empty
                            <tr>
                                <td colspan="6" class="text-center">
                                    Tidak ada paket dipilih.
                                </td>
                            </tr>
                        @endforelse
                    </tbody>
                </table>
            </div>
            @if ($transaksi->dibayar == 'belum_dibayar')
                @include('transaksi.detail-form', ['transaksi' => $transaksi])
                @else
                @include('transaksi.detail-cash', ['transaksi' => $transaksi])
            @endif
        </div>
    </x-content>
@endsection
```

#### invoice.blade.php

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Invoice</title>
    <style>
        body {
            font-family: Arial, Helvetica, sans-serif;
            font-size: 12px;
        }

        .invoice {
            width: 70mm;
        }

        table {
            width: 100%;
        }

        .center {
            text-align: center;
        }

        .right {
            text-align: right;
        }

        hr {
            border-top: 1px solid #8c8b8b;
        }
    </style>
</head>
<body onload="javascript:window.print()">
    <div class="invoice">
        <h3 class="center">{{ $outlet->nama }}</h3>
        <p class="center">
            {{ $outlet->alamat }} <br> {{ $outlet->tlp }}
        </p>
        <hr>
        <p>
            Kode Transaksi : {{ $transaksi->kode_invoice }} <br>
            Tanggal Transaksi : {{ date('d/m/Y', strtotime($transaksi->tgl)) }} <br>
            Nama Pelanggan : {{ $member->nama }} <br>
            Kasir : {{ $user->nama }}
        </p>
        <hr>

        <table>
            @foreach ($items as $item)
                <tr>
                    <td>
                        {{ $item->qty }} {{ $item->nama_paket }}
                        x {{ number_format($item->harga,0,',','.') }} <br>
                        Diskon : {{ $item->diskon_paket }} <br>
                        <small>Ket : {{ $item->keterangan }}</small>
                    </td>
                    <td class="right">
                        @if ($item->diskon_paket != null)
                        <del>{{ number_format($item->harga * $item->qty,0,',','.') }}</del>
                        {{ number_format($item->sub_total,0,',','.') }}
                        @else
                        {{ number_format($item->sub_total,0,',','.') }}
                        @endif
                    </td>
                </tr>
            @endforeach
        </table>

        <hr>

        <p class="right">
            Sub Total : {{ number_format($transaksi->sub_total,0,',','.') }} <br>
            Diskon Tambahan : {{ number_format($transaksi->diskon,0,',','.') }} <br>
            Biaya Tambahan : {{ number_format($transaksi->biaya_tambahan,0,',','.') }}
            <br>
            Pajak PPN(10%) : {{ number_format($transaksi->pajak,0,',','.') }} <br>
            Total : {{ number_format($transaksi->total_bayar,0,',','.') }}
            @if ($transaksi->dibayar == 'dibayar')
            <br>
                Tunai : {{ number_format($transaksi->cash,0,',','.') }} <br>
                Kembalian : {{ number_format($transaksi->kembalian,0,',','.') }} <br>
            @endif
            @if ($transaksi->dibayar == 'dibayar')
                <h3 class="center">Terima Kasih</h3>
            @endif
        </p>
        <p class="right">
            @if ($transaksi->tgl_diambil == null)
                <small>
                    <i>
                    </i>
                </small>
                @else
                <small>
                    <i>
                    Diambil: {{ date('d-M-Y', strtotime($transaksi->tgl_diambil)) }}
                    </i>
                </small>
            @endif
        </p>
    </div>
</body>
</html>
```

#### tambahkan diskon_paket ke transaksi_details

```sh
php artisan make:migration add_diskon_paket_to_transaksi_details_table --table=transaksi_details
```

```php
    public function up()
    {
        Schema::create('transaksi_details', function (Blueprint $table) {
            $table->integer('diskon_paket')->nullable()->after('harga');
        });
    }
```

### isi fillable pada model TransaksiDetail

```php
class TransaksiDetail extends Model
{
    use HasFactory;

    public $timestamps = false;

    protected $fillable = [
        'transaksi_id',
        'paket_id',
        'harga',
        'diskon_paket',
        'qty',
        'sub_total',
        'keterangan',
    ];
}
```
