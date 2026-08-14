Bu proje, Türkiye'nin son 10 yıllık dış ticaret verilerini (İhracat, İthalat ve Dış Ticaret Dengesi) kullanarak yüksek çözünürlüklü, dikey formatta (mobil cihazlara uygun) ve hareketli bir grafik (GIF) oluşturmayı amaçlamaktadır.

## 1. Gerekli Kütüphanelerin Yüklenmesi

Veri temizleme, dönüştürme ve animasyonlu görselleştirme için aşağıdaki paketleri kullanıyoruz:

```r
library(readxl)
library(ggplot2)
library(gganimate)
library(hrbrthemes)
library(dplyr)
library(tidyr)
library(scales)
library(gifski)
```

## 2. Verinin İçe Aktarılması ve Temizlenmesi

Excel dosyasını içe aktardıktan sonra, sadece gerekli sütunları seçiyor, isimlerini standartlaştırıyor ve veriyi `ggplot2`'nin çalışabileceği uzun (long) formata çeviriyoruz. Ayrıca rakamların içindeki gizli boşlukları silip tam sayı formatına dönüştürüyoruz.

*(Not: Bu kodun çalışması için Excel dosyasının proje klasöründe olması gerekir.)*

```r
# Veriyi oku
veriler <- read_excel("Dis ticaretin son on yili.xlsx")

# Sütun seçimi ve yeniden isimlendirme
temiz_veri <- veriler %>%
  select(
    Yil = Yıllar,                     
    Ihracat = İhracat,                
    Ithalat = İthalat,                
    Denge = `Dış Ticaret Dengesi`     
  )

# Veriyi uzun formata (long format) çevirme
veriler_uzun <- temiz_veri %>%
  pivot_longer(
    cols = c(Ihracat, Ithalat, Denge),
    names_to = "Kategori",
    values_to = "Deger"
  )

# Veri tiplerini dönüştürme (Boşlukları temizleme)
veriler_uzun$Deger <- as.numeric(gsub("\\s+", "", veriler_uzun$Deger))
veriler_uzun$Yil <- as.numeric(veriler_uzun$Yil)
```

## 3. Animasyonlu Grafiğin Tasarlanması

Temizlenen veri seti ile statik grafiğimizi oluşturuyor ve `gganimate` paketinden `transition_reveal()` fonksiyonu ile grafiğe zaman ekseninde hareket kazandırıyoruz.

```r
# Bilimsel gösterimi (1e6 vb.) kapat
options(scipen = 999) 

animasyon <- ggplot(veriler_uzun, aes(x = Yil, y = Deger, group = Kategori, color = Kategori)) +    
  geom_line(linewidth = 1.5) +  
  geom_point(size = 4) +       
  scale_color_viridis_d(option = "D") +    
  
  scale_y_continuous(
    labels = comma_format(big.mark = ".", decimal.mark = ","), 
    breaks = pretty_breaks(n = 10) 
  ) +
  
  scale_x_continuous(
    breaks = seq(min(veriler_uzun$Yil), max(veriler_uzun$Yil), by = 2)
  ) +
  
  ggtitle("Turkiye Dis Ticaretinin Son 10 Yillik Degisimi") +    
  theme_ipsum() +
  
  theme(
    axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1),
    axis.title.y = element_text(angle = 90, hjust = 0.5, vjust = 0.5, size = 14, margin = margin(r = 15)),
    panel.grid.minor.x = element_blank(),
    legend.position = "bottom"
  ) +
  
  ylab("Ticaret Degeri ($)") + 
  xlab("Yil") +
  labs(color = "Gostergeler:") + 
  
  # Animasyon katmanı
  transition_reveal(Yil)
```

## 4. Yüksek Kaliteli GIF Olarak Dışa Aktarma

Grafiği sosyal medya platformlarına (Reels, Shorts) tam uyumlu olması için dikey (9:16) formatta ve yüksek çözünürlükte (HD) işleyerek kaydediyoruz.

```r
# Render işlemi (Kalite ve çözünürlük ayarları)
yuksek_kalite_anim_dik <- animate(
  plot = animasyon, 
  width = 1080,        
  height = 1920,       
  res = 150,           
  nframes = 200,       
  fps = 20,            
  renderer = gifski_renderer()
)

# Dosyayı kaydet
anim_save("dis_ticaret_animasyonu.gif", animation = yuksek_kalite_anim_dik)
```
