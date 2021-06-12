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
            encode_atbash(de->d_name);
            printf("== readdir::encode:%s\n", de->d_name);
        }
```
b. Jika sebuah direktori di-rename dengan awalan “AtoZ_”, maka direktori tersebut akan menjadi direktori ter-encode.

Sama seperti poin a, jika terdapat awalan "AtoZ" akan memenggil fungsi `encode_atbash()`.

c. Apabila direktori yang terenkripsi di-rename menjadi tidak ter-encode, maka isi direktori tersebut akan terdecode.

Agar menjadi tidak ter-encode, berarti awalan "AtoZ" dari penamaan direktori dihapus. Jika pada awalan direktori tidak ada "AtoZ", akan memanggil fungsi `decode_atbash()`.
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
