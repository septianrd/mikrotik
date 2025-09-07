# Tutorial HTB (Hierarchical Token Bucket) MikroTik

## Identitas Tim
Video tutorial ini dibuat oleh tim dari [Nama Sekolah] dengan anggota [Nama Siswa 1] dan [Nama Siswa 2].  
Nomor peserta kami adalah [Nomor Peserta].  

---

## Prolog
Assalamualaikum Warahmatullahi Wabarakatuh.  
Pada kesempatan kali ini kami akan menjelaskan mengenai Hierarchical Token Bucket atau HTB, yaitu mekanisme manajemen bandwidth pada MikroTik yang bekerja dengan konsep parent dan child queue. Parent queue berfungsi sebagai wadah utama yang menentukan total kapasitas bandwidth, sementara child queue digunakan untuk membagi bandwidth tersebut ke setiap user atau subnet. Dengan HTB, administrator jaringan dapat mengatur distribusi bandwidth secara terstruktur, adil, dan fleksibel.  

Berbeda dengan Simple Queue yang lebih sederhana, HTB memiliki keunggulan pada fleksibilitas dan efisiensi terutama ketika jumlah user semakin banyak. HTB memungkinkan pembagian bandwidth minimum dan maksimum serta penggunaan prioritas, sehingga performa jaringan tetap stabil meskipun dalam kondisi padat.  

---

## Konsep Dasar HTB
HTB bekerja dengan membagi bandwidth total yang dimiliki sebuah jaringan ke dalam struktur hirarkis. Parent queue menentukan kapasitas total bandwidth, sedangkan child queue membagi bandwidth tersebut ke user tertentu. Dengan model hirarki ini, administrator dapat memberikan batasan dan jaminan kecepatan kepada setiap user sesuai kebutuhan.  

Dengan konsep hirarki, bandwidth tidak hanya dibagi rata, tetapi juga diatur agar lebih adil. Ketika satu user tidak aktif, bandwidth yang tidak terpakai dapat dipakai oleh user lain. Sebaliknya, ketika jaringan padat, setiap user tetap mendapatkan jatah minimum sesuai konfigurasi.  

---

## Inner Queue dan Leaf Queue
Dalam HTB terdapat dua jenis queue, yaitu inner queue dan leaf queue.  

Inner queue adalah queue induk atau parent yang berfungsi sebagai wadah pembagi bandwidth. Inner queue biasanya tidak mengikat langsung ke packet mark, melainkan hanya menetapkan kapasitas total.  

Sebaliknya, leaf queue adalah child queue yang langsung mengikat ke packet mark hasil mangle. Leaf queue inilah yang benar-benar melakukan pembatasan atau shaping pada trafik. Parameter seperti Committed Information Rate (CIR), Max Limit (MIR), dan priority diterapkan pada leaf queue.  

Struktur umum HTB dapat digambarkan sebagai berikut:  
- Parent queue sebagai inner queue, misalnya total bandwidth download sepuluh megabit per detik.  
- Child queue sebagai leaf queue, misalnya user satu dengan packet mark tertentu dan user dua dengan packet mark lainnya.  

Dengan pemisahan ini, manajemen bandwidth menjadi lebih terstruktur karena inner queue membatasi total kapasitas, sementara leaf queue membatasi trafik per user atau subnet.  

---

## Konfigurasi Mangle
Sebelum membuat queue tree, administrator perlu menandai trafik dengan menggunakan mangle. Mangle digunakan untuk memberi tanda pada koneksi atau paket agar dapat dikenali oleh queue.  

Prinsip konfigurasi mangle adalah sebagai berikut. Pertama, buat rule baru pada menu **IP > Firewall > Mangle**. Chain menggunakan `prerouting`, karena kita ingin menandai paket sedini mungkin sebelum masuk ke proses routing. Source address diisi dengan subnet user, misalnya `172.168.10.0/24`.  

Action pertama adalah `mark-connection`. Dengan ini, koneksi diberi tanda tertentu, misalnya `conn-client`. Parameter passthrough tetap diaktifkan agar rule selanjutnya tetap berjalan.  

Setelah itu, dibuat rule kedua dengan action `mark-packet`. Pada rule ini, connection mark yang sudah ada dipanggil, kemudian diarahkan ke `mark-packet`, misalnya `pkt-client`. Passthrough tetap diaktifkan.  

Bagian advanced dan extra tidak perlu diisi jika tidak ada kondisi tambahan. Dengan dua tahap ini, setiap trafik memiliki packet mark yang unik sehingga bisa dipisahkan ke dalam child queue.  

---

## Queue Tree (Teori)
<img width="1920" height="1080" alt="Mikrotik 2" src="https://github.com/user-attachments/assets/a1efe65f-4ce3-413d-af56-949e82329a9b" />

Setelah mangle selesai dibuat, kita lanjut ke konfigurasi queue tree. Karena bandwidth upload dan download berbeda, maka kita perlu membuat dua parent queue:  
- Parent download pada interface WAN.  
- Parent upload pada interface LAN.  

Di bawah setiap parent, dibuat child queue sesuai packet mark masing-masing user. Child queue inilah yang berfungsi sebagai leaf queue, yang langsung melakukan shaping pada trafik user.  

Dua parameter penting dalam child queue adalah:  
- **CIR (Committed Information Rate) / limit-at**, yaitu bandwidth minimum yang dijamin akan diterima user meskipun jaringan padat.  
- **MIR (Max Limit)**, yaitu bandwidth maksimum yang boleh digunakan user jika jaringan dalam kondisi idle.  

Dengan kombinasi CIR dan MIR, setiap user memperoleh jaminan minimum sekaligus fleksibilitas penggunaan bandwidth maksimum.  

---

## Studi Kasus
<img width="1920" height="1080" alt="Mikrotik 1" src="https://github.com/user-attachments/assets/0e0f6558-cb02-46e4-8448-7f18fb133b4c" />

Pada studi kasus ini, diasumsikan ISP memberikan bandwidth sepuluh megabit per detik untuk download dan enam megabit per detik untuk upload. Bandwidth tersebut akan dibagi ke dua kelompok user, yaitu Manager dan Karyawan.  

### 1. Mangle
Konsep mangle yang sudah dijelaskan sebelumnya sekarang diterapkan dengan subnet nyata. Untuk contoh sederhana, subnet `172.168.10.0/24` digunakan sebagai LAN utama. Trafik dari subnet ini diberi tanda connection mark `conn-client` kemudian diarahkan ke packet mark `pkt-client`.  

Dengan cara ini, trafik user sudah dapat dipisahkan dari jaringan lain dan siap dimasukkan ke queue tree.  

### 2. Queue Tree
- **Parent Download**: sepuluh megabit per detik di interface WAN.  
  - Child download Manager: CIR tiga megabit per detik, MIR lima megabit per detik, diarahkan ke `pkt-client`.  
  - Child download Karyawan: CIR tiga megabit per detik, MIR lima megabit per detik, diarahkan ke `pkt-client`.  
- **Parent Upload**: enam megabit per detik di interface LAN.  
  - Child upload Manager: CIR satu megabit per detik, MIR tiga megabit per detik, diarahkan ke `pkt-client`.  
  - Child upload Karyawan: CIR satu megabit per detik, MIR tiga megabit per detik, diarahkan ke `pkt-client`.  

Dengan konfigurasi ini, kedua user dijamin selalu mendapat bandwidth minimum baik pada arah download maupun upload. Namun ketika salah satu user tidak aktif, user lain dapat menggunakan bandwidth hingga batas maksimum yang ditentukan.  

---

## Kesimpulan
HTB pada MikroTik merupakan mekanisme manajemen bandwidth yang lebih fleksibel dibandingkan Simple Queue. Dengan konsep inner queue dan leaf queue, administrator dapat membuat struktur pembagian bandwidth yang teratur dan efisien.  

Melalui kombinasi CIR dan MIR, HTB tidak hanya memberikan jaminan bandwidth minimum tetapi juga fleksibilitas penggunaan maksimum. Studi kasus pembagian sepuluh megabit per detik download dan enam megabit per detik upload ke dua user menunjukkan bagaimana HTB dapat digunakan untuk membagi bandwidth secara adil dan adaptif sesuai kondisi jaringan.  

---

## Penutup
Demikian tutorial mengenai HTB MikroTik. Semoga dapat memberikan pemahaman tentang konsep dasar, komponen utama, konfigurasi mangle, dan penerapan queue tree dalam manajemen bandwidth.  

Wassalamualaikum Warahmatullahi Wabarakatuh.  
