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

### 2.2. Sabit Gamma ve Dinamik ATR Optimizasyonu (v6.2)
Son güncellemelerle birlikte indikatörün hesaplama stratejisi tamamen otomatik ATR periyodu üzerine kurulmuştur:
*   **Sabit Gamma:** Filtre genişliğini belirleyen Gamma katsayısı her zaman **9.0** değerine sabitlenmiştir.
*   **Konsolide ATR Listesi:** Eskiden çekmeceler halinde ayrılan ATR periyotları tek bir listede birleştirilmiştir (0.09 - 900.0).
*   **Asimetrik Optimizasyon:** Tepe (Peak) ve Dip (Valley) filtreleri, bu geniş listedeki tüm değerleri kullanarak kendileri için en ideal (fiyatı kırmayan en yakın) ATR periyodunu bağımsız olarak seçer.

---

## 3. Filtre Güncelleme Mantığı (upd_filt)

Filtrenin çizilme algoritması `upd_filt` metodu içinde tanımlanmıştır. Bu metod, bir "çapa" noktasından başlar ve her yeni barda şu mantığı izler:

1.  **Yukarı İtme (Peak Tracking):** Eğer `Yüksek Fiyat - r > Mevcut Filtre` ise, filtre yukarı kaydırılır.
2.  **Aşağı İtme (Valley Tracking):** Eğer `Düşük Fiyat + r < Mevcut Filtre` ise, filtre aşağı kaydırılır.
3.  **Sabit Kalma:** Fiyat bu sınırların arasındaysa, filtre değerini korur (yatay basamak çizer).

---

## 4. Ek Katman Filtreleri (Görünmez Mekanizmalar)

Grafikte görünen ana filtrelerin haricinde, sistemi stabilize eden ve "filtre içinde filtre" görevi gören 3 ek katman bulunmaktadır:

### 4.1. Simülasyon Katmanı (Auto Motor)
Sistem arka planda 0.09'dan 900.0'a kadar olan tüm ATR değerlerini simüle eder. Fiyatın temas ettiği katmanlar elenir ve en yakın kararlı katman seçilir.

### 4.2. Çapa Sabitleme Katmanı (Anchor Filters)
- **Settle Threshold:** Fiyat filtreden belirli bir mesafe uzaklaşana kadar çapa noktası değişmez.
- **Proximity Threshold:** Fiyat filtreye çok yaklaştığında sistem çapa noktasını güncelleyerek kendini yeniden kalibre eder.

---

## 5. Görselleştirme ve Çapalama
- **Lookback Mode:** Geçmişteki en yüksek tepe veya en düşük dip noktalarını bulur.
- **Stepline (Basamaklı):** Filtrenin yatay kaldığı klasik görünüm.

---

## Özet
Dokuzatr v6.2, karmaşık Gamma ayarlarını ortadan kaldırarak odağını **zaman duyarlılığına (ATR Periyodu)** çevirmiştir. Sabit 9.0 Gamma koruması altında, piyasanın hızına göre kendini 0.09 ile 900 bar arasında bir derinliğe otomatik olarak ayarlar.
