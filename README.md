# Tutorial HTB (Hierarchical Token Bucket) MikroTik

## Identitas Tim
Video tutorial ini dibuat oleh tim dari [Nama Sekolah] dengan anggota [Nama Siswa 1] dan [Nama Siswa 2]. Nomor peserta kami adalah [Nomor Peserta].

---

## Prolog
Assalamualaikum Warahmatullahi Wabarakatuh.  
Pada kesempatan kali ini kami akan menjelaskan mengenai Hierarchical Token Bucket atau HTB, yaitu mekanisme manajemen bandwidth pada MikroTik yang bekerja dengan konsep parent dan child queue. Parent queue berfungsi sebagai wadah utama yang menentukan total kapasitas bandwidth, sementara child queue digunakan untuk membagi bandwidth tersebut ke setiap user atau subnet. Dengan HTB, administrator jaringan dapat mengatur distribusi bandwidth secara terstruktur, adil, dan fleksibel.  

Berbeda dengan Simple Queue yang memang lebih mudah digunakan, HTB memiliki keunggulan pada fleksibilitas dan efisiensi terutama ketika jumlah user semakin banyak. HTB memungkinkan pembagian bandwidth minimum dan maksimum serta penggunaan prioritas, sehingga performa jaringan tetap stabil meskipun dalam kondisi padat.  

Pada tutorial ini, kami akan menjelaskan konsep dasar HTB, komponen yang diperlukan, konfigurasi mangle, konfigurasi queue tree, serta studi kasus sederhana pembagian bandwidth untuk dua user dengan total bandwidth sepuluh megabit per detik.

---

## Konsep Dasar HTB
HTB bekerja dengan membagi bandwidth total yang dimiliki sebuah jaringan ke dalam struktur hirarkis. Parent queue menentukan kapasitas total bandwidth, sedangkan child queue membagi bandwidth tersebut ke user tertentu. Dengan model hirarki ini, administrator dapat memberikan batasan dan jaminan kecepatan kepada setiap user sesuai kebutuhan.  

Konsep ini memungkinkan bandwidth tidak hanya dibagi rata, tetapi juga diatur agar lebih adil. Misalnya, ketika satu user tidak aktif, bandwidth yang tidak terpakai dapat dipakai oleh user lain. Sebaliknya, ketika jaringan padat, setiap user tetap dijamin mendapat kapasitas minimum tertentu. Inilah alasan HTB dianggap lebih unggul daripada Simple Queue.

---

## Komponen Konfigurasi HTB
Konfigurasi HTB pada MikroTik membutuhkan dua komponen utama yaitu mangle dan queue tree. Mangle digunakan untuk memberi tanda pada trafik sehingga MikroTik dapat mengenali data yang berasal dari masing-masing user. Queue tree kemudian memanfaatkan tanda tersebut untuk membagi bandwidth sesuai aturan yang telah ditentukan.  

Kedua komponen ini saling melengkapi. Mangle bertugas untuk klasifikasi, sedangkan queue tree bertugas untuk pengaturan bandwidth. Tanpa mangle, queue tree tidak dapat membedakan trafik antar user. Sebaliknya, tanpa queue tree, tanda dari mangle tidak memiliki fungsi nyata dalam manajemen bandwidth.

---

## Mangle
Pada tahap awal, konfigurasi dilakukan di menu IP Firewall Mangle. Chain yang digunakan adalah prerouting karena marking perlu dilakukan sebelum proses routing berlangsung. Dengan chain prerouting, semua paket yang masuk bisa dikenali sejak awal, sehingga pembagian bandwidth akan berjalan konsisten.  

Source address diisi dengan subnet dari masing-masing user. Misalnya user pertama menggunakan jaringan 172.168.10.0/24 dan user kedua menggunakan jaringan 172.168.20.0/24. Rule pertama digunakan untuk menandai koneksi dari subnet 172.168.10.0/24 dengan action mark connection. Nama connection mark dapat dibuat connection user1. Opsi passthrough diaktifkan agar rule selanjutnya tetap diproses. Rule kedua dibuat untuk subnet 172.168.20.0/24 dengan cara yang sama, menggunakan connection mark bernama connection user2.  

Setelah koneksi ditandai, rule baru ditambahkan untuk menandai paket. Pada rule ketiga, action yang digunakan adalah mark packet dengan mengacu pada connection mark connection user1. Packet mark baru dibuat dengan nama packet user1. Pada rule keempat, dilakukan hal serupa untuk connection user2 dengan packet mark bernama packet user2.  

Dengan demikian, setiap paket data dari subnet 172.168.10.0/24 dan 172.168.20.0/24 akan memiliki tanda khusus yang dapat digunakan di queue tree. Bagian advanced dan extra pada mangle tidak perlu diisi jika tidak ada kebutuhan filter tambahan seperti DSCP atau TTL. Inti dari mangle adalah memastikan setiap subnet atau user memiliki packet mark yang berbeda agar bisa diproses lebih lanjut oleh queue tree.

---

## Queue Tree
Setelah proses marking selesai, konfigurasi dilanjutkan ke queue tree. Parent queue dibuat pada interface yang digunakan untuk mengakses internet, misalnya ether1. Pada parent ini ditentukan total bandwidth, misalnya sepuluh megabit per detik. Parent queue berperan sebagai pembatas global, sehingga semua child akan tunduk pada kapasitas yang ditentukan parent.  

Child queue kemudian dibuat untuk masing-masing packet mark. Child queue pertama dibuat untuk packet user1 dan child queue kedua untuk packet user2. Pada child queue terdapat dua parameter penting yaitu CIR dan MIR. CIR atau Committed Information Rate adalah bandwidth minimum yang dijamin akan didapatkan sebuah child queue meskipun jaringan sedang padat. Dengan adanya CIR, setiap user memiliki jaminan kecepatan tertentu sehingga koneksi tidak putus. MIR atau Max Limit adalah batas maksimum bandwidth yang boleh digunakan child queue. Jika jaringan tidak padat, user dapat menggunakan bandwidth hingga MIR, namun ketika jaringan padat, alokasi akan menurun kembali ke nilai CIR.  

Dengan kata lain, CIR berfungsi sebagai jaminan minimum, sementara MIR menjadi batas maksimum. Kombinasi keduanya menjadikan HTB lebih fleksibel dan adil dibandingkan Simple Queue. Dalam studi kasus pembagian bandwidth sepuluh megabit per detik untuk dua user, jika CIR masing-masing child queue ditentukan sebesar tiga megabit per detik dan MIR ditentukan sebesar lima megabit per detik, maka setiap user dijamin mendapatkan minimal tiga megabit. Namun saat kondisi idle atau ketika user lain tidak menggunakan jaringan, bandwidth dapat meningkat hingga lima megabit.  

Selain itu, parameter priority juga dapat diatur. Child queue dengan angka priority lebih kecil akan diproses lebih dahulu ketika jaringan padat. Dengan demikian, administrator dapat memberikan prioritas pada user tertentu jika diperlukan.

---

## Studi Kasus
Topologi percobaan terdiri dari satu router MikroTik dengan koneksi internet sepuluh megabit per detik. Terdapat dua client yaitu user pertama pada subnet 172.168.10.0/24 dan user kedua pada subnet 172.168.20.0/24.  

Pertama, dilakukan konfigurasi mangle untuk menandai koneksi dan paket dari kedua subnet. Kemudian dibuat parent queue pada interface internet dengan kapasitas sepuluh megabit per detik. Setelah itu, dua child queue dibuat dengan mengacu pada packet mark masing-masing user. Masing-masing child diberikan CIR tiga megabit per detik dan MIR lima megabit per detik.  

Pengujian dilakukan dengan cara user pertama melakukan download file berukuran besar sementara user kedua tidak aktif. Hasilnya, user pertama mampu memanfaatkan bandwidth hingga lima megabit per detik. Ketika user kedua mulai melakukan download bersamaan, alokasi bandwidth terbagi rata sesuai konfigurasi. Masing-masing user tetap mendapat jaminan minimal tiga megabit per detik, sementara bandwidth lebih hanya bisa digunakan ketika tersedia.  

Dengan demikian terbukti bahwa HTB mampu menjaga keadilan distribusi bandwidth sekaligus memberikan fleksibilitas pemanfaatan kapasitas.

---

## Penutup
HTB merupakan mekanisme manajemen bandwidth yang lebih fleksibel dan adil dibandingkan Simple Queue. Dengan menggunakan marking melalui mangle dan pembagian bandwidth melalui queue tree, administrator dapat menjamin kapasitas minimum melalui CIR sekaligus membatasi kapasitas maksimum melalui MIR.  

Dalam studi kasus dua user, HTB berhasil menjamin bandwidth minimum untuk setiap user sekaligus memungkinkan pemanfaatan bandwidth lebih ketika salah satu user tidak aktif. Hal ini menunjukkan bahwa HTB adalah solusi ideal untuk jaringan dengan jumlah user yang banyak dan kebutuhan manajemen bandwidth yang presisi.  

Sekian penjelasan dari kami. Wassalamualaikum Warahmatullahi Wabarakatuh.
