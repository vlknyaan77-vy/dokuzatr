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
- **Sonuç:** `f_kama_atr` fonksiyonu, piyasa gürültüsünün az olduğu (trend olan) dönemlerde daha hızlı, gürültülü dönemlerde ise daha yavaş tepki veren bir volatilite ölçümü sağlar. Bu, filtrenin gereksiz kırılmalarını (whipsaw) engeller.

### 2.2. Gamma Katsayısı
Gamma, hesaplanan KAMA-ATR değerinin ne kadar genişletileceğini belirleyen bir çarpandır.
- `Filtre Mesafesi (r) = KAMA(TR, Period) * Gamma`
- Gamma değeri ne kadar büyükse, filtre fiyattan o kadar uzaklaşır ve trend takibi o kadar "geniş" olur.

---

## 3. Filtre Güncelleme Mantığı (upd_filt)

Filtrenin çizilme algoritması `upd_filt` metodu içinde tanımlanmıştır. Bu metod, bir "çapa" noktasından başlar ve her yeni barda şu mantığı izler:

1.  **Yukarı İtme (Peak Tracking):** Eğer `Yüksek Fiyat - r > Mevcut Filtre` ise, filtre yukarı kaydırılır.
2.  **Aşağı İtme (Valley Tracking):** Eğer `Düşük Fiyat + r < Mevcut Filtre` ise, filtre aşağı kaydırılır.
3.  **Sabit Kalma:** Fiyat bu sınırların arasındaysa, filtre değerini korur (yatay basamak çizer).

Bu mantık, fiyatın extrem noktalarından itibaren belirlenen volatilite kadar bir "esneklik payı" bırakır.

---

## 4. Ek Katman Filtreleri (Görünmez Mekanizmalar)

Grafikte görünen ana filtrelerin haricinde, sistemi stabilize eden ve "filtre içinde filtre" görevi gören 3 ek katman bulunmaktadır:

### 4.1. Simülasyon Katmanı (Auto Gamma Motor)
`auto_gamma_mode` aktif olduğunda, sistem arka planda onlarca farklı filtreyi simüle eder.
- **Dinamik Seçim:** Seçilen "Çekmece" içindeki tüm Gamma değerleri için hayali filtreler oluşturulur.
- **Eleme Filtresi:** Fiyatın (`close`) temas ettiği veya kırdığı tüm katmanlar elenir.
- **Sonuç:** Kırılmadan kalan en yakın (en hassas) katman ana filtre olarak atanır.

### 4.2. Çapa Sabitleme Katmanı (Anchor Filters)
Filtrenin nereden başlayacağını belirleyen mantıksal bir filtredir:
- **Settle Threshold (Sabitleme Eşiği):** Fiyat filtreden belirli bir mesafe uzaklaşana kadar çapa noktası değişmez. Bu, geçici dalgalanmaların (noise) filtreyi bozmasını engeller.
- **Proximity Threshold (Yaklaşma Eşiği):** Fiyat filtreye çok yaklaştığında sistem bunu bir "tehdit" veya "potansiyel trend değişimi" olarak algılar ve çapa noktasını güncelleyerek kendini yeniden kalibre eder.

### 4.3. Adaptif Volatilite Filtresi (KAMA-TR)
Hesaplamanın en başında ham volatilite (True Range) verisi bir "verimlilik" filtresinden geçer. Piyasa verimli (trendli) ise filtre daralır, piyasa verimsiz (testere/yatay) ise filtre genişleyerek hatalı sinyalleri engeller.

---

## 5. Otomatik Çapa (Auto Anchor) ve Sabitleme

İndikatörün en güçlü yanı, filtrenin nereden başlayacağını (çapa noktasını) dinamik olarak seçebilmesidir.
- **Lookback Mode:** Manuel, Otomatik (Hassasiyet), Global Maksimum gibi modlarla geçmişteki en yüksek tepe veya en düşük dip bulunur.

---

## 6. Görselleştirme

- **Stepline (Basamaklı):** Filtrenin yatay kaldığı ve sadece fiyat ittiğinde hareket ettiği klasik görünüm.
- **Diagonal (Eğik):** Filtrenin noktalar arasında düz çizgilerle bağlandığı görünüm.
- **Polyline:** Performans optimizasyonu için tüm çizimler Pine Script'in `polyline` fonksiyonu ile barstate.islast durumunda tek seferde çizilir.

---

## Özet
Dokuzatr, **KAMA-ATR** ile piyasa oynaklığını ölçen, **Auto Gamma Motoru** ile en uygun hassasiyeti kendi bulan ve **Dinamik Çapalama** ile trendin başlangıç noktasını sürekli güncelleyen ileri seviye bir takip (trailing) sistemidir. Filitre çizme mantığı, fiyatın extrem noktalarından itibaren volatilite kadar bir "güvenlik alanı" bırakma ve bu alanın ihlali durumunda kendini yeniden optimize etme üzerine kuruludur.
