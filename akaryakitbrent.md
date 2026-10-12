```r
# Gerekli Kütüphaneler
library(readxl)
library(readr)
library(dplyr)
library(lubridate)
library(tidyr)
library(ggplot2)

# 1. Verileri Okuma ve Düzenleme
df_akaryakit <- read_excel("Gecmisdonemankarakaryakitfiyatlari.xlsx") %>%
  select(1, 2, 4) %>%
  rename(Tarih = 1, Benzin = 2, Motorin = 3) %>%
  mutate(Tarih = as.Date(Tarih))

df_brent <- read_csv("Brent Petrol Vadeli İşlemleri Geçmiş Verileri.csv", show_col_types = FALSE) %>%
  select(1, 2) %>%
  rename(Tarih = 1, Brent_Fiyati = 2) %>%
  mutate(Tarih = dmy(Tarih), Brent_Fiyati = Brent_Fiyati / 100)

# 2. Verileri Birleştirme, Eksiklikleri Doldurma ve Son 3 Yılı Filtreleme
df_son3yil <- df_akaryakit %>%
  full_join(df_brent, by = "Tarih") %>%
  arrange(Tarih) %>%
  fill(Benzin, Motorin, Brent_Fiyati, .direction = "down") %>%
  filter(!is.na(Benzin) & !is.na(Brent_Fiyati)) %>%
  filter(Tarih >= as.Date("2023-10-01"))

# 3. İki Eksen İçin Ölçek Katsayısı Hesaplama
scale_factor <- max(df_son3yil$Brent_Fiyati, na.rm = TRUE) / max(df_son3yil$Motorin, na.rm = TRUE)

# 4. ggplot2 ile Görselleştirme
p_son3yil <- ggplot(df_son3yil, aes(x = Tarih)) +
  geom_line(aes(y = Benzin, color = "Kurşunsuz Benzin"), linewidth = 1.2) +
  geom_line(aes(y = Motorin, color = "Motorin"), linewidth = 1.2) +
  geom_line(aes(y = Brent_Fiyati / scale_factor, color = "Brent Petrol ($)"), linewidth = 1.2, alpha = 0.7) +
  scale_y_continuous(
    name = "Akaryakıt Fiyatları (TL/LT)",
    sec.axis = sec_axis(~ . * scale_factor, name = "Brent Petrol Fiyatı ($)")
  ) +
  scale_color_manual(
    values = c("Kurşunsuz Benzin" = "#D55E00", "Motorin" = "#E69F00", "Brent Petrol ($)" = "#56B4E9"),
    name = "Gösterge"
  ) +
  labs(title = "Son 3 Yıllık Akaryakıt ve Brent Petrol Fiyatları", x = "Tarih") +
  theme_minimal() +
  theme(
    legend.position = "top",
    plot.title = element_text(hjust = 0.5, face = "bold", size = 16),
    axis.title.y.left = element_text(face = "bold", size = 12),
    axis.title.y.right = element_text(color = "#56B4E9", face = "bold", size = 12),
    axis.text = element_text(size = 11),
    legend.text = element_text(size = 12)
  )

# 5. Instagram Formatında Kaydetme
ggsave(filename = "instagram_son3yil_grafigi.png", plot = p_son3yil, width = 10, height = 8, dpi = 300)
```
