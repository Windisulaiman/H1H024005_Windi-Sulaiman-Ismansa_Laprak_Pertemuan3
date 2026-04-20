# Pertanyaan Praktikum Percobaan 3B:  Inter-Integrated Circuit (I2C)

## 1. Jelaskan bagaimana cara kerja komunikasi I2C antara Arduino dan LCD pada rangkaian tersebut!

I2C menggunakan 2 jalur: **SDA** (data) dan **SCL** (clock). Arduino bertindak sebagai **master**, modul I2C LCD (PCF8574) sebagai **slave** dengan alamat `0x20`.

Setiap pengiriman data, Arduino memulai sinyal START di SDA, mengirim alamat slave 7-bit (`0x20`) + bit write, lalu mengirim data byte per byte yang di-clock oleh SCL. PCF8574 menerima data tersebut dan mengkonversinya dari serial (2 kabel) menjadi 8 pin paralel yang terhubung ke kontroler LCD HD44780. Dengan ini, Arduino cukup menggunakan 2 pin (A4/A5) untuk mengendalikan LCD 16 pin.

---

## 2. Apakah pin potensiometer harus seperti itu? Jelaskan yang terjadi apabila pin kiri dan pin kanan tertukar!

Potensiometer memiliki 3 kaki:
- **Kaki kiri** → VCC (5V)
- **Kaki tengah (wiper)** → A0 (input ADC)
- **Kaki kanan** → GND

**Jika kaki kiri dan kanan tertukar (VCC dan GND dibalik):**
Potensiometer tetap berfungsi, tetapi arah putarannya terbalik. Yang semula putar kiri = nilai ADC 0, menjadi putar kiri = nilai ADC 1023. Tidak merusak komponen, hanya membalik respons.

**Jika kaki tengah tidak terhubung ke A0:**
Pin A0 floating, nilai ADC acak dan tidak stabil.

---

## 3. Modifikasi program dengan menggabungkan antara UART dan I2C (keduanya sebagai output) sehingga:
- Data tidak hanya ditampilkan di LCD tetapi juga di Serial Monitor
- Adapun data yang ditampilkan pada Serial Monitor sesuai dengan table berikut:
  
  | ADC:0 | Volt:0,00 V | Persen:0% |
  
Tampilan jika potensiometer dalam kondisi diputar paling kiri
- ADC: 0 0% | setCursor(0, 0) dan Bar (level) | setCursor(0, 1)
- Berikan penjelasan disetiap baris kode nya

**Kode Program:**

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x20, 16, 2); // inisialisasi LCD I2C alamat 0x20, 16 kolom, 2 baris

const int pinPot = A0; // pin analog potensiometer

void setup() {
  Serial.begin(9600);       // inisialisasi komunikasi serial 9600 baud
  lcd.init();               // inisialisasi LCD
  lcd.backlight();          // nyalakan backlight LCD
}

void loop() {
  int nilai = analogRead(pinPot); // baca nilai ADC dari potensiometer (0-1023)

  float volt = (nilai / 1023.0) * 5.0;  // konversi ADC ke tegangan (0.00-5.00 V)
  int persen = (nilai / 1023.0) * 100;  // konversi ADC ke persentase (0-100%)
  int panjangBar = (nilai / 1023.0) * 16; // konversi ADC ke panjang bar (0-16)

  // Tampilkan ke Serial Monitor sesuai format tabel
  Serial.print("ADC: ");
  Serial.print(nilai);
  Serial.print(" | Volt: ");
  Serial.print(volt, 2);   // 2 angka desimal
  Serial.print(" V | Persen: ");
  Serial.print(persen);
  Serial.println("%");

  // Baris 1 LCD: nilai ADC dan persentase
  lcd.setCursor(0, 0);               // cursor ke kolom 0, baris 0
  lcd.print("ADC:");
  lcd.print(nilai);
  lcd.print(" ");
  lcd.print(persen);
  lcd.print("%  ");                  // spasi untuk clear sisa karakter

  // Baris 2 LCD: bar level
  lcd.setCursor(0, 1);               // cursor ke kolom 0, baris 1
  for (int i = 0; i < 16; i++) {
    if (i < panjangBar) {
      lcd.print('#');                // karakter bar aktif
    } else {
      lcd.print(' ');                // kosong jika tidak aktif
    }
  }

  delay(200); // jeda 200ms sebelum pembacaan berikutnya
}
```

**Format Serial Monitor:**
```
ADC: 0 | Volt: 0.00 V | Persen: 0%
ADC: 512 | Volt: 2.50 V | Persen: 50%
ADC: 1023 | Volt: 5.00 V | Persen: 100%
```

---

## 4. Lengkapi table berikut berdasarkan pengamatan pada Serial Monitor

| ADC | Volt (V) | Persen (%) |
|-----|----------|------------|
| 1   | 0.00 V   | 0%         |
| 21  | 0.10 V   | 2%         |
| 49  | 0.24 V   | 4%         |
| 74  | 0.36 V   | 7%         |
| 96  | 0.47 V   | 9%         |

Nilai dihitung dengan rumus:
- Volt = (ADC / 1023) × 5.0
- Persen = (ADC / 1023) × 100
