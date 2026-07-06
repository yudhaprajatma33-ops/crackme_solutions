# Easy Reverse Write-up

## Informasi Challenge

- *Kategori* : Reverse Engineering
- *Nama Challenge* : Easy Reverse
- *Platform* : Linux ELF 64-bit

---

## Deskripsi

Pada challenge ini diberikan sebuah binary Linux 64-bit. Tujuannya adalah menganalisis program untuk mengetahui bagaimana proses validasi input dilakukan dan mendapatkan flag.

---

## Tools yang Digunakan

- file
- strings
- Ghidra
- GDB (opsional)

---

## Analisis

### 1. Identifikasi File

Pertama cek tipe file.

bash
file rev50_linux64-bit


Output:

text
ELF 64-bit LSB executable, x86-64


---

### 2. Membuka Binary di Ghidra

Import file ke Ghidra kemudian lakukan proses *Auto Analysis*.

Setelah analisis selesai, buka fungsi main().

---

### 3. Analisis Fungsi main()

Seluruh proses pengecekan password dilakukan pada fungsi main().

Dari hasil dekompilasi dapat diketahui program melakukan tiga pengecekan:

### Syarat 1

Jumlah argumen harus *2*.

c
argc == 2


Artinya program harus dijalankan dengan sebuah parameter.

Contoh:

bash
./rev50_linux64-bit password


---

### Syarat 2

Panjang input harus *10 karakter*.

c
strlen(argv[1]) == 10


Jika kurang atau lebih dari 10 karakter maka input ditolak.

---

### Syarat 3

Karakter kelima harus berupa simbol @.

c
argv[1][4] == '@'


Perlu diingat bahwa indeks array dimulai dari *0*.

| Index | 0 | 1 | 2 | 3 | 4 |
|------|---|---|---|---|---|
| Karakter | A | A | A | A | @ |

---

## Kesimpulan

Program *tidak melakukan pengecekan isi password secara keseluruhan*.

Program hanya memastikan bahwa:

- terdapat satu argumen input
- panjang input tepat 10 karakter
- karakter kelima adalah @

Jika ketiga syarat tersebut terpenuhi, maka program langsung mencetak:

c
printf("flag{%s}", argv[1]);


Karena itu terdapat banyak password yang dianggap valid.

---

## Contoh Input Valid

text
AAAA@AAAAA


Menjalankan program:

bash
./rev50_linux64-bit 'AAAA@AAAAA'


Output:

text
flag{AAAA@AAAAA}


---

## Flag

text
flag{AAAA@AAAAA}


Karena validasi hanya memeriksa format input, maka semua string sepanjang *10 karakter* dengan karakter kelima berupa *@* akan menghasilkan output dengan format:

text
flag{input}


Contoh lain yang juga valid:

text
1234@56789


Output:

text
flag{1234@56789}


---

# Kesimpulan Akhir

Challenge ini bertujuan memperkenalkan dasar-dasar proses *reverse engineering* menggunakan decompiler. Dengan membaca kode hasil dekompilasi pada fungsi main(), dapat diketahui bahwa validasi password sangat sederhana sehingga tidak diperlukan proses brute force maupun analisis algoritma yang kompleks.
