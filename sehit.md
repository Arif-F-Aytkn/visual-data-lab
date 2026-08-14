## 🌲 Proje Hakkında

Bu çalışma, 1968 - 2025 yılları arasında orman yangınlarıyla mücadelede yitirdiğimiz 179 kahramanın yıllara göre dağılımını görselleştirmek amacıyla hazırlanmıştır. Klasik sütun grafikler yerine, veriyi daha estetik ve modern bir infografik formatında sunan **Lollipop Chart (Lolipop Grafiği)** tercih edilmiştir.

### 📊 Veri Seti
Projede kullanılan `sehitler.csv` dosyası, Orman Genel Müdürlüğü (OGM) resmi kaynaklarından derlenmiştir. Veri seti, şehitlerimizin;

* Ad-Soyad
* Unvan / Görev
* Şehadet Tarihi
* Görev Yeri (Bölge/İşletme/Şeflik) bilgilerini içermektedir.

---

## 💻 Görselleştirme Kodu

Aşağıdaki kod bloğu veriyi okur, yıllara göre özetler ve grafiği çizer.

```{r lollipop-chart}
# Gerekli Kütüphaneler
library(tidyverse)

# 1. Veriyi Okuma
sehitler <- read.csv("sehitler.csv", stringsAsFactors = FALSE, encoding = "UTF-8")

# 2. Tarihten Yıl Bilgisini Çıkarma ve Yıllık Toplamları Hesaplama
sehitler_yillik <- sehitler %>%
  # Tarih formatından yılı alıyoruz (DD.MM.YYYY)
  mutate(Yil = as.numeric(sub(".*\\.", "", Tarih))) %>%
  group_by(Yil) %>%
  summarise(Sehit_Sayisi = n()) %>%
  ungroup()

# 3. Lollipop Grafiğini Oluşturma
ggplot(sehitler_yillik, aes(x = Yil, y = Sehit_Sayisi)) +
  # Çizgi (Çöp) Kısmı
  geom_segment(aes(x = Yil, xend = Yil, y = 0, yend = Sehit_Sayisi), 
               color = "gray60", size = 0.8) +
  # Nokta (Lolipop Başı) Kısmı
  geom_point(color = "#B22222", size = 3.5, alpha = 0.9) +
  # 5 ve Üzeri Şehit Verilen Kritik Yıllara Sayı Etiketi Ekleme (Okunabilirlik İçin)
  geom_text(data = subset(sehitler_yillik, Sehit_Sayisi >= 5),
            aes(label = Sehit_Sayisi),
            vjust = -0.8, size = 3.2, fontface = "bold", color = "#8B0000") +
  # Tema ve Eksen Ayarları
  scale_x_continuous(breaks = seq(1970, 2025, by = 5)) +
  scale_y_continuous(expand = expansion(mult = c(0, 0.15))) +
  labs(
    title = "Yeşil Vatan İçin Verilen Canlar",
    subtitle = "Yıllara Göre Orman Yangınlarında Yitirdiğimiz Şehit Sayısı (1968 - 2025)",
    x = "Yıl",
    y = "Şehit Sayısı",
    caption = "Kaynak: OGM Kayıtları | Toplam: 179 Şehit"
  ) +
  theme_minimal(base_family = "sans") +
  theme(
    plot.title = element_text(face = "bold", size = 16, color = "#1A1A1A"),
    plot.subtitle = element_text(size = 11, color = "#555555", margin = margin(b = 15)),
    panel.grid.major.x = element_blank(), # Dikey çizgileri kaldırıp temiz bir görünüm veriyoruz
    panel.grid.minor = element_blank(),
    axis.title.x = element_text(margin = margin(t = 10), face = "bold"),
    axis.title.y = element_text(margin = margin(r = 10), face = "bold")
  )
```

---

## 🎨 Tasarım Detayları

* **Veri Vurgusu:** Görselin okunabilirliğini artırmak adına yalnızca **5 ve daha fazla şehit verdiğimiz** kritik yılların (örn. 1985, 2004, 2021 ve 2025) üzerine veri etiketleri (label) eklenmiştir.
* **Minimalist Tema:** `theme_minimal()` ile grafik arkasındaki dikey çizgiler kaldırılarak dikkat doğrudan veri trendine odaklanmıştır.
* **Renk Paleti:** Alev ve yas temasını simgelemesi açısından kırmızı ve gri tonları (`#B22222` ve `#8B0000`) tercih edilmiştir.

> **Not:** *Bu grafik, özel bir anma koleksiyonunun ve 20 sayfalık infografik sergisinin giriş sayfası olarak tasarlanmıştır.*
