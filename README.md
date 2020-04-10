# Shift 1 SISOP 2020 - T08
Penyelesaian Soal Shift 3 Sistem Operasi 2020\
Kelompok T08
  * I Made Dindra Setyadharma (05311840000008)
  * Muhammad Irsyad Ali (05311840000041)

---
## Table of Contents
* [Soal 3](#soal-2)
* [Soal 4](#soal-3)
  * [Soal 3.a.](#soal-3a)
  * [Soal 3.b.](#soal-3b)
  * [Soal 3.c.](#soal-3c)
---

## Soal 3
Source Code : [source](https://github.com/DSlite/SoalShiftSISOP20_modul3_T08/blob/master/soal3/soal3.c)

**Deskripsi:**\
Buatlah sebuah program dari C untuk mengkategorikan file. Program ini akan memindahkan file sesuai ekstensinya (tidak case sensitive. JPG dan jpg adalah sama) ke dalam folder sesuai ekstensinya yang folder hasilnya terdapat di working directory ketika program kategori tersebut dijalankan. Terdapa 3 arguman yang dapat di inputkan yaitu **(-f)**, **(*)** dan **(-d)**. Dengan ketentuan sebagai berikut:  

    * (-f) : 
                 *  user bisa menambahkan argumen file yang bisa dikategorikan sebanyak yang user inginkan  
                 *  Pada program kategori tersebut, folder jpg,c,zip tidak dibuat secara manual,
                    melainkan melalui program c. Semisal ada file yang tidak memiliki ekstensi,
                    maka dia akan disimpan dalam folder “Unknown”.  

    * (-d) :  
                 *  user hanya bisa menginputkan 1 directory saja.
                 *  Hasilnya perintah di atas adalah mengkategorikan file di /path/to/directory dan
                    hasilnya akan disimpan di working directory di mana program C tersebut
                    berjalan (hasil kategori filenya bukan di /path/to/directory).
                 *  Program ini tidak rekursif.
                 *  Setiap 1 file yang dikategorikan dioperasikan oleh 1 thread

    * (*) :  
                 *  mengkategorikan seluruh file yang ada di working directory

**Pembahasan:**\

``` bash
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <ctype.h>
#include <dirent.h>
#include <pthread.h>
#include <errno.h>
```
* `#include <sys/types.h>` Library tipe data khusus (e.g. pid_t)
* `#include <sys/stat.h>` LIbrary untuk
* `#include <stdio.h>` Library untuk fungsi input-output (e.g. printf(), sprintf())
* `#include <stdlib.h>` Library untuk fungsi umum (e.g. exit(), atoi())
* `#include <unistd.h>` Llibrary untuk melakukan system call kepada kernel linux (e.g. fork())
* `#include <string.h>` Library untuk 
* `#include <ctype.h>` Library untuk
* `#include <dirent.h>` Library untuk merepresentasikan directory stream & struct dirent(e.g. struct dirent *entry)
* `#include <pthread.h>` Library untuk operasi thread (e.g. pthread_create(), ptrhead_exit() )
* `#include <errno.h>` Library untuk error handling (e.g. errno)


Pertama kami melakukan pendefinisian 3 fungsi dan 1 routine(thread) untuk program ini yaitu: `getFileName`, `getExtension`, `dirChecking` dan `routine`.

**Fungsi *getFileName***
``` bash
char *getFileName(char *fName, char buff[]) {
  char *token = strtok(fName, "/");
  while (token != NULL) {
    sprintf(buff, "%s", token);
    token = strtok(NULL, "/");
  }
}
```
* Fungsi ini didefinisikan menggunakan dua parameter yaitu `*fname` sebagai pointernya dan `buff[]` untuk store hasil dari fungsi ini sendiri, dan akan mereturn file name yang masih beserta ekstensinya.
  * Selanjutnya nama dari file akan diambil menggunakan fungsi **strtok()** untuk memecah string dengan dengan delimiter `/` dan akan disimpan di dalam `*token` 
  * Lalu **while** loop akan berjalan selama token belum habis dan file name yang sudah diambil akan di print kedalam buffer.
  * Fungsi **strtok()** akan dijalankan lagi dengan parameter pertama = **NULL** untuk mencari token selanjutnya hingga akhir dari input.


**Fungsi *getExtension***
``` bash
char *getExtension(char *fName, char buff[]) {
  char buffFileName[1337];
  char *token = strtok(fName, "/");
  while (token != NULL) {
    sprintf(buffFileName, "%s", token);
    token = strtok(NULL, "/");
  }
```
* Fungsi ini didefinisikan menggunakan dua parameter yaitu `*fname` sebagai pointernya dan `buff[]` untuk store hasil dari fungsi ini sendiri, dan akan mereturn ekstensi dari sebuah file.
* Selanjutnya Fungsi akan melakukan hal yang sama persis seperti fungsi getFile name yang nantinya akan menghasilkan nama file yang masih beserta ekstensinya 

``` bash
 int count = 0;
  token = strtok(buffFileName, ".");
  while (token != NULL) {
    count++;
    sprintf(buff, "%s", token);
    token = strtok(NULL, ".");
  }
```
* Disini file name yang masih beserta ekstensinya akan dipecah kembali menggunakan fungsi **strtok()** dengan delimiter `.` sebagai pemisah antara nama file dengan ekstensi dan disimpan kedalam `token*`
* Karna yang akan pertama di return oleh fungsi **strtok()** adalah `token` pertma/kata pertama seblum delimiter maka **while** loop akan berjalan selama `token` belum habis atau belum sampai ekstensinya dan `counter` akan di increment sebagai variabel untuk checking
* Ekstensi yang sudah didapat akan di print ke dalam `buffer`.

``` bash
 if (count <= 1) {
    strcpy(buff, "unknown");
  }

  return buff;
```
* Pengecekan untuk jumlah `counter` yang kurang atau kondisi dimana file tidak ada ekstensi
* Untuk file yang tidak memiliki ekstensi, `buffer` akan berisi `unknown`

**Fungsi *directory Checking***
``` bash
 void dirChecking(char buff[]) {
  DIR *dr = opendir(buff);
  if (ENOENT == errno) {
    mkdir(buff, 0775);
    closedir(dr);
  }
}
```
* Fungsi didefinisikan menggunakan satu parameter yaitu `buff[]` untuk menyimpan hasil dari fungsi ini sendiri
* Disini pembuatan directory baru akan dilakukan jika ada sebuah error yang dihasilakan oleh fungsi **opendir()**, lalu **if** akan melakukan error handling.
* Fungsi ini melakukan pembuatan directory baru menggunakan fungsi **mkdir()** dengan nama yang di return `buffer` dan permission `0775` atau `read and execute` lalu akan di tutup kembali.

***Routine***
``` bash
void *routine(void* arg) {
  char buffExt[100];
  char buffFileName[1337];
  char buffFrom[1337];
  char buffTo[1337];
  char cwd[1337];
  getcwd(cwd, sizeof(cwd));
  strcpy(buffFrom, (char *) arg);
}
```
* Pada Routine ini kami mendefinisikan lima `buffer` yang masih masingnya akan menghandle: `ext` `fileName`
`path input` `path to` dan `cwd`
* Dimana `buffer` untuk `cwd` akan langsung diisi dengan fungsi **getcwd()** yang mereturn 
current working directory beserta sizenya dan untuk `from` atau argumen path yang diinpukan user 
akan diambil menggunakan **(char *)arg**

``` bash
  if (access(buffFrom, F_OK) == -1) {
    printf("File %s tidak ada\n", buffFrom);
    pthread_exit(0);
  }
  DIR* dir = opendir(buffFrom);
  if (dir) {
    printf("file %s berupa folder\n", buffFrom);
    pthread_exit(0);
  }
  closedir(dir);
```
note: dua ini kayanya buat -f 

* Disini program akan melakukan mengecek eksistensi dan bentuk dari file yang diinputkan oleh user
dari argumen yang diinputkan user menggunakan fungsi: 
    * **access()** dengan source `buffFrom` & `F_OK` sebagai `amode` untuk eksistensi. Jika argumen yang diinputkan
      tidak sesuai makan ditampilkan error message dan `thread` akan diselesaikan menggunakan **pthread_exit(0)**
    * Pengecekan bentuk file yang diinputkan adalah dengan menggunakan **dir** yang dimana jika terbuka sebagai 
      directory maka akan ditampilkan error message dan `thread` akan diselesaikan menggunakan **pthread_exit(0)**

``` bash
  getFileName(buffFrom, buffFileName);
  strcpy(buffFrom, (char *) arg);

  getExtension(buffFrom, buffExt);
  for (int i = 0; i < sizeof(buffExt); i++) {
    buffExt[i] = tolower(buffExt[i]);
  }
  strcpy(buffFrom, (char *) arg);
```
* Selanjutnya kami memanggil fungsi **getFilename()** dengan filename yang akan masuk ke `buffer` 
baru `buffFilename`
* Lalu kami memanggil fungsi **getExtension()** untuk mengambil setiap ext dari filename yang ada di `buffFrom`
dan merubah tiap extension dari file yang ada menjadi `lowercase` menggunakan **for** loop degam fungsi **tolower()** dan `i` sebagai `counter`nya.

``` bash
  dirChecking(buffExt);

  sprintf(buffTo, "%s/%s/%s", cwd, buffExt, buffFileName);
  rename(buffFrom, buffTo);

  pthread_exit(0);
```
* Selanjutnya fungsi **dirChecking()** akan dipanggil yang akan membuat directory baru untuk setiap ekstensi didalam `buffExt` yang belum memiliki directory
* Lalu `buffTo` akan diisi dengan value setiap `buffer` yang sudah di set sebelumnya dengan urutan `cwd`,`buffExt`
dan `buffFilename` menggunakan **sprintf()**. Kemudian file name yang ada di `buffFrom` `(const char *old_filename)`
akan di **rename()** dengan urutan dari `buffTo` `(const char *new_filename)` yang sudah di set.

``` bash
int main(int argc, char *argv[]) {
  if (argc == 1) {
    printf("Argument kurang\n");
    exit(1);
  }
  if (strcmp(argv[1], "-f") != 0 && strcmp(argv[1], "*") != 0 && strcmp(argv[1], "-d")) {
    printf("Argument tidak ada\n");
    exit(1);
  }

  if (strcmp(argv[1], "-f") == 0) {
    if (argc <= 2) {
      printf("Argument salah\n");
      exit(1);
    }
```
* 
























### Soal 1.b.
**Deskripsi:**\
Tampilkan 2 negara bagian (state) yang memiliki keuntungan (profit) paling sedikit berdasarkan hasil poin a

**Pembahasan:**\
Untuk menentukan keuntungan paling sedikit, dapat menggunakan `awk` seperti pada poin a, namun dengan sedikit perbedaan.

``` bash
read -r state1 state1profit state2 state2profit <<< `awk -F "\t" -v region=$region '{if (match($13, region)) seen[$11]+=$NF} END {for (i in seen) printf "%s?%f\n", i, seen[i]}' $PWD/Sample-Superstore.tsv | sort -g -t? -k2 | awk -F? 'NR < 3 {printf "%s %f ", $1, $2}'`
printf "2 State dengan profit paling sedikit dari region $region:\n$state1($state1profit)\n$state2($state2profit)\n\n"
```

* Pada bagian `awk -F "\t" -v region=$region '{if (match($13, region)) seen[$11]+=$NF} END {for (i in seen) printf "%s?%f\n", i, seen[i]}' $PWD/Sample-Superstore.tsv`, akan menjalankan perintah awk dengan "**tab**" sebagai field separatornya. Selain itu `-v` digunakan untuk meng-set variable ***region*** (region dengan profit terkecil) kedalam awk.
  * Dalam block **BODY**: akan mengecek apakah kolom ke-`$13`(**regionnya**) sama dengan region yang didapat pada poin a. Jika sama, maka **profit**(`$NF`) dari **state** tersebut akan dijumlahkan ke dalam array `seen` dengan menggunakan **state**(`$11`) sebagai index dari array tersebut.
  * Dalam block **END**: akan melakukan loop untuk setiap index dari array `seen`, lalu setiap index dan nilainya akan diprint menggunakan `printf "%s?$f\n", i, seen[i]`. Format yang dihasilkan berupa "**state**?**profit**\n".
* Lalu dari `awk` tersebut akan di *pipe* ke dalam command `sort -g -t? -k2`. Kegunaannya sama dengan poin a, yaitu untuk mensortir berdasarkan kolom kedua(**profit**).
* Lalu akan di *pipe* lagi ke dalam `awk -F? 'NR < 3 {printf "%s %f ", $1, $2}'`. Kegunaannya sama dengan poin a, tetapi disini akan mengambil 2 nilai terkecil dari hasil sortir sebelumnya. Lalu diprint dengan format "**state** **profit** ".
* Setelah mendapat 2 state dengan region terkecil, hasil tersebut akan disimpan kedalam variable `$state1`, `$state1profit`, `$state2`, `$state2profit`. Cara memasukkannya sama dengan poin a.
* Lalu 2 **state** dan **profit**nya akan diprint menggunakan `printf "2 State dengan profit paling sedikit dari region $region:\n$state1($state1profit)\n$state2($state2profit)\n\n"`

### Soal 1.c.
**Deskripsi:**\
Tampilkan 10 produk (product name) yang memiliki keuntungan (profit) paling sedikit berdasarkan 2 negara bagian (state) hasil poin b

**Pembahasan:**\
Untuk poin c, bisa menggunakan `awk` yang mirip dengan poin b. Untuk variablenya, ditambahkan variable `$state1` dan `$state2` kedalam `awk`.

``` bash
list=`awk -F "\t" -v state1=$state1 -v state2=$state2 '{if (match ($11, state1)||match ($11, state2)) seen[$17]+=$NF} END {for (i in seen) printf "%s?%f\n", i, seen[i]}' $PWD/Sample-Superstore.tsv | sort -g -t? -k2 | awk -F? 'NR < 11 {printf "%s(%f)\n", $1, $2}'`
printf "List barang dengan profit paling sedikit antara state $state1 dan $state2:\n$list\n\n"
```

* Pada `awk` ditambahkan `-v` untuk masing-masing state, lalu akan dicari baris yang **state**nya sama dengan **state** dengan profit terkecil. Untuk konsep pada block **BODY** dan **END** sama seperti poin sebelumnya.
* Lalu kegunaan fungsi `sort` sama seperti poin sebelumnya
* Hasil `sort` akan di *pipe* ke dalam `awk` untuk mencari 10 **product** dengan **profit** terkecil. Format yang dihasilkan akan menjadi "**product**(**profit**)\n"
* Hasil command tersebut akan dimasukkan ke dalam variable `$list`. Disini tidak menggunakan `read -r` agar "**\n**" masuk ke dalam variable tersebut.
* Lalu 10 **product** dan **profit**nya akan diprint menggunakan `printf "List barang dengan profit paling sedikit antara state $state1 dan $state2:\n$list\n\n"`

#### ScreenShot
**Contoh Output:**\
![Output Soal 1](https://user-images.githubusercontent.com/17781660/74916187-ff17d180-53f7-11ea-814e-693ffe29028e.png)

---
