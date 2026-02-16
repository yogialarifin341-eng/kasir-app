# kasir-app
Aplikasi kasir sederhana

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Super Kasir</title>

<meta name="viewport" content="width=device-width, initial-scale=1">

<style>

body{
font-family: Arial;
background:#f0f2f5;
padding:20px;
}

.container{
background:white;
padding:20px;
border-radius:12px;
box-shadow:0 0 10px rgba(0,0,0,0.1);
max-width:500px;
margin:auto;
}

input, select{
width:100%;
padding:10px;
margin-top:10px;
}

button{
width:100%;
padding:12px;
margin-top:10px;
border:none;
border-radius:8px;
background:#4CAF50;
color:white;
font-size:16px;
}

button:hover{
background:#45a049;
}

.total{
font-size:22px;
font-weight:bold;
margin-top:20px;
}

.struk{
background:#eee;
padding:10px;
margin-top:20px;
white-space:pre-line;
}

</style>
</head>

<body>

<div class="container">

<h2>🧾 SUPER KASIR</h2>

<h3>Produk Database</h3>

<select id="produkSelect"></select>
<input type="number" id="jumlahDB" placeholder="Jumlah">
<button onclick="tambahDB()">Tambah Dari Database</button>

<hr>

<h3>Tambah Produk Manual</h3>

<input type="text" id="namaManual" placeholder="Nama Barang">
<input type="number" id="hargaManual" placeholder="Harga">
<input type="number" id="jumlahManual" placeholder="Jumlah">

<button onclick="tambahManual()">Tambah Manual</button>

<hr>

<button onclick="resetKasir()">Reset Transaksi</button>

<h3>Daftar Belanja</h3>
<ul id="list"></ul>

<div class="total">Total: Rp <span id="total">0</span></div>

<hr>

<h3>Pembayaran</h3>
<input type="number" id="bayar" placeholder="Uang Pembeli">
<button onclick="bayar()">Hitung Kembalian</button>

<div id="kembalian"></div>

<button onclick="printStruk()">Cetak Struk</button>

<div class="struk" id="strukArea"></div>

</div>

<script>

// ===== DATABASE =====
let produkDB = JSON.parse(localStorage.getItem("produkDB")) || [
{nama:"Indomie", harga:3500, stok:50},
{nama:"Kopi", harga:2000, stok:30},
{nama:"Teh", harga:1500, stok:40}
];

let transaksi = [];
let total = 0;

// ===== LOAD PRODUK =====
function loadProduk(){
let select = document.getElementById("produkSelect");
select.innerHTML="";

produkDB.forEach((p,i)=>{
let option = document.createElement("option");
option.value=i;
option.text=p.nama+" - Rp "+p.harga+" (Stok:"+p.stok+")";
select.appendChild(option);
});
}

loadProduk();

// ===== TAMBAH DB =====
function tambahDB(){

let index=document.getElementById("produkSelect").value;
let jumlah=parseInt(document.getElementById("jumlahDB").value);

let p=produkDB[index];

if(!jumlah || jumlah>p.stok){
alert("Jumlah salah / stok kurang");
return;
}

let subtotal=p.harga*jumlah;
total+=subtotal;

p.stok-=jumlah;

transaksi.push({
nama:p.nama,
harga:p.harga,
jumlah:jumlah,
subtotal:subtotal
});

render();

simpanData();

}

// ===== TAMBAH MANUAL =====
function tambahManual(){

let nama=document.getElementById("namaManual").value;
let harga=parseInt(document.getElementById("hargaManual").value);
let jumlah=parseInt(document.getElementById("jumlahManual").value);

if(!nama||!harga||!jumlah){
alert("Isi data lengkap");
return;
}

let subtotal=harga*jumlah;
total+=subtotal;

transaksi.push({nama,harga,jumlah,subtotal});

render();
simpanData();

}

// ===== RENDER =====
function render(){

let list=document.getElementById("list");
list.innerHTML="";

transaksi.forEach((t,i)=>{
let li=document.createElement("li");
li.innerHTML=`${t.nama} | ${t.jumlah} x Rp ${t.harga} = Rp ${t.subtotal}
<button onclick="hapusItem(${i})">❌</button>`;
list.appendChild(li);
});

document.getElementById("total").innerText=total;
loadProduk();
}

// ===== HAPUS ITEM =====
function hapusItem(i){
total-=transaksi[i].subtotal;
transaksi.splice(i,1);
render();
simpanData();
}

// ===== RESET =====
function resetKasir(){
transaksi=[];
total=0;
render();
localStorage.removeItem("transaksi");
}

// ===== BAYAR =====
function bayar(){

let uang=parseInt(document.getElementById("bayar").value);

if(uang<total){
alert("Uang kurang bro!");
return;
}

let kembali=uang-total;

document.getElementById("kembalian").innerText=
"Kembalian: Rp "+kembali;

buatStruk(uang,kembali);

}

// ===== STRUK =====
function buatStruk(uang,kembali){

let teks="===== STRUK BELANJA =====\n";

transaksi.forEach(t=>{
teks+=`${t.nama} ${t.jumlah} x ${t.harga} = ${t.subtotal}\n`;
});

teks+=`\nTotal : ${total}`;
teks+=`\nBayar : ${uang}`;
teks+=`\nKembali : ${kembali}`;

document.getElementById("strukArea").innerText=teks;

}

// ===== PRINT =====
function printStruk(){
window.print();
}

// ===== SIMPAN =====
function simpanData(){
localStorage.setItem("transaksi",JSON.stringify(transaksi));
localStorage.setItem("produkDB",JSON.stringify(produkDB));
localStorage.setItem("total",total);
}

// ===== LOAD =====
function loadData(){

let t=JSON.parse(localStorage.getItem("transaksi"));
let tot=localStorage.getItem("total");

if(t){
transaksi=t;
total=parseInt(tot);
render();
}
}

loadData();


// ===== PWA INSTALL =====
if('serviceWorker' in navigator){
navigator.serviceWorker.register('sw.js');
}

</script>

</body>
</html>
