---
title: "Kredi Analiz Raporu"
author: "Veri Analizi"
date: "`r Sys.Date()`"
output: html_document
---

```{r setup, include=FALSE}
# R Markdown ayarları (çıktıda uyarı ve mesajları gizlemek için)
knitr::opts_chunk$set(echo = TRUE, warning = FALSE, message = FALSE)
```

## Gerekli Kütüphaneler ve Veri Yükleme

```{r veri_yukleme}
Sys.setlocale("LC_ALL", "Turkish")

library(readxl)
library(dplyr)
library(ggplot2)
library(tidyr)
library(scales)

# Veriyi İçe Aktarma (Dosya yolunu kendi bilgisayarına göre güncelle)
Rapor_1_ <- read_excel("C:/Users/afay1/Downloads/Rapor (1).xlsx",
                       col_types = c("text", rep("numeric", 11)))
```

## GRAFİK 1: En Çok Kredi Kullanan İlk 10 Şehir

Kredi türlerine göre yığılmış (stacked) olarak Türkiye genelinde en çok kredi kullanan ilk 10 ilin dağılımı:

```{r grafik_1, fig.width=10, fig.height=6}
# 1. "TOPLAM" satırını hariç tut, sırala ve ilk 10 şehri al
ilk_10_kredi <- Rapor_1_ %>%
  filter(Şehir != "TOPLAM") %>% 
  arrange(desc(`Toplam Kredi`)) %>%
  head(10)

# 2. X ekseninde sıralı görünmesi için factor yapıyoruz
ilk_10_kredi$Şehir <- factor(ilk_10_kredi$Şehir, levels = ilk_10_kredi$Şehir)

# 3. Kredi türlerini uzun (long) formata çeviriyoruz
uzun_veri_stack <- pivot_longer(ilk_10_kredi, 
                                cols = c(`Taşıt Kredisi`, `Konut Kredisi`, `Kredili Mevduat Hesabı`, 
                                         `Diğer Tüketici Kredileri`, `Bireysel Kredi Kartları`),
                                names_to = "Kredi_Turu", 
                                values_to = "Miktar")

# 4. Yığılmış (Stacked) Grafiği Çizdirme
ggplot(uzun_veri_stack, aes(fill = Kredi_Turu, y = Miktar, x = Şehir)) + 
  geom_bar(position = "stack", stat = "identity") +
  scale_y_continuous(labels = label_comma(big.mark = ".", decimal.mark = ",")) +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
  labs(title = "En Çok Kredi Kullanan İlk 10 Şehir",
       x = "Şehir",
       y = "Kredi Miktarı (TL)",
       fill = "Kredi Türü")
```

## GRAFİK 2: Takipteki Kredi Oranları

Takipteki kredi oranının en yüksek olduğu 10 il, en düşük olduğu 10 il ve Türkiye ortalamasının karşılaştırması:

```{r grafik_2, fig.width=12, fig.height=6}
# 1. Türkiye ortalamasını ayırıp adını değiştiriyoruz
turkiye_ortalama <- Rapor_1_ %>%
  filter(Şehir == "TOPLAM") %>%
  mutate(Şehir = "TÜRKİYE ORTALAMASI")

# 2. Şehirleri (TOPLAM hariç) sıralayıp ilk 10 ve son 10 şehri seçiyoruz
sehirler_veri <- Rapor_1_ %>%
  filter(Şehir != "TOPLAM") %>%
  arrange(desc(`Takipteki Kredi Oranı`))

ilk_10_takip <- head(sehirler_veri, 10)
son_10_takip <- tail(sehirler_veri, 10)

# 3. Verileri birleştirip renklendirme için Kategori sütunu ekliyoruz
birlestirilmis_veri <- bind_rows(ilk_10_takip, turkiye_ortalama, son_10_takip) %>%
  mutate(Kategori = case_when(
    Şehir == "TÜRKİYE ORTALAMASI" ~ "Türkiye Ortalaması",
    Şehir %in% ilk_10_takip$Şehir ~ "En Yüksek 10 İl",
    Şehir %in% son_10_takip$Şehir ~ "En Düşük 10 İl"
  ))

# 4. Çubukların sıralı görünmesi için factor yapıyoruz
birlestirilmis_veri <- birlestirilmis_veri %>% arrange(desc(`Takipteki Kredi Oranı`))
birlestirilmis_veri$Şehir <- factor(birlestirilmis_veri$Şehir, levels = birlestirilmis_veri$Şehir)

# 5. Grafiği Çizdirme
ggplot(birlestirilmis_veri, aes(x = Şehir, y = `Takipteki Kredi Oranı`, fill = Kategori)) +
  geom_bar(stat = "identity") +
  scale_fill_manual(values = c("En Yüksek 10 İl" = "#FF6A6A", 
                               "Türkiye Ortalaması" = "#FF0000", 
                               "En Düşük 10 İl" = "#8DEEEE")) +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1),
        legend.position = "none") +
  labs(title = "İllere Göre Takipteki Kredi Oranları (İlk ve Son 10 İl)",
       x = "Şehir",
       y = "Takipteki Kredi Oranı (%)")
```
