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

## 4. Otomatik Çapa (Auto Anchor) ve Sabitleme

İndikatörün en güçlü yanı, filtrenin nereden başlayacağını (çapa noktasını) dinamik olarak seçebilmesidir.

- **Lookback Mode:** Manuel, Otomatik (Hassasiyet), Global Maksimum gibi modlarla geçmişteki en yüksek tepe veya en düşük dip bulunur.
- **Settle Threshold (Sabitleme Eşiği):** Fiyat, mevcut filtreden belirli bir oranda (`auto_threshold * sz`) uzaklaşırsa, çapa noktası güncel tepe/dip noktasına "resetlenir".
- **Proximity Threshold (Yaklaşma Eşiği):** Fiyat filtreye çok yaklaşırsa (`auto_prox_threshold`), çapa yine güncellenir. Bu, trend değişimlerini veya konsolidasyonları yakalamak için kullanılır.

---

## 5. Otomatik Gamma Motoru (Auto Gamma Motor)

`auto_gamma_mode` aktif edildiğinde, indikatör brute-force (kaba kuvvet) benzeri bir optimizasyon döngüsü çalıştırır:

1.  **Tarama:** Önceden tanımlanmış Gamma veya ATR değerleri listesi (Çekmeceler) üzerinde döngüye girer.
2.  **Simülasyon:** Her bir değer için çapa noktasından günümüze kadar bir filtre simülasyonu yapar.
3.  **Başarısızlık Kontrolü:** Eğer fiyat (`close`), simülasyon sırasında filtreyi aşağı veya yukarı yönlü kırarsa, o değer "başarısız" kabul edilir.
4.  **En Uygun Seçim:** Fiyatı kırmayan (en yakın takip eden) ilk ve en küçük Gamma/ATR değeri "ideal" olarak seçilir ve grafiğe yansıtılır.

---

## 6. Görselleştirme

- **Stepline (Basamaklı):** Filtrenin yatay kaldığı ve sadece fiyat ittiğinde hareket ettiği klasik görünüm.
- **Diagonal (Eğik):** Filtrenin noktalar arasında düz çizgilerle bağlandığı görünüm.
- **Polyline:** Performans optimizasyonu için tüm çizimler Pine Script'in `polyline` fonksiyonu ile barstate.islast durumunda tek seferde (veya değişimlerde) çizilir.

---

## Özet
Dokuzatr, **KAMA-ATR** ile piyasa oynaklığını ölçen, **Auto Gamma Motoru** ile en uygun hassasiyeti kendi bulan ve **Dinamik Çapalama** ile trendin başlangıç noktasını sürekli güncelleyen ileri seviye bir takip (trailing) sistemidir. Filitre çizme mantığı, fiyatın extrem noktalarından itibaren volatilite kadar bir "güvenlik alanı" bırakma ve bu alanın ihlali durumunda kendini yeniden optimize etme üzerine kuruludur.
