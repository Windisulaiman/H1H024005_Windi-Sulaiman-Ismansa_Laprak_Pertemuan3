# Pertanyaan Praktikum Percobaan 3A: Komunikasi Serial (UART)

## 1. Jelaskan proses dari input keyboard hingga LED menyala/mati!

Ketika pengguna mengetik karakter di Serial Monitor dan menekan Enter, karakter tersebut dikirim melalui USB ke buffer UART Arduino. `Serial.available()` mendeteksi bahwa buffer tidak kosong, lalu `Serial.read()` mengambil 1 karakter dari buffer dan menyimpannya ke variabel `data`. Program mengevaluasi nilai `data`: jika `'1'`, fungsi `digitalWrite(PIN_LED, HIGH)` mengalirkan arus ke pin 12 sehingga LED menyala; jika `'0'`, `digitalWrite(PIN_LED, LOW)` memutus arus sehingga LED mati.

---

## 2. Mengapa digunakan Serial.available() sebelum membaca data? Apa yang terjadi jika baris tersebut dihilangkan? 

karena `Serial.available()` mengembalikan jumlah byte yang tersedia di buffer penerima UART. Pembacaan dilakukan hanya jika nilainya lebih dari 0, artinya ada data yang benar-benar masuk.

**Jika baris tersebut dihilangkan**, `Serial.read()` tetap dipanggil setiap iterasi `loop()` meskipun tidak ada data. Saat buffer kosong, `Serial.read()` mengembalikan nilai `-1` (dicast ke `char` menjadi karakter tidak valid). Program akan terus mengevaluasi karakter sampah tersebut, menyebabkan kondisi `else if (data != '\n' && data != '\r')` terus terpenuhi dan Serial Monitor dibanjiri output `"Perintah tidak dikenal"` secara terus-menerus.

---

## 3. Modifikasi program agar LED berkedip (blink) ketika menerima input '2' dengan kondisi jika ‘2’ aktif maka LED akan terus berkedip sampai perintah selanjutnya diberikan dan berikan penjelasan disetiap baris kode nya!

### Kode Program

```cpp
const int PIN_LED = 12;

bool modeBlink = false;           // status apakah mode blink aktif
unsigned long waktuSebelumnya = 0; // menyimpan waktu terakhir LED berubah state
const long intervalBlink = 500;   // interval kedip dalam milidetik
bool statusLED = false;           // menyimpan state LED saat ini (nyala/mati)

void setup() {
  Serial.begin(9600);
  Serial.println("Ketik '1' nyala, '0' mati, '2' blink");
  pinMode(PIN_LED, OUTPUT); // set pin 12 sebagai output
}

void loop() {
  // Cek apakah ada input baru dari Serial
  if (Serial.available() > 0) {
    char data = Serial.read(); // ambil 1 karakter dari buffer

    if (data == '1') {
      modeBlink = false;              // matikan mode blink
      digitalWrite(PIN_LED, HIGH);   // nyalakan LED
      Serial.println("LED ON");
    }
    else if (data == '0') {
      modeBlink = false;             // matikan mode blink
      digitalWrite(PIN_LED, LOW);   // matikan LED
      Serial.println("LED OFF");
    }
    else if (data == '2') {
      modeBlink = true;              // aktifkan mode blink
      Serial.println("LED BLINK");
    }
    else if (data != '\n' && data != '\r') {
      // filter karakter newline, tampilkan error jika bukan 1, 0, atau 2
      Serial.println("Perintah tidak dikenal");
    }
  }

  // Eksekusi blink jika mode aktif, tanpa memblokir loop
  if (modeBlink) {
    unsigned long waktuSekarang = millis(); // baca waktu saat ini

    // cek apakah sudah melewati interval
    if (waktuSekarang - waktuSebelumnya >= intervalBlink) {
      waktuSebelumnya = waktuSekarang;  // perbarui waktu referensi
      statusLED = !statusLED;           // balik state LED
      digitalWrite(PIN_LED, statusLED ? HIGH : LOW); // terapkan state
    }
  }
}
```

---

## 4. Tentukan apakah menggunakan delay() atau milis()! Jelaskan pengaruhnya terhadap sistem!

**yang digunakan adalah `millis()`**

Penjelasan:
`delay()` memblokir seluruh eksekusi program. Selama jeda itu, `Serial.available()` tidak dicek, sehingga input `'0'` atau `'1'` dari pengguna tidak akan diproses sampai delay selesai. Sistem menjadi tidak responsif.

`millis()` tidak memblokir. Program terus menjalankan `loop()` secara penuh setiap siklus. Perubahan state LED hanya terjadi ketika selisih waktu antara `millis()` sekarang dan `waktuSebelumnya` sudah melewati `intervalBlink`. Akibatnya, Serial tetap terbaca di setiap iterasi dan perintah baru langsung diproses tanpa menunggu kedip selesai.
