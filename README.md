# Tutorial HTB (Hierarchical Token Bucket) MikroTik

## Identitas Tim
Video tutorial ini dibuat oleh tim dari [Nama Sekolah] dengan anggota [Nama Siswa 1] dan [Nama Siswa 2]. Nomor peserta kami adalah [Nomor Peserta].

---

## Prolog
Assalamualaikum Warahmatullahi Wabarakatuh.  
Pada kesempatan kali ini kami akan menjelaskan mengenai Hierarchical Token Bucket atau HTB, yaitu mekanisme manajemen bandwidth pada MikroTik yang bekerja dengan konsep parent dan child queue. Parent queue berfungsi sebagai wadah utama yang menentukan total kapasitas bandwidth, sementara child queue digunakan untuk membagi bandwidth tersebut ke setiap user atau subnet. Dengan HTB, administrator jaringan dapat mengatur distribusi bandwidth secara terstruktur, adil, dan fleksibel.  

Berbeda dengan Simple Queue yang memang lebih mudah digunakan, HTB memiliki keunggulan pada fleksibilitas dan efisiensi terutama ketika jumlah user semakin banyak. HTB memungkinkan pembagian bandwidth minimum dan maksimum serta penggunaan prioritas, sehingga performa jaringan tetap stabil meskipun dalam kondisi padat.  

Pada tutorial ini, kami akan menjelaskan konsep dasar HTB, komponen yang diperlukan, teori konfigurasi mangle, serta studi kasus pembagian bandwidth upload dan download untuk dua user dengan total bandwidth sepuluh megabit per detik download dan dua megabit per detik upload.

---

## Konsep Dasar HTB
HTB bekerja dengan membagi bandwidth total yang dimiliki sebuah jaringan ke dalam struktur hirarkis. Parent queue menentukan kapasitas total bandwidth, sedangkan child queue membagi bandwidth tersebut ke user tertentu. Dengan model hirarki ini, administrator dapat memberikan batasan dan jaminan kecepatan kepada setiap user sesuai kebutuhan.  

Konsep ini memungkinkan bandwidth tidak hanya dibagi rata, tetapi juga diatur agar lebih adil. Misalnya, ketika satu user tidak aktif, bandwidth yang tidak terpakai dapat dipakai oleh user lain. Sebaliknya, ketika jaringan padat, setiap user tetap dijamin mendapat kapasitas minimum tertentu. Inilah alasan HTB dianggap lebih unggul dibandingkan Simple Queue.

---

## Komponen Konfigurasi HTB
Konfigurasi HTB pada MikroTik membutuhkan dua komponen utama yaitu mangle dan queue tree. Mangle digunakan untuk memberi tanda pada trafik sehingga MikroTik dapat mengenali data yang berasal dari masing-masing user. Queue tree kemudian memanfaatkan tanda tersebut untuk membagi bandwidth sesuai aturan yang telah ditentukan.  

Kedua komponen ini saling melengkapi. Mangle bertugas untuk klasifikasi, sedangkan queue tree bertugas untuk pengaturan bandwidth. Tanpa mangle, queue tree tidak dapat membedakan trafik antar user. Sebaliknya, tanpa queue tree, tanda dari mangle tidak memiliki fungsi nyata dalam manajemen bandwidth.

---

## Mangle (Teori)
Pada tahap awal, konfigurasi dilakukan di menu IP Firewall Mangle. Chain yang digunakan adalah prerouting karena marking perlu dilakukan sebelum proses routing berlangsung. Dengan chain prerouting, semua paket yang masuk bisa dikenali sejak awal, sehingga pembagian bandwidth akan berjalan konsisten.  

Source address diisi dengan subnet dari masing-masing user. Misalnya user pertama menggunakan jaringan 172.168.10.0/24 dan user kedua menggunakan jaringan 172.168.20.0/24. Rule pertama digunakan untuk menandai koneksi dari subnet 172.168.10.0/24 dengan action mark connection. Nama connection mark dapat dibuat connection user1. Opsi passthrough diaktifkan agar rule selanjutnya tetap diproses. Rule kedua dibuat untuk subnet 172.168.20.0/24 dengan cara yang sama, menggunakan connection mark bernama connection user2.  

Setelah koneksi ditandai, rule baru ditambahkan untuk menandai paket. Pada rule ketiga, action yang digunakan adalah mark packet dengan mengacu pada connection mark connection user1. Packet mark baru dibuat dengan nama packet user1. Pada rule keempat, dilakukan hal serupa untuk connection user2 dengan packet mark bernama packet user2.  

Bagian advanced dan extra pada mangle tidak perlu diisi jika tidak ada kebutuhan filter tambahan seperti DSCP atau TTL. Inti dari mangle adalah memastikan setiap subnet atau user memiliki packet mark yang berbeda agar bisa diproses lebih lanjut oleh queue tree.

---

## Studi Kasus
Studi kasus dilakukan pada jaringan dengan total bandwidth sepuluh megabit per detik untuk download dan dua megabit per detik untuk upload. Terdapat dua client, yaitu user pertama pada subnet 172.168.10.0/24 dan user kedua pada subnet 172.168.20.0/24.  

### Konfigurasi Mangle (Praktik)
Langkah pertama adalah membuat rule mangle untuk kedua user. Rule pertama menandai koneksi dari subnet 172.168.10.0/24 dengan connection mark connection user1. Rule kedua menandai koneksi dari subnet 172.168.20.0/24 dengan connection mark connection user2. Kemudian dibuat rule tambahan untuk memberi packet mark, yaitu packet user1 untuk connection user1 dan packet user2 untuk connection user2.  

Dengan konfigurasi ini, setiap paket dari user pertama dan user kedua akan diberi tanda sehingga dapat dibedakan di queue tree.

### Konfigurasi Queue Tree
Setelah mangle selesai, konfigurasi dilanjutkan dengan membuat queue tree. Karena bandwidth upload dan download berbeda, dibuat dua parent queue. Parent pertama ditempatkan pada interface WAN atau global-in dengan kapasitas sepuluh megabit per detik untuk download. Parent kedua ditempatkan pada interface LAN atau global-out dengan kapasitas dua megabit per detik untuk upload.  

Child queue kemudian dibuat untuk masing-masing packet mark. Untuk download, dibuat child queue pertama untuk packet user1 dan child queue kedua untuk packet user2. Untuk upload, dilakukan hal yang sama, child queue pertama untuk packet user1 dan child queue kedua untuk packet user2.  

Setiap child queue diberikan CIR sebesar tiga megabit per detik untuk download dan enam ratus kilobit per detik untuk upload, serta MIR sebesar lima megabit per detik untuk download dan satu megabit per detik untuk upload. Dengan konfigurasi ini, setiap user dijamin mendapatkan bandwidth minimum baik pada arah download maupun upload, dan dapat memanfaatkan bandwidth lebih ketika jaringan tidak padat.  

### Pengujian
Pengujian dilakukan dengan cara user pertama melakukan download file berukuran besar sementara user kedua tidak aktif. Hasilnya, user pertama mampu memanfaatkan bandwidth hingga lima megabit per detik pada arah download. Ketika user kedua mulai melakukan download bersamaan, alokasi bandwidth terbagi sesuai konfigurasi. Masing-masing user tetap mendapat jaminan minimal tiga megabit per detik, sementara bandwidth lebih hanya bisa digunakan ketika tersedia.  

Pada arah upload, ketika hanya satu user yang aktif, bandwidth dapat mencapai maksimal satu megabit per detik. Namun ketika kedua user melakukan upload bersamaan, masing-masing mendapat alokasi minimal enam ratus kilobit per detik. Dengan demikian, distribusi bandwidth tetap adil baik pada arah download maupun upload.

---

## Penutup
HTB merupakan mekanisme manajemen bandwidth yang lebih fleksibel dan adil dibandingkan Simple Queue. Dengan menggunakan marking melalui mangle dan pembagian bandwidth melalui queue tree, administrator dapat menjamin kapasitas minimum melalui CIR sekaligus membatasi kapasitas maksimum melalui MIR.  

Dalam studi kasus dua user dengan bandwidth sepuluh megabit per detik download dan dua megabit per detik upload, HTB berhasil menjamin bandwidth minimum untuk setiap user sekaligus memungkinkan pemanfaatan bandwidth lebih ketika salah satu user tidak aktif. Hal ini menunjukkan bahwa HTB adalah solusi ideal untuk jaringan dengan jumlah user yang banyak dan kebutuhan manajemen bandwidth yang presisi.  

Sekian penjelasan dari kami. Wassalamualaikum Warahmatullahi Wabarakatuh.
