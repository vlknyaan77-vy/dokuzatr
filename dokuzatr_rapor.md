# Dokuzatr (Volcanic Auto Peaks & Valleys v6) - Filtre Çizme Mantığı Raporu

Bu rapor, "Volcanic Auto Peaks & Valleys v6" (dokuzatr) indikatörünün çalışma prensiplerini, filtreleme mekanizmalarını ve kullanılan matematiksel modelleri detaylandırmak amacıyla hazırlanmıştır.

---

## 1. Genel Bakış
Dokuzatr, piyasadaki tepe ve dip noktalarını dinamik bir şekilde takip eden ve bu noktalardan itibaren fiyatın dalgalanma payını (volatiliteyi) hesaplayarak bir "takip eden filtre" (trailing filter) çizen bir teknik analiz aracıdır. Diğer indikatörlerden temel farkı, **çapa (anchor)** noktalarını otomatik belirlemesi ve bu noktalardan itibaren **KAMA tabanlı ATR** kullanarak esnek bir koridor oluşturmasıdır.

---

## 2. Temel Matematiksel Bileşenler

### 2.1. KAMA Tabanlı ATR (Adaptive ATR)
İndikatör, klasik ATR (Average True Range) yerine Kaufman'ın Uyarlanabilir Hareketli Ortalaması (KAMA) mantığını True Range üzerinde uygular.
- **Efficiency Ratio (ER):** Fiyatın yönlü hareketi ile toplam oynaklığı arasındaki oran hesaplanır.
- **Smoothing Constant (SC):** ER'ye bağlı olarak hızlanan veya yavaşlayan bir düzeltme katsayısıdır.
- **Sonuç:** `f_kama_atr` fonksiyonu, piyasa gürültüsünün az olduğu (trend olan) dönemlerde daha hızlı, gürültülü dönemlerde ise daha yavaş tepki veren bir volatilite ölçümü sağlar.

### 2.2. ATR Modu ve ATR Periyodu İşleyişi (Tepe ve Dip İçin Bağımsız)
İndikatörde her iki filtre (Tepe ve Dip) için de ATR periyodu **ayrı ayrı ve bağımsız** olarak çalışır:

#### A. Standart Mod (ATR Modu Kapalı)
*   Hem Tepe (Peak) hem de Dip (Valley) filtreleri, genel ayarlardaki sabit **ATR Periyodu** (varsayılan 9) değerini kullanır.
*   Bu modda filtrelerin hassasiyeti Gamma çarpanı ile ayarlanır.

#### B. ATR Modu (Etkinleştirildiğinde)
*   Tepe ve Dip için ayrı "ATR Modunu Etkinleştir" seçenekleri bulunur (`p_use_atr_mode` ve `v_use_atr_mode`).
*   Eğer etkinse: Gamma 9.0'a sabitlenir ve o filtreye özel seçilen **ATR Periyodu** (Tepe ATR veya Dip ATR) hesaplamaya dahil edilir.
*   **Örnek:** Tepe filtresi 14 periyotluk ATR ile yavaş hareket ederken, Dip filtresi 5 periyotluk ATR ile çok daha agresif/hızlı takip yapabilir.

---

## 3. Filtre Güncelleme Mantığı (upd_filt)

Filtrenin çizilme algoritması `upd_filt` metodu içinde tanımlanmıştır. Bu metod, bir "çapa" noktasından başlar ve her yeni barda şu mantığı izler:

1.  **Yukarı İtme (Peak Tracking):** Eğer `Yüksek Fiyat - r > Mevcut Filtre` ise, filtre yukarı kaydırılır.
2.  **Aşağı İtme (Valley Tracking):** Eğer `Düşük Fiyat + r < Mevcut Filtre` ise, filtre aşağı kaydırılır.
3.  **Sabit Kalma:** Fiyat bu sınırların arasındaysa, filtre değerini korur (yatay basamak çizer).

---

## 4. Ek Katman Filtreleri (Görünmez Mekanizmalar)

Grafikte görünen ana filtrelerin haricinde, sistemi stabilize eden ve "filtre içinde filtre" görevi gören 3 ek katman bulunmaktadır:

### 4.1. Simülasyon Katmanı (Auto Gamma Motor)
`auto_gamma_mode` aktif olduğunda, sistem arka planda onlarca farklı filtreyi simüle eder.
- **Eleme Filtresi:** Fiyatın temas ettiği veya kırdığı tüm katmanlar elenir. En yakın kararlı katman seçilir.

### 4.2. Çapa Sabitleme Katmanı (Anchor Filters)
- **Settle Threshold (Sabitleme Eşiği):** Fiyat filtreden belirli bir mesafe uzaklaşana kadar çapa noktası değişmez.
- **Proximity Threshold (Yaklaşma Eşiği):** Fiyat filtreye çok yaklaşırsa sistem çapa noktasını güncelleyerek kendini yeniden kalibre eder.

---

## 5. Görselleştirme ve Çapalama
- **Lookback Mode:** Geçmişteki en yüksek tepe veya en düşük dip noktalarını bulur.
- **Stepline (Basamaklı):** Filtrenin yatay kaldığı klasik görünüm.
- **Polyline:** Performans için çizimler `polyline` fonksiyonu ile optimize edilmiştir.

---

## Özet
Dokuzatr'da ATR periyodu hesaplaması **asimetriktir**. Yani tepe ve dip filtreleri birbirinden tamamen farklı zaman derinliklerine (periyotlara) sahip olabilir. Bu, piyasanın yukarı ve aşağı yönlü hareketlerindeki hız farklarını (volatilite farklarını) ayrı ayrı optimize etmenize olanak tanır.
