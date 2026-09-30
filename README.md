# Tugas Keamanan Informasi 

Repositori ini berisi implementasi simulasi komunikasi jaringan dua arah (Client-Server) menggunakan protokol TCP/IP yang diamankan dengan algoritma **Data Encryption Standard (DES)**. Seluruh algoritma kriptografi dibangun secara manual dari awal (*from scratch*) tanpa menggunakan *library* kriptografi bawaan.

**Identitas:**
- **Nama:** Muhammad Hilbran Akmal Abrar
- **Departemen:** Teknik Informatika, Institut Teknologi Sepuluh Nopember (ITS)

## Alur Kerja Program

Secara garis besar, alur pengiriman pesan rahasia pada aplikasi ini adalah sebagai berikut:
1. **Koneksi Jaringan:** Komputer A (Server) membuka *port* 5000 dan menunggu. Komputer B (Client) menyambung ke IP Komputer A. Keduanya saling terhubung.
2. **Input Teks:** Pengguna mengetik pesan teks (Plaintext) di terminal.
3. **Proses Enkripsi:** Teks tersebut diolah oleh algoritma DES menggunakan *Secret Key* (kunci rahasia) menjadi data acak biner. Biner ini kemudian diubah ke format Hexadecimal (Ciphertext) agar aman dan stabil saat dikirim melintasi jaringan.
4. **Transmisi:** Ciphertext Hexadecimal dikirim melalui *socket* TCP jaringan (contoh: dari Host Windows ke Virtual Machine Ubuntu).
5. **Proses Dekripsi:** Komputer penerima menangkap pesan Hexadecimal tersebut, lalu mendekripsinya menggunakan *Secret Key* yang sama untuk mengubahnya kembali menjadi pesan asli (Plaintext) di layar.

## Penjelasan Kode

Proyek ini menggunakan konsep *Modular Programming* dan dipisah menjadi dua file utama:

### 1. `cipher.py` (Inti Kriptografi DES)
File ini berisi seluruh fungsi dan rumus matematika dari algoritma DES. DES adalah *block cipher* yang mengenkripsi data per blok 64 bit (8 byte) dengan *key* 64 bit (56 bit efektif).
- **Fungsi `generate_keys()`:** Bertugas mengambil *key* rahasia, lalu membaginya menjadi blok C dan D. Di setiap ronde, bagian C dan D dirotasi ke kiri lalu dipermutasi dengan PC2 (56 bit $\rightarrow$ 48 bit) untuk menghasilkan 16 *subkey* (kunci anak) yang berbeda.
- **Fungsi `des_block()`:** Ini adalah mesin utama Jaringan Feistel. Blok pesan 64-bit dipecah melalui Initial Permutation (IP) menjadi L dan R (masing-masing 32 bit)[cite: 7]. Selama 16 ronde, data ini mengalami proses berulang:
  - Ekspansi E (32 bit $\rightarrow$ 48 bit), lalu di-XOR dengan *subkey*[cite: 7].
  - Substitusi 8 S-Box (6 bit $\rightarrow$ 4 bit per S-box) yang bertindak sebagai bagian non-linear pemecah pola[cite: 7].
  - Permutasi P (32 bit) untuk mengacak posisi[cite: 7].
- **Fungsi `encrypt_des()`:** Bertugas memotong pesan panjang menjadi blok-blok 8 karakter. Fungsi ini memiliki mekanisme **Padding** untuk menggenapkan panjang pesan menjadi kelipatan 8 byte jika karakter teks tidak pas[cite: 7].
- **Fungsi `decrypt_des()`:** Mengubah data kembali ke aslinya dengan menggunakan fungsi `des_block()` yang sama, namun urutan *subkey* dibalik (mulai dari $K_{16} \rightarrow K_1$)[cite: 7].

### 2. `chat.py` (Jaringan & Antarmuka)
File ini bertugas mengurus koneksi TCP antar perangkat (Socket Programming) dan antarmuka *chat* di terminal.
- **Konfigurasi Key:** Kunci rahasia dideklarasikan secara konstan di dalam file (`SHARED_KEY = "ITS-2026"`) dan tidak pernah dikirim lewat jaringan untuk mencegah penyadapan[cite: 7]. Kunci harus berjumlah tepat 8 karakter (64-bit).
- **Socket Client-Server:** Mode 1 digunakan sebagai Server yang melakukan *listen* di *port* `5000` dan menunggu satu *client*[cite: 7]. Mode 2 digunakan sebagai Client yang terhubung ke IP server pada *port* `5000`[cite: 7].
- **Multi-threading:** Memanfaatkan fungsi `terima_pesan_masuk()` yang dijalankan di *thread* terpisah (berjalan di latar belakang) sehingga pengguna dapat menerima pesan dari lawan *chat* tanpa mengganggu kolom input ketikan secara *real-time*.
