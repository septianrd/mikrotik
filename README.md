# Tutorial HTB (Hierarchical Token Bucket) MikroTik
## Identitas Tim
Video tutorial ini dibuat oleh tim dari [Nama Sekolah] dengan anggota [Nama Siswa 1] dan [Nama Siswa 2].  
Nomor peserta kami adalah [Nomor Peserta].

---

## Prolog
Assalamualaikum Warahmatullahi Wabarakatuh.  
Pada kesempatan kali ini kami akan menjelaskan tentang manajemen bandwidth menggunakan Hierarchical Token Bucket atau HTB pada MikroTik. Sebenarnya, MikroTik sudah menyediakan **simple queue** untuk membatasi bandwidth, namun simple queue memiliki keterbatasan. Ketika jumlah user bertambah banyak, simple queue sering tidak efisien, sulit diatur prioritasnya, dan terkadang hasil pembagian bandwidth tidak konsisten. Oleh karena itu digunakan **HTB**.  

<img width="1920" height="1080" alt="Mikrotik 2" src="https://github.com/user-attachments/assets/b6a62220-0702-4959-a69d-e78f77c07adc" />

Keunggulan HTB adalah pembagian bandwidth yang lebih fleksibel karena memakai konsep **hierarki**. Di dalam HTB ada dua istilah penting: **inner queue** dan **leaf queue**. Inner queue adalah parent queue yang berfungsi sebagai wadah utama, biasanya dipasang di interface dengan kapasitas bandwidth total. Sedangkan leaf queue adalah child queue yang menempel pada inner queue, digunakan untuk mengikat packet mark tertentu dan membagi bandwidth sesuai kebutuhan. Dengan HTB, kita bisa memberikan bandwidth minimum, maksimum, serta prioritas untuk masing-masing user atau kelompok IP sehingga jaringan tetap adil dan efisien.

---

## Studi Kasus
<img width="1920" height="1080" alt="Mikrotik 3" src="https://github.com/user-attachments/assets/686d01fd-8274-4b47-b848-e90adf0c838c" />


Agar lebih mudah dipahami, mari kita ambil contoh kasus di sebuah SMK. Ada dua jurusan yang sama-sama menggunakan internet sekolah, yaitu jurusan SIJA dan jurusan Teknik Elektronika. Total bandwidth dari ISP adalah lima megabit per detik untuk download dan tiga megabit per detik untuk upload. Karena jurusan SIJA sering melakukan ujian online dan praktik jaringan, mereka membutuhkan bandwidth lebih besar dibanding Teknik Elektronika yang hanya memanfaatkan internet untuk browsing ringan.  

Untuk mempermudah pembagian, administrator jaringan membagi alamat IP. Jurusan SIJA mendapatkan rentang `172.168.10.2–172.168.10.20`, sedangkan Teknik Elektronika menggunakan `172.168.10.21–172.168.10.40`. Dengan skema ini, trafik bisa dipisahkan berdasarkan IP, ditandai dengan mangle, lalu diatur dengan queue tree menggunakan HTB.

---
<img width="1920" height="1080" alt="Mikrotik 1 - Copy" src="https://github.com/user-attachments/assets/ccdfe931-6644-4539-bd0e-d3e28db89ef1" />

## Mangle dalam Studi Kasus
Langkah pertama adalah membuat rule mangle untuk menandai trafik. Kenapa harus mangle? Karena queue tree hanya bisa bekerja jika trafik sudah diberi tanda. Kita pakai chain **prerouting** karena paket ditangani sejak awal sebelum diteruskan, sehingga semua trafik bisa ditandai baik upload maupun download.  

Pertama kita buat rule dengan action **mark-connection**. Rule ini akan menempelkan connection mark, misalnya `conn-sija` untuk jurusan SIJA dan `conn-elektro` untuk jurusan Teknik Elektronika. Connection mark berguna supaya semua paket dalam satu koneksi internet mendapat tanda yang sama. Opsi passthrough diaktifkan agar rule berikutnya tetap berjalan.  

Setelah itu dibuat rule kedua dengan action **mark-packet**. Rule ini mengambil connection mark tadi dan mengubahnya menjadi packet mark, misalnya `pkt-sija` dan `pkt-elektro`. Di bagian ini passthrough dimatikan, karena kita ingin setiap paket hanya memakai satu tanda yang jelas. Dengan cara ini, semua trafik sudah terbagi menjadi dua kelompok sesuai jurusannya.

---

## Queue Tree dalam Studi Kasus
Tahap berikutnya adalah membuat queue tree. Seperti dijelaskan di awal, ada inner queue sebagai parent dan leaf queue sebagai child.  

Pertama kita buat **inner queue** untuk download dengan parent `ether1` karena interface ini terhubung ke WAN. Kapasitasnya sesuai bandwidth ISP yaitu lima megabit. Kita juga buat inner queue upload dengan kapasitas tiga megabit di interface yang sama. Inner queue tidak perlu memakai packet mark karena fungsinya hanya sebagai wadah utama.  

Setelah itu kita buat **leaf queue**. Leaf queue selalu membutuhkan packet mark karena ia benar-benar mengikat trafik yang sudah ditandai di mangle. Untuk parent download, leaf queue pertama diarahkan ke `pkt-sija` dengan limit-at tiga megabit dan max-limit lima megabit. Leaf queue kedua diarahkan ke `pkt-elektro` dengan limit-at dua megabit dan max-limit tiga megabit. Demikian juga untuk parent upload, SIJA mendapat jaminan dua megabit dengan maksimum tiga, sedangkan Teknik Elektronika hanya satu megabit dengan maksimum dua.  

Parameter lain yang penting adalah **priority** yang menentukan urutan antrean ketika bandwidth penuh, **queue type** yang biasanya default `pcq` untuk distribusi rata, serta **bucket size** yang mengatur distribusi token bandwidth. Sementara **tab statistics** dipakai untuk memastikan trafik benar-benar masuk ke antrian yang sesuai dan sesuai dengan konfigurasi yang kita buat.

---

## Kesimpulan
Dari studi kasus ini, terlihat bahwa HTB lebih unggul dibanding simple queue dalam mengelola bandwidth. Dengan mangle, trafik dari jurusan SIJA dan Teknik Elektronika bisa dipisahkan jelas. Dengan queue tree, bandwidth bisa dibagi sesuai kebutuhan dengan inner queue sebagai pembatas utama dan leaf queue sebagai pembagi untuk setiap jurusan. Hasilnya, SIJA yang membutuhkan bandwidth lebih besar mendapatkan alokasi prioritas, sementara Teknik Elektronika tetap mendapat jatah yang sesuai.  

---

## Penutup
Demikianlah penjelasan mengenai HTB pada MikroTik beserta penerapannya dalam studi kasus di sebuah SMK. Dengan memahami konsep mangle, inner queue, dan leaf queue, administrator dapat mengatur bandwidth lebih adil dan efisien. Semoga penjelasan ini bermanfaat dan bisa diterapkan pada jaringan nyata.  

Wassalamualaikum Warahmatullahi Wabarakatuh.

---
# README - Konfigurasi HTB MikroTik (GUI Step by Step)

## Identitas
SMK [Nama Sekolah]  
Jurusan: SIJA & Teknik Pendingin  
Bandwidth ISP: Download 10 Mbps, Upload 3 Mbps

---

## Langkah-langkah Konfigurasi via GUI

1. **Buat Bridge**
   - Buka menu **Bridge > Bridge** → klik **+ Add**
   - Beri nama: `bridge1`

2. **Masukkan Port ke Bridge**
   - Buka menu **Bridge > Ports**
   - Tambahkan port `ether2` dan `ether3` ke `bridge1`

3. **Tambahkan IP Address ke Bridge**
   - Buka menu **IP > Address**
   - Tambahkan `172.168.10.1/24` ke interface `bridge1`

4. **Konfigurasi NAT (biar client bisa internet)**
   - Masuk ke **IP > Firewall > NAT**
   - Tambahkan rule baru:
     - Chain: `srcnat`
     - Out Interface: `wlan1`
     - Action: `masquerade`

5. **Mangle – Tandai Koneksi (PREROUTING)**
   - Masuk ke **IP > Firewall > Mangle**
   - Rule 1: Jurusan SIJA  
     - Chain: `prerouting`  
     - Src-Address: `172.168.10.2-172.168.10.5`  
     - Action: `mark-connection` → `con-sija`  
     - Passthrough: **Yes**  
   - Rule 2: Jurusan Pendingin  
     - Chain: `prerouting`  
     - Src-Address: `172.168.10.6-172.168.10.10`  
     - Action: `mark-connection` → `con-pendingin`  
     - Passthrough: **Yes**

6. **Mangle – Tandai Paket (FORWARD)**
   - Rule 3: SIJA Download  
     - Chain: `forward`  
     - Connection Mark: `con-sija`  
     - Out Interface: `bridge1`  
     - Action: `mark-packet` → `sija-download`  
     - Passthrough: **No**
   - Rule 4: Pendingin Download  
     - Chain: `forward`  
     - Connection Mark: `con-pendingin`  
     - Out Interface: `bridge1`  
     - Action: `mark-packet` → `pendingin-download`  
     - Passthrough: **No**
   - Rule 5: SIJA Upload  
     - Chain: `forward`  
     - Connection Mark: `con-sija`  
     - Out Interface: `wlan1`  
     - Action: `mark-packet` → `sija-upload`  
     - Passthrough: **No**
   - Rule 6: Pendingin Upload  
     - Chain: `forward`  
     - Connection Mark: `con-pendingin`  
     - Out Interface: `wlan1`  
     - Action: `mark-packet` → `pendingin-upload`  
     - Passthrough: **No**

7. **Queue Tree – Parent Download**
   - Buka **Queues > Queue Tree**
   - Buat Parent Queue:  
     - Name: `parent-download`  
     - Parent: `bridge1`  
     - Max-Limit: `10M`  
   - Child SIJA Download:  
     - Name: `sija-download`  
     - Parent: `parent-download`  
     - Packet Mark: `sija-download`  
     - Limit-at: `3M`  
     - Max-Limit: `5M`  
     - Priority: `1`  
   - Child Pendingin Download:  
     - Name: `pendingin-download`  
     - Parent: `parent-download`  
     - Packet Mark: `pendingin-download`  
     - Limit-at: `2M`  
     - Max-Limit: `3M`  
     - Priority: `8`  

8. **Queue Tree – Parent Upload**
   - Buat Parent Queue:  
     - Name: `parent-upload`  
     - Parent: `bridge1`  
     - Max-Limit: `3M`  
   - Child SIJA Upload:  
     - Name: `sija-upload`  
     - Parent: `parent-upload`  
     - Packet Mark: `sija-upload`  
     - Limit-at: `1M`  
     - Max-Limit: `2M`  
     - Priority: `1`  
   - Child Pendingin Upload:  
     - Name: `pendingin-upload`  
     - Parent: `parent-upload`  
     - Packet Mark: `pendingin-upload`  
     - Limit-at: `1M`  
     - Max-Limit: `2M`  
     - Priority: `8`  

---

## Catatan
- **Limit-at** = jatah minimum bandwidth yang dijamin.  
- **Max-limit** = batas maksimal yang bisa dipakai kalau bandwidth kosong.  
- **Priority** = angka lebih kecil berarti lebih diprioritaskan.  
- **Parent Queue** = tidak pakai packet-mark, hanya jadi wadah bandwidth.  
- **Child Queue** = wajib pakai packet-mark hasil mangle.  

---
