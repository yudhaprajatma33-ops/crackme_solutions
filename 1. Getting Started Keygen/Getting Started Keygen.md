# Mazzotti's Getting Started Keygen

## Challenge Information

| Item | Value |
|------|-------|
| Challenge | Getting Started Keygen |
| Author | Mazzotti |
| Language | C/C++ |
| Platform | Linux |
| Architecture | x86-64 |
| Difficulty | 1.0 |
| Binary Type | ELF 64-bit PIE |

---

# Tujuan

Melakukan reverse engineering terhadap binary menggunakan **Ghidra** untuk mengetahui bagaimana program menghasilkan key yang benar.

---

# Tools

- Ghidra
- file
- strings (opsional)
- Linux Terminal

---

# Recon

Pertama, identifikasi tipe binary.

```bash
file getting_started_keygen
```

Output:
A
```
getting_started_keygen:
ELF 64-bit LSB pie executable,
x86-64,
dynamically linked,
stripped
```

Dari hasil tersebut diketahui bahwa binary merupakan executable Linux 64-bit dan telah di-strip sehingga nama fungsi asli tidak tersedia.

**Screenshot**

- Output `file`

---

# Menjalankan Program

Jalankan binary.

```bash
./getting_started_keygen
```

Program meminta dua input.

```
Enter a string of characters (no spaces):
Enter correct number (no spaces):
```

Belum diketahui hubungan kedua input tersebut sehingga dilakukan analisis statis menggunakan Ghidra.

**Screenshot**

- Program saat pertama dijalankan.

---

# Analisis Menggunakan Ghidra

## Import Binary

1. Buka Ghidra.
2. Buat Project baru.
3. Import file `getting_started_keygen`.
4. Jalankan Auto Analyze.

**Screenshot**

- Import binary
- Auto Analyze

---

# Analisis Fungsi Main

Karena binary sudah di-strip, nama fungsi menjadi `FUN_xxxxxxxx`.

Dari hasil decompile pada fungsi utama diperoleh alur program sebagai berikut.

1. Membaca input string.
2. Membaca input angka.
3. Memanggil fungsi `FUN_001014b0()`.
4. Membandingkan hasil fungsi dengan angka yang dimasukkan pengguna.
5. Jika sama maka program menampilkan

```
OMG! You did it! :3
```

Diagram alur program

```
Input String
      │
      ▼
FUN_001014b0()
      │
      ▼
Menghasilkan Angka
      │
      ▼
Bandingkan dengan Input Kedua
      │
      ▼
 Sama ?
 /    \
Ya    Tidak
 │       │
 ▼       ▼
Success  Failed
```

**Screenshot**

- Fungsi main
- Pemanggilan `FUN_001014b0`

---

# Analisis Fungsi FUN_001014b0

Decompiler menghasilkan kode berikut.

```c
int FUN_001014b0(long *param_1)

{
  char *pcVar1;
  uint *puVar2;
  long lVar3;
  int iVar4;

  if (param_1[1] != 0) {
    lVar3 = 0;
    iVar4 = 0;
    do {
      pcVar1 = (char *)(*param_1 + lVar3);
      puVar2 = &DAT_00104020 + lVar3;
      lVar3 = lVar3 + 1;
      iVar4 = iVar4 + ((int)*pcVar1 ^ *puVar2);
    } while (param_1[1] != lVar3);
    return iVar4;
  }
  return 0;
}
```

---

# Analisis Kode

## Inisialisasi

```c
lVar3 = 0;
iVar4 = 0;
```

`lVar3` digunakan sebagai indeks karakter.

`iVar4` digunakan untuk menyimpan hasil akhir.

---

## Mengambil karakter

```c
pcVar1 = (char *)(*param_1 + lVar3);
```

Program mengambil karakter ke-`i` dari string.

Misalnya

```
AAAAA
```

Iterasi pertama mengambil

```
'A'
```

---

## Mengambil Lookup Table

```c
puVar2 = &DAT_00104020 + lVar3;
```

Program mengambil nilai dari tabel pada alamat

```
DAT_00104020
```

Isi tabel adalah

```
[4,79,129,171,254,123,224,204,70,53]
```

**Screenshot**

- Lookup table pada `DAT_00104020`

---

## Operasi XOR

Baris terpenting adalah

```c
iVar4 = iVar4 + ((int)*pcVar1 ^ *puVar2);
```

Artinya

```
hasil += karakter XOR tabel
```

Misalnya karakter pertama

```
'A'
```

ASCII

```
65
```

Lookup table

```
4
```

Perhitungan

```
65 XOR 4 = 69
```

Nilai tersebut ditambahkan ke variabel total.

Loop akan terus berjalan sampai seluruh karakter selesai diproses.

---

# Pseudocode

```c
int keygen(char input[])
{
    int table[10] =
    {
        4,
        79,
        129,
        171,
        254,
        123,
        224,
        204,
        70,
        53
    };

    int total = 0;

    for(int i=0;i<strlen(input);i++)
    {
        total += input[i] ^ table[i];
    }

    return total;
}
```

---

# Perhitungan Manual

Gunakan string

```
AAAAA
```

ASCII

```
A = 65
```

Perhitungan

| Karakter | XOR | Hasil |
|----------|-----|-------|
|65 ^ 4|69|
|65 ^ 79|14|
|65 ^ 129|192|
|65 ^ 171|234|
|65 ^ 254|191|

Jumlah

```
69 + 14 + 192 + 234 + 191

=

700
```

---

# Verifikasi

Jalankan program.

```bash
./getting_started_keygen
```

Masukkan

```
AAAAA
```

Kemudian

```
700
```

Output

```
OMG! You did it! :3
```

Berarti algoritma yang diperoleh sudah benar.

**Screenshot**

- Program berhasil dijalankan

---

# Kesimpulan

Challenge ini menggunakan algoritma keygen sederhana.

Setiap karakter input di-XOR dengan nilai pada lookup table kemudian seluruh hasil dijumlahkan menjadi sebuah integer.

Algoritma dapat dituliskan sebagai

```
key = Σ(input[i] XOR table[i])
```

Selama angka yang dimasukkan pengguna sama dengan hasil perhitungan tersebut, program akan menerima input dan menampilkan pesan keberhasilan.

---


