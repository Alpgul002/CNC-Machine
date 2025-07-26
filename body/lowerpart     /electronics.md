https://gemini.google.com/u/2/app/81dee80d69701a28


Proje Raporu: IoT Destekli Akıllı CNC Sistemi
Proje Başlığı: Raspberry Pi ve LinuxCNC Tabanlı, Donanım Korumalı ve Uzaktan Erişimli Modüler CNC Kontrol Sistemi

Hazırlayan: Gemini (Deniz'in Projesi Üzerine Geliştirilmiştir)

Tarih: 26 Temmuz 2025

1. Giriş ve Proje Amacı
Bu proje raporu; düşük maliyetli, yüksek performanslı ve modern bağlantı özelliklerine sahip 3 eksenli bir CNC makinesinin kontrol sistemi tasarımını detaylandırmaktadır. Projenin temel amacı, piyasadaki pahalı ve kapalı kaynaklı kontrol kartlarına bir alternatif olarak, Raspberry Pi 4'ün işlem gücünü ve LinuxCNC'nin endüstriyel standartlardaki yeteneklerini bir araya getirmektir.

Sistem, harici bir "breakout kartı" ihtiyacını minimuma indirerek, sinyal güvenilirliğini bir tampon entegre ile sağlayan, hem yerel bir dokunmatik ekranla (HMI) hem de ağ üzerinden uzaktan (IoT) yönetilebilen esnek bir mimari sunar.

2. Sistem Mimarisi
Kontrol sistemi üç ana katmandan oluşur:

Elektronik Donanım Katmanı: Güç kaynakları, motorlar, sürücüler, sensörler ve sistemi elektriksel gürültü ve hatalardan koruyan arayüz bileşenlerini içerir.

Yazılım ve Gerçek Zamanlı Kontrol Katmanı: Standart bir işletim sistemini, hassas zamanlama gerektiren görevler için gerçek zamanlı bir çekirdek (kernel), CNC görevlerini yürüten ana kontrol yazılımı ve bu yazılımın donanımla haberleşmesini sağlayan sürücüleri kapsar.

Arayüz ve IoT Katmanı: Kullanıcının makine ile etkileşime girdiği yerel grafik arayüz (GUI) ve G-code dosyalarını uzaktan yükleyip işleri başlatmayı sağlayan ağ servislerini içerir.

3. Elektronik Donanım ve Seçim Gerekçeleri
3.1. Ana Kontrolcü: Raspberry Pi 4 Model B (4GB)
Ne İşe Yarar? Sistemin beynidir. LinuxCNC yazılımını çalıştırır, G-code komutlarını yorumlar ve motorları hareket ettirecek STEP/DIR sinyallerini üretir.

Neden Seçildi? Yeterli işlem gücü, Linux için geniş topluluk desteği, PREEMPT_RT kerneli ile gerçek zamanlı performansa uygunluğu ve en önemlisi, doğrudan programlanabilir 40 adet GPIO pini sunması sebebiyle ideal bir seçimdir.

Örnek Ürün: Raspberry Pi 4 Model B 4GB

3.2. Step Motor Sürücüleri: TB6600 (x3)
Ne İşe Yarar? Raspberry Pi'nin ürettiği düşük akımlı mantık sinyallerini (5V), NEMA 23 motorlarını hareket ettirebilecek yüksek akımlı (örn. 2-3A) sinyallere dönüştüren bir amplifikatördür.

Neden Seçildi? Fiyat/performans oranı yüksektir. 4A'e kadar akım desteği, ayarlanabilir mikro-adım (microstepping) özellikleri ile hareket hassasiyetini artırması ve opto-izolasyonlu girişleri sayesinde kontrolcüyü motor gürültüsünden koruması en önemli avantajlarıdır.

Örnek Ürün: TB6600 Step Motor Sürücü Kartı

3.3. Step Motorlar: NEMA 23 (3.0 Nm) (x3)
Ne İşe Yarar? Elektriksel darbeleri, X, Y ve Z eksenlerinde hassas dönme hareketine çevirerek makinenin mekanik aksamını hareket ettirir.

Neden Seçildi? 3.0 Nm tork, küçük ve orta ölçekli hobi/atölye tipi CNC makinelerinin alüminyum, ahşap ve plastik işleme ihtiyaçları için yeterli gücü ve hassasiyeti sunar. NEMA 23, yaygın bulunan bir standarttır.

Örnek Ürün: 3 Nm NEMA 23 Step Motor

3.4. Güç Kaynakları: 24V 10A ve 5V 3A+
Ne İşe Yarar? Sistemin tüm bileşenlerine stabil ve doğru voltajda enerji sağlar.

Neden Seçildi? İki ayrı güç kaynağı kullanmak kritik bir tasarım kararıdır.

24V 10A PSU: Motorların yüksek ve dalgalı akım ihtiyacını karşılar.

5V 3A+ PSU: Raspberry Pi gibi hassas bir dijital devrenin, motor gürültüsünden etkilenmeden stabil çalışmasını garanti eder. Tek bir kaynaktan dönüştürücü ile 5V elde etmek, motorlar yüke bindiğinde Pi'nin çökmesine neden olabilir.

Örnek Ürünler: 24V 10A Metal Kasa Güç Kaynağı ve Raspberry Pi 5.1V 3A Orijinal Güç Adaptörü

3.5. Sinyal Tamponu (Buffer): 74HCT245 Entegresi
Ne İşe Yarar? Raspberry Pi'nin 3.3V'luk GPIO sinyallerini, TB6600 sürücülerinin en kararlı çalıştığı 5V seviyesine yükseltir ve sinyali güçlendirir.

Neden Seçildi? Bu entegre, projenin güvenilirliğinin temel taşıdır. Doğrudan 3.3V bağlantı, özellikle yüksek hızlarda adım kayıplarına neden olabilir. 74HCT serisi, 3.3V giriş sinyalini 5V çıkışa dönüştürmek için özel olarak tasarlanmıştır ve Pi'yi olası bir elektriksel geri tepmeden koruyan ek bir izolasyon katmanı sağlar. Uygulama Notu: Bu entegrenin DIR (Pin 1) ve /OE (Pin 19) pinleri toprağa (GND) bağlanarak yönü ve çıkışları sabitlenmelidir.

Örnek Ürün: 74HCT245 DIP Entegre

3.6. Güvenlik Bileşenleri: Limit Switch (NC) ve E-Stop Butonu (NC)
Ne İşe Yarar? Limit switch'ler makinenin fiziksel sınırlarını tanımlar, E-Stop ise acil bir durumda tüm sistemi anında durdurur.

Neden Seçildi? Her ikisi için de NC (Normally Closed - Normalde Kapalı) tip seçilmesi bir güvenlik standardıdır. Bu sayede, bir switch'in kablosu kopsa veya butonda bir arıza olsa bile devre "açık" konuma geçeceği için sistem bunu bir hata olarak algılar ve kendini durdurur. Bu, fail-safe (hatada güvenli) bir tasarımdır.

Örnek Ürünler: CNC Mekanik Limit Switch ve Acil Stop Butonu

4. Yazılım Katmanı ve Seçim Gerekçeleri
4.1. İşletim Sistemi: Raspberry Pi OS Lite (64-bit)
Ne İşe Yarar? Donanım üzerinde çalışacak temel yazılım platformudur.

Neden Seçildi? "Lite" sürümü, gereksiz grafik arayüzleri ve servisleri barındırmaz. Bu, CNC kontrolü için kritik olan işlemci ve RAM kaynaklarının boşa harcanmasını önler ve sistemin daha kararlı çalışmasını sağlar. 64-bit mimari, modern yazılımlarla uyumluluk ve performans artışı sunar.

4.2. Çekirdek (Kernel): PREEMPT_RT Patch
Ne İşe Yarar? Standart Linux çekirdeğini, bir Gerçek Zamanlı İşletim Sistemine (RTOS) dönüştürür.

Neden Seçildi? CNC kontrolünde motorlara gönderilen STEP sinyallerinin zamanlaması mikrosaniye düzeyinde hassas olmalıdır. Standart bir işletim sistemi, arka plan işlemlerini yürütmek için bu sinyal gönderimini geciktirebilir (bu duruma "jitter" denir). PREEMPT_RT kerneli, bu gecikmeleri minimize ederek (<20-30 µs), motorların adım kaybetmeden, pürüzsüz ve hassas bir şekilde hareket etmesini garanti eder. Bu, projenin olmazsa olmazıdır.

4.3. CNC Kontrol Yazılımı: LinuxCNC
Ne İşe Yarar? G-code'u yorumlayan, hareket planlaması yapan, kullanıcı arayüzünü sunan ve tüm donanımı yöneten ana CNC yazılım paketidir.

Neden Seçildi? Açık kaynaklı, son derece güçlü ve esnektir. En büyük avantajı HAL (Hardware Abstraction Layer - Donanım Soyutlama Katmanı)'dır. HAL sayesinde, "şu GPIO pini, X ekseninin STEP sinyalidir" gibi donanım bağlantıları basit bir metin dosyası ile tanımlanabilir. Bu, projeyi standart dışı donanımlarla (Raspberry Pi GPIO'ları gibi) uyumlu hale getirir.

4.4. HAL Sürücüsü: pigpio
Ne İşe Yarar? LinuxCNC'nin HAL katmanı ile Raspberry Pi'nin GPIO pinleri arasında köprü görevi gören özel bir sürücüdür.

Neden Seçildi? pigpio, Raspberry Pi'nin DMA (Direct Memory Access) motorunu kullanır. Bu sayede, STEP sinyallerini üretme işini CPU'dan alıp doğrudan DMA donanımına devreder. Sonuç olarak, CPU başka işlerle meşgulken bile mikrosaniye hassasiyetinde, sıfıra yakın "jitter" ile sinyal üretilebilir. Bu, Raspberry Pi'yi profesyonel bir hareket kontrol kartına en çok yaklaştıran teknolojidir.

5. Sonuç
Bu rapor, Raspberry Pi tabanlı modern bir CNC kontrol sisteminin tasarımını ve temelini oluşturan mühendislik kararlarını özetlemektedir. Seçilen donanım ve yazılım bileşenleri, aralarındaki sinyal uyumluluğu ve güvenlik katmanları ile birlikte, hem hobi amaçlı kullanıcılar hem de küçük atölyeler için düşük maliyetli, yüksek performanslı ve güvenilir bir platform sunmaktadır. Projenin başarısı, raporda ve bağlantı matrisinde belirtilen topraklama, sinyal tamponlama ve güvenlik devresi prensiplerine titizlikle uyulmasına bağlıdır.

