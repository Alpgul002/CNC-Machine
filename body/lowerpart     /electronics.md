Elektronik için linuxcnc ve mesa kartı ile step motorlar düşünürken, mesa kartının pahalı olması ve tr de olmaması nedeniyle projenin elektroniğini değiştirme kararı aldım.



**CNC Sistemi Mühendislik El Kitabı**

**Proje Başlığı**: Raspberry Pi + LinuxCNC (pigpio HAL) Tabanlı, Yerel Ekran ve Uzaktan G-code Erişimli IoT Destekli Akıllı CNC Sistemi
**Hazırlayan**: Deniz
**Tarih**: Temmuz 2025

---

## 1. Donanım Kısmı

### 1.1. Parça Listesi ve Adetleri

| Bileşen                                 | Model / Tip               | Adet | Açıklama                                            |
| --------------------------------------- | ------------------------- | ---- | --------------------------------------------------- |
| Raspberry Pi 4 Model B (4GB)            | SBC                       | 1    | PREEMPT\_RT kernel, pigpio HAL, LinuxCNC barındırır |
| microSD Kart (32GB veya üzeri)          | UHS-I, Class 10           | 1    | OS ve yazılım imajları için                         |
| TB6600 Step Motor Sürücü                | Opto-izolatörlü           | 3    | X, Y, Z eksenleri için                              |
| NEMA 23 Step Motor                      | 3.0 Nm, 4 kablolu bipolar | 3    | X, Y, Z eksen hareketi                              |
| Limit Switch (NC tipi)                  | Mekanik NC switch         | 6    | Her eksenin +/– uçlarında güvenlik                  |
| Endüstriyel Güç Kaynağı (PSU)           | 24 V, 10 A                | 1    | TB6600 sürücüler için güç                           |
| 5 V DC Regülatör / USB adaptör          | 5 V, 2 A                  | 1    | Pi’yi beslemek için                                 |
| Shielded Twisted-Pair Kablo             | 26 AWG                    | 8 m  | STEP/DIR/ENA sinyalleri                             |
| Çift Damarlı Kablo                      | 24 AWG                    | 5 m  | Limit switch hatları                                |
| Tek Damarlı Kablo                       | ≥1.5 mm²                  | 2 m  | 24 V PSU güç hattı                                  |
| Vida Tipi Terminal Bloğu (8 kutup)      | DIN ray montajlı          | 2    | Kablolama düzeni                                    |
| M2.5-M3 Klemens                         | 5 mm adım aralıklı        | 20   | Elektrik bağlantı noktaları                         |
| Perf-board (Delikli devre)              | UV-fırınlanmış            | 1    | Harici pull-up devresi                              |
| Direnç (10 kΩ, SIP8 paketi)             | Pull-up array             | 1    | Harici +5 V pull-up için                            |
| Somun, Pul ve Kablo Bağlama Malzemeleri | —                         | —    | Montaj ve kablo düzeni                              |

### 1.2. GPIO Pin Bağlantıları (BCM Numarasını Kullanarak)

| Fonksiyon          | Eksen | BCM GPIO | Açıklama                                      |
| ------------------ | ----- | -------- | --------------------------------------------- |
| STEP\_X            | X     | 17       | X ekseni step sinyali                         |
| DIR\_X             | X     | 18       | X ekseni yön sinyali                          |
| ENA\_X             | X     | 27       | X ekseni enable                               |
| STEP\_Y            | Y     | 22       | Y ekseni step                                 |
| DIR\_Y             | Y     | 23       | Y ekseni yön                                  |
| ENA\_Y             | Y     | 24       | Y ekseni enable                               |
| STEP\_Z            | Z     | 5        | Z ekseni step                                 |
| DIR\_Z             | Z     | 6        | Z ekseni yön                                  |
| ENA\_Z             | Z     | 13       | Z ekseni enable                               |
| LIMIT\_X+          | X     | 19       | X ekseni pozitif limit (NC, internal pull-up) |
| LIMIT\_X-          | X     | 26       | X ekseni negatif limit                        |
| LIMIT\_Y+          | Y     | 16       | Y ekseni pozitif limit                        |
| LIMIT\_Y-          | Y     | 20       | Y ekseni negatif limit                        |
| LIMIT\_Z+          | Z     | 21       | Z ekseni pozitif limit                        |
| LIMIT\_Z-          | Z     | 12       | Z ekseni negatif limit                        |
| E-STOP (Emergency) | —     | 4        | Acil durdurma giriş ucu (harici buton)        |

### 1.3. Fiziksel ve Elektriksel Gereksinimler

* **Topraklama**: PSU GND, Pi GND ve tüm switch/girişlerin ortak topraklama noktasında (DIN ray und terminal bloğunda) birleştirilmesi zorunludur.
* **Güç Dağılımı**: 24 V PSU’nun +24 V çıkışı TB6600 VCC+ uçlarına, GND çıkışı TB6600 VCC- uçları ve Pi GND’ye dağıtılmalıdır.
* **Seviye Uyumu**: TB6600 opto-izolatör girişleri 3.3 V TTL sinyalini algılayabilir. Gerekirse 74HCT14 seviye çevirici veya NPN transistörlü buffer kullanılabilir.
* **EMI Koruması**: STEP/DIR/ENA hatları için shielded twisted-pair, limit switch hatları için twisted-pair önerilir. Sinyal ve güç hatları ayrı kılavuz kanallarında götürülmeli.
* **Çalışma Ortamı**: ±10–50 °C aralığında, nem <%70, tozdan ari kapalı pano.
* **Montaj**: Terminal blok ve perf-board panoya sabitlenmeli, titreşim ve darbelere karşı kablo bağlama klipsleri kullanılmalı.

---

### 1.4 Tüm Kablo Bağlantıları
| Kaynak Cihaz             | Hedef Cihaz          | Sinyal Tipi                                                   | GPIO Pin (BCM)         | Kablo Tipi            | Kablo Kesiti |
| ------------------------ | -------------------- | ------------------------------------------------------------- | ---------------------- | --------------------- | ------------ |
| Raspberry Pi GPIO17      | TB6600 X STEP        | STEP\_X                                                       | 17                     | Shielded Twisted-Pair | 26 AWG       |
| Raspberry Pi GPIO18      | TB6600 X DIR         | DIR\_X                                                        | 18                     | Shielded Twisted-Pair | 26 AWG       |
| Raspberry Pi GPIO27      | TB6600 X ENA         | ENA\_X                                                        | 27                     | Shielded Twisted-Pair | 26 AWG       |
| Raspberry Pi GPIO22      | TB6600 Y STEP        | STEP\_Y                                                       | 22                     | Shielded Twisted-Pair | 26 AWG       |
| Raspberry Pi GPIO23      | TB6600 Y DIR         | DIR\_Y                                                        | 23                     | Shielded Twisted-Pair | 26 AWG       |
| Raspberry Pi GPIO24      | TB6600 Y ENA         | ENA\_Y                                                        | 24                     | Shielded Twisted-Pair | 26 AWG       |
| Raspberry Pi GPIO5       | TB6600 Z STEP        | STEP\_Z                                                       | 5                      | Shielded Twisted-Pair | 26 AWG       |
| Raspberry Pi GPIO6       | TB6600 Z DIR         | DIR\_Z                                                        | 6                      | Shielded Twisted-Pair | 26 AWG       |
| Raspberry Pi GPIO13      | TB6600 Z ENA         | ENA\_Z                                                        | 13                     | Shielded Twisted-Pair | 26 AWG       |
| Pi internal +5 V pull-up | Limit Switch (NC)    | LIMIT\_X+/LIMIT\_X-, LIMIT\_Y+/LIMIT\_Y-, LIMIT\_Z+/LIMIT\_Z- | 19, 26, 16, 20, 21, 12 | Twisted-Pair          | 24 AWG       |
| 24 V PSU +24 V           | TB6600 VCC+          | Power                                                         | –                      | Single Core           | ≥ 1.5 mm²    |
| 24 V PSU GND             | TB6600 VCC– & Pi GND | Ground                                                        | –                      | Single Core           | ≥ 1.5 mm²    |


## 2. Yazılım ve Kontrol Katmanları

### 2.1. LinuxCNC + pigpio HAL

* **İşletim Sistemi**: Raspberry Pi OS Lite (64 bit önerilir)
* **Real-Time Kernel**: `linux-image-rt-arm64` + `linux-headers-rt-arm64`
* **Yazılımlar**:

  * `linuxcnc-hal-python`
  * `pigpio`, `python3-pigpio`, `libpigpio-dev`
* **Daemon**: `systemctl enable pigpiod` ile başlatılıp `pigpiod` üzerinden HAL okuma/yazma işlemleri gerçekleştirilir.
* **HAL Dosyası**: `my_machine.hal` içinde `loadrt pigpio`, `loadrt hal_gpio_pins`, `stepgen` ile üç eksen yapılandırılır.
* **Latency**: `latency-test` ile jitter <20 µs doğrulanmalı.

### 2.2. HMI (Yerel Ekran)

* **Ekran**: 7"–10" HDMI dokunmatik LCD veya DSI ekran.
* **Arayüz**: LinuxCNC Axis veya Gmoccapy GUI.
* **Giriş Kontrolleri**: Dokunmatik veya USB klavye/fare.

### 2.3. Web Sunucu ve Uzaktan Kontrol

* **Sunucu Ortamı**: Python (Flask veya FastAPI) tabanlı, bulut veya yerel Docker konteyner.
* **Fonksiyonlar**:

  * STL dosya upload (Max 50 MB)
  * JWT tabanlı kimlik doğrulama
  * Dosya depolama: `/uploads` (roket takım panoya NFS veya Samba da olabilir)
  * HTTP API: `/slice` endpoint’i → URL al → Raspberry Pi tarafından çekilme
* **İletişim**: HTTPS (Let’s Encrypt sertifikası)
* **Veri Formatı**: JSON (durum raporu, %complete, hata mesajları)

### 2.4. Otomasyon Script (Python)

```python
import requests, subprocess, linuxcnc, logging

logging.basicConfig(level=logging.INFO)
url = "https://server/uploads/part.stl"
r = requests.get(url, timeout=30)
with open("/home/pi/linuxcnc/parts/part.stl","wb") as f:
    f.write(r.content)
subprocess.run([
    "CuraEngine", "slice",
    "-l", "/home/pi/linuxcnc/parts/part.stl",
    "-o", "/home/pi/linuxcnc/nc/part.ngc",
    "--load", "/home/pi/linuxcnc/configs/myprinter.def.json"
])
c = linuxcnc.command(); c.mode(linuxcnc.MODE_AUTO); c.file("/home/pi/linuxcnc/nc/part.ngc"); c.run()
logging.info("Job started")
```

### 2.5. Durum Takibi ve Raporlama

* **GPIO Okumaları**: Limit switch durumları, acil durdurma kontrolleri Python’dan okunup web’e iletilebilir.
* **JSON API**: `/status` endpoint’i; `{"x":posx, "y":posy, "z":posz, "state":"running"}`

---

## 3. Entegrasyon ve Test Planı

1. **Donanım Bağlantı Kontrolü**: Multimetre ile güç ve toprak sürekliliği.
2. **PIN Fonksiyon Testi**: LED ve butonlarla GPIO doğrulama.
3. **HAL ve Latency Testi**: `latency-test` araçları.
4. **Kuru Koşu**: Şaft boşta, düşük hız G-code testi.
5. **Yük Testi**: Mininal kesici uç takıp hız profili deneyleri.
6. **IoT İş Akışı**: Web upload → slicing → otomatik başlatma testi.

---

**Not:** Bu belge, tüm sistem bileşenlerini ve katmanlarını içeren mühendislik el kitabı niteliğindedir. Her bölüm detaylı protokol ve test adımlarıyla, sahada ve geliştirme aşamasında referans alınabilir.


