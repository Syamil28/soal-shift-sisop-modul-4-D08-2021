
# soal-shift-sisop-modul-4-D08-2021

## **Soal 1**
### Penjelasan soal
Di suatu jurusan, terdapat admin lab baru yang super duper gabut, ia bernama Sin. Sin baru menjadi admin di lab tersebut selama 1 bulan. Selama sebulan tersebut ia bertemu orang-orang hebat di lab tersebut, salah satunya yaitu Sei. Sei dan Sin akhirnya berteman baik. Karena belakangan ini sedang ramai tentang kasus keamanan data, mereka berniat membuat filesystem dengan metode encode yang mutakhir. Berikut adalah filesystem rancangan Sin dan Sei :
```
NOTE : 
Semua file yang berada pada direktori harus ter-encode menggunakan Atbash cipher(mirror).
Misalkan terdapat file bernama kucinglucu123.jpg pada direktori DATA_PENTING
“AtoZ_folder/DATA_PENTING/kucinglucu123.jpg” → “AtoZ_folder/WZGZ_KVMGRMT/pfxrmtofxf123.jpg”
Note : filesystem berfungsi normal layaknya linux pada umumnya, Mount source (root) filesystem adalah directory /home/[USER]/Downloads, dalam penamaan file ‘/’ diabaikan, dan ekstensi tidak perlu di-encode.
```
a. Jika sebuah direktori dibuat dengan awalan “AtoZ_”, maka direktori tersebut akan menjadi direktori ter-encode.

Men-scan kata "AtoZ" pada nama direktori:
```c
    char *enc1 = strstr(path, ATOZ);
```
Jika terdapat awalan "AtoZ" memanggil fungsi `encode_atbash()`
```
        if (enc1 != NULL) {
            decryptvigenere(de->d_name);
            printf("== readdir::encode:%s\n", de->d_name);
        }
```
b. Jika sebuah direktori di-rename dengan awalan “AtoZ_”, maka direktori tersebut akan menjadi direktori ter-encode.

Sama seperti poin a, jika terdapat awalan "AtoZ" akan memenggil fungsi `encode_atbash()`.

c. Apabila direktori yang terenkripsi di-rename menjadi tidak ter-encode, maka isi direktori tersebut akan terdecode.

Agar menjadi tidak ter-encode, berarti awalan "AtoZ" dari penamaan direktori dihapus. Jika pada awalan direktori tidak ada "AtoZ", akan memanggil fungsi `decryptvigenere()`.
```
void decode_atbash(char *str) {
    if (!strcmp(str, ".") || !strcmp(str, "..")) return;
    if (strstr(str, "/") == NULL) return;
    printf("==== before:%s\n", str);
    
    int str_length = strlen(str);
    int start=0;
    int i;
    for (i = str_length; i >= 0; i--) {
        if (str[i] == '/') break;

        if (str[i] == '.') {
            str_length = i;
            break;
        }
    }
    for (i = 0; i < str_length; i++) {
        if (str[i] == '/') {
            start = i + 1;
            break;
        }
    }

    atbash(str, start, str_length);
    printf("==== dec:atb:%s\n", str);
}
```
Pertama melihat apakah parent direktori, jika iya maka tidak berlu didecode. Lalu jika merupakan root direktori juga tidak perlu didecode. Setelahnya mencari panjang string nama file atau folder, dan menentukan titik awal penamaan file atau folder. Lalu memanggil fungsi `atbash()` untuk didecode.

d. Setiap pembuatan direktori ter-encode (mkdir atau rename) akan tercatat ke sebuah log. Format : **/home/[USER]/Downloads/[Nama Direktori] → /home/[USER]/Downloads/AtoZ_[Nama Direktori]**

Pada fuse_operation `rename`, dipanggil fungsi `log1()`. Fungsi `log1()` mencatat perubahan nama dan menyimpannya ke sebuah log.
```
void log1(char *from, char *to) {
    int i;
    for (i = strlen(to); i >= 0; i--) {
        if (to[i] == '/') 
          break;
    }
    if (strstr(to + i, ATOZ) == NULL) 
      return;

    FILE *log_file = fopen(logpath, "a");
    fprintf(log_file, "%s -> %s\n", from, to);
}

```
e. Metode encode pada suatu direktori juga berlaku terhadap direktori yang ada di dalamnya.(rekursif)

Metode encode melihat tanda `/` untuk bisa memisahkan antar direktorinya. Jika direktori didalamnya ter-encode, berarti telah terename, dan pada fuse_operation `rename` akan dipanggil lagi fungsi `encode_atbash()` sehingga direktori yang ada di dalamnya ikut ter-encode.

### Screenshot
- Membuat direktori baru dan menjalankan FUSE:
![pict](https://i.postimg.cc/rmSYnY18/Virtual-Box-Ubuntu-New-12-06-2021-22-11-16.png)

- Muncul FUSE sesuai nama direktorinya, dengan isi dari folder Downloads:
![pict](https://i.postimg.cc/7ZZhqFhW/Virtual-Box-Ubuntu-New-13-06-2021-01-06-31.png)

- Isi dari direktori "inifolder" (tanpa awalan AtoZ):
![pict](https://i.postimg.cc/TY7GJp2t/Virtual-Box-Ubuntu-New-12-06-2021-22-18-50.png)

- Setelah folder direname dengan awalan AtoZ:
![pict](https://i.postimg.cc/9X1jcr9c/Virtual-Box-Ubuntu-New-12-06-2021-22-19-32.png)

## **Soal 2**
### Penjelasan Soal
Selain itu Sei mengusulkan untuk membuat metode enkripsi tambahan agar data pada komputer mereka semakin aman. Berikut rancangan metode enkripsi tambahan yang dirancang oleh Sei

 a. Jika sebuah direktori dibuat dengan awalan “RX_[Nama]”, maka direktori tersebut akan menjadi direktori terencode beserta isinya dengan perubahan nama isi sesuai kasus nomor 1 dengan algoritma tambahan ROT13 (Atbash + ROT13).

```
    char *enc2 = strstr(path, RX);
```
Jika terdapat awalan "RX_" program akn memanggil fungsi `encode_atbash()`
```
    else if (enc2 != NULL) {
        printf("YEY::getattr:%s\n", path);
        decryptvigenere(enc2);
    }
```

 b. Jika sebuah direktori di-rename dengan awalan “RX_[Nama]”, maka direktori tersebut akan menjadi direktori terencode beserta isinya dengan perubahan nama isi sesuai dengan kasus nomor 1 dengan algoritma tambahan Vigenere Cipher dengan key “SISOP” (Case-sensitive, Atbash + Vigenere).
sama seperti pada poin a jika terdapat folder dengan awalan `RX_` maka program akan menjalanka fungsi `encryptyvigenere` dan dilakukan encode.

c. Apabila direktori yang terencode di-rename (Dihilangkan “RX_” nya), maka folder menjadi tidak terencode dan isi direktori tersebut akan terdecode berdasar nama aslinya.
Agar menjadi tidak ter-encode, berarti awalan `RX_` dari penamaan direktori harus dihapus atau di re-name. Jika pada awalan direktori tidak ada `RX_`, maka program akan akan memanggil fungsi `decryptvigenere()`.
```
void decryptvigenere(char *str) {
    if (!strcmp(str, ".") || !strcmp(str, "..")) return;
    if (strstr(str, "/") == NULL) return;
    printf("==== before:%s\n", str);
    
    int str_length = strlen(str);
    int start=0;
    int i;
    for (i = str_length; i >= 0; i--) {
        if (str[i] == '/') break;

        if (str[i] == '.') {
            str_length = i;
            break;
        }
    }
    for (i = 0; i < str_length; i++) {
        if (str[i] == '/') {
            start = i + 1;
            break;
        }
    }

    atbash(str, start, str_length);
    printf("==== dec:atb:%s\n", str);
}
```

d. etiap pembuatan direktori terencode (mkdir atau rename) akan tercatat ke sebuah log file beserta methodnya (apakah itu mkdir atau rename).
Pada fuse_operation `rename`, dipanggil fungsi `log1()`. Fungsi `log1()` mencatat perubahan nama dan menyimpannya ke sebuah log.
```
void log1(char *from, char *to) {
    int i;
    for (i = strlen(to); i >= 0; i--) {
        if (to[i] == '/') 
          break;
    }
    if (strstr(to + i, ATOZ) == NULL) 
      return;

    FILE *log_file = fopen(logpath, "a");
    fprintf(log_file, "%s -> %s\n", from, to);
}

```

e. Pada metode enkripsi ini, file-file pada direktori asli akan menjadi terpecah menjadi file-file kecil sebesar 1024 bytes, sementara jika diakses melalui filesystem rancangan Sin dan Sei akan menjadi normal. Sebagai contoh, Suatu_File.txt berukuran 3 kiloBytes pada directory asli akan menjadi 3 file kecil yakni:

Suatu_File.txt.0000
Suatu_File.txt.0001
Suatu_File.txt.0002

Ketika diakses melalui filesystem hanya akan muncul Suatu_File.txt


### Screenshot
- Membuat direktori baru dan menjalankan FUSE:
![Screenshot (125)](https://user-images.githubusercontent.com/62102884/121807576-ad035b00-cc8f-11eb-9ed7-aea1a2b32896.png)

- Muncul FUSE sesuai nama direktorinya, dengan isi dari folder Downloads:
![Screenshot (126)](https://user-images.githubusercontent.com/62102884/121807569-a379f300-cc8f-11eb-9b28-2e85aeaabb2c.png)

- Isi dari direktori "contoh" (tanpa awalan `RX_`):
![Screenshot (127)](https://user-images.githubusercontent.com/62102884/121807619-e5a33480-cc8f-11eb-9c85-0eb0aefc7906.png)

- Setelah folder direname dengan awalan `RX_`:
![Screenshot (128)](https://user-images.githubusercontent.com/62102884/121807700-361a9200-cc90-11eb-8f53-77f62bc7af6d.png)

- isi dari file *SinSeiFS.log* ketika telah dilakukan re-name, make directory, dan remove folder/file:
![Screenshot (129)](https://user-images.githubusercontent.com/62102884/121807803-a7f2db80-cc90-11eb-87e3-8978b06ce351.png)

### Kendala
1. masih belum bisa menemukan cara untuk mengerjakan point e
2. sempat bingung dengan cara agar bisa melakukan encode
3. sempat bingung dengan cara agar bisa mendapatkan real path dari direktori pada folder mount
