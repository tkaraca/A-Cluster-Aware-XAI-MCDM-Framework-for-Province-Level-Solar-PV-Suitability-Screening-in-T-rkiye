# PV XAI-ANN-MCDM V4 — Türkiye Province Cluster-Aware Extension

Bu paket, V3'te doğrulanan **capacity-normalized, leakage-safe, multi-XAI + MEREC-MCDM** yaklaşımını Türkiye'nin 81 ili için **province-level / makro screening** çalışmasına genişletir.

V4'ün ana fikri:

1. V3 gerçek PV üretim verileriyle ANN-XAI kriter ağırlıklarını üretir.
2. Türkiye'nin 81 ili için NASA POWER saatlik solar/meteoroloji verisi indirilir.
3. İller solar-iklim profillerine göre cluster edilir.
4. V3 ANN modeli, her ilin beklenen PV performansını tahmin eder.
5. Applicability-domain skoru ile modelin hangi illerde daha güvenli transfer edildiği ölçülür.
6. Her cluster içinde ve Türkiye genelinde MCDM sıralaması yapılır.

> Not: Bu çalışma il merkezi/province-center koordinatlarını kullanan **makro düzey aday il taraması**dır. Gerçek santral parsel seçimi için sonraki aşamada grid/GIS katmanları gerekir.

---

## Kaynak notları

- İl koordinatları `inputs/turkiye_province_candidates.csv` dosyasında hazırdır. Kaynak: `ozdemirburak/cities_of_turkey.json` GitHub Gist. 81 ilin adı, latitude, longitude, population ve region alanları kullanıldı.
- Saatlik meteoroloji/solar veri çekimi için `scripts/11_fetch_nasa_power_hourly.py` NASA POWER Hourly API kullanır.
- NASA POWER parametreleri:
  - `ALLSKY_SFC_SW_DWN`: solar irradiance proxy
  - `T2M`: 2 m temperature
  - `RH2M`: 2 m relative humidity
  - `CLOUD_AMT`: cloud amount
  - `WS10M`: 10 m wind speed

---

## Klasör yapısı

```text
pv_xai_mcdm_pipeline_v4_cluster_turkiye/
│
├── data/
│   ├── five_cities_clean_day_combined_all_columns.csv
│   └── five_cities_common_model_view.csv
│
├── inputs/
│   ├── city_coordinates.csv
│   ├── installed_capacities_kWp.csv
│   ├── turkiye_province_candidates.csv
│   └── installed_capacities_kWp_TEMPLATE.csv
│
├── scripts/
│   ├── 01_prepare_dataset.py
│   ├── 02a_leakage_audit.py
│   ├── 02b_train_baseline_models.py
│   ├── 02c_train_ann_extract_xai_weights.py
│   ├── 03_build_decision_matrix.py
│   ├── 04_run_mcdm_ranking.py
│   ├── 05_sensitivity_analysis.py
│   ├── 07_common_period_sensitivity.py
│   ├── 08_xai_method_sensitivity.py
│   ├── 09_repeated_seed_stability.py
│   ├── 10_prepare_turkiye_province_candidates.py
│   ├── 11_fetch_nasa_power_hourly.py
│   ├── 12_build_province_feature_table.py
│   ├── 13_train_ann_predict_turkiye_provinces.py
│   ├── 14_cluster_turkiye_provinces.py
│   ├── 15_clusterwise_mcdm_turkiye_provinces.py
│   ├── run_all_v4_with_fetch.py
│   └── run_all_v4_no_fetch_existing_meteo.py
│
└── outputs/
```

---

# Çalıştırma sırası

İlk turda tek tek Run etmen önerilir.

## Aşama 0 — V3 doğrulama pipeline'ını tekrar üret

Önce V3 tarafı çalışır. Bu aşama gerçek santral verisinden ANN-XAI ağırlığını üretir.

### Scriptler

```text
01_prepare_dataset.py
02a_leakage_audit.py
02b_train_baseline_models.py
02c_train_ann_extract_xai_weights.py
03_build_decision_matrix.py
04_run_mcdm_ranking.py
05_sensitivity_analysis.py
07_common_period_sensitivity.py
08_xai_method_sensitivity.py
09_repeated_seed_stability.py
```

### Ana çıktılar

```text
outputs/01_prepared_hourly_dataset.csv
outputs/02a_leakage_audit_report.csv
outputs/02b_model_comparison_metrics.csv
outputs/02c_ann_model_metrics.csv
outputs/02_ann_xai_feature_weights.csv
outputs/04_mcdm_rankings_alpha_0_50.csv
outputs/05_sensitivity_alpha_rankings.csv
outputs/07_common_period_rankings_alpha_0_50.csv
outputs/08_xai_method_sensitivity_rankings.csv
outputs/09_repeated_seed_ann_metrics_summary.csv
```

Makalede bu çıktılar V3 metodunun geçerlilik kanıtıdır:

- leakage audit
- ANN performansı
- baseline karşılaştırması
- XAI ağırlık karşılaştırması
- repeated-seed stability
- common-period sensitivity

---

# V4 Türkiye il/cluster genişletmesi

## Aşama 10 — Türkiye il adaylarını doğrula

### Run

```text
scripts/10_prepare_turkiye_province_candidates.py
```

### Input

```text
inputs/turkiye_province_candidates.csv
```

### Output

```text
outputs/10_turkiye_province_candidates_validated.csv
```

Bu dosyada 81 il, koordinatlar, bölge bilgisi ve V3 eğitim şehirleri işaretlenir.

---

## Aşama 11 — NASA POWER saatlik veri indir

### Run

```text
scripts/11_fetch_nasa_power_hourly.py
```

### Input

```text
outputs/10_turkiye_province_candidates_validated.csv
```

### Output

```text
data/turkiye_province_nasa_power_hourly/
data/turkiye_province_nasa_power_hourly_combined.csv
```

Bu script internet ister. 81 il için 2020-2024 saatlik veriyi indirir. İlk denemede hızlı test için `scripts/00_config.py` içinde şunu değiştirebilirsin:

```python
NASA_POWER_MAX_PROVINCES = 5
```

Tam çalışma için:

```python
NASA_POWER_MAX_PROVINCES = 0
```

> Not: Bu aşama veri boyutu ve internet hızına bağlı olarak uzun sürebilir.

---

## Aşama 12 — İl bazlı saatlik feature tablosu oluştur

### Run

```text
scripts/12_build_province_feature_table.py
```

### Input

```text
data/turkiye_province_nasa_power_hourly_combined.csv
```

### Output

```text
outputs/12_turkiye_province_hourly_features.csv
outputs/12_turkiye_province_feature_summary.csv
outputs/12_turkiye_province_feature_quality_report.csv
```

Bu aşamada NASA değişkenleri V3 model feature isimlerine çevrilir:

```text
ALLSKY_SFC_SW_DWN → global_tilted_irradiance (W/m²) proxy
T2M               → temperature_2m (°C)
RH2M              → relative_humidity_2m (%)
CLOUD_AMT         → cloud_cover (%)
WS10M × 3.6       → wind_speed_10m (km/h)
```

---

## Aşama 13 — ANN ile 81 ilin PV performansını tahmin et + applicability-domain hesapla

### Run

```text
scripts/13_train_ann_predict_turkiye_provinces.py
```

### Input

```text
outputs/01_prepared_hourly_dataset.csv
outputs/12_turkiye_province_hourly_features.csv
outputs/12_turkiye_province_feature_summary.csv
```

### Output

```text
outputs/13_turkiye_province_hourly_ann_predictions.csv
outputs/13_turkiye_province_prediction_summary.csv
outputs/13_turkiye_province_applicability_domain.csv
outputs/13_turkiye_province_final_feature_table.csv
outputs/figures/13_turkiye_province_applicability_domain_risk.png
```

Bu aşama çok önemli. Çünkü V3 modelini Türkiye illerine transfer ederken modelin güven alanını da ölçer.

Makalede kullanılacak tablo:

```text
Table: Province-level ANN-predicted PV performance and applicability-domain scores
```

---

## Aşama 14 — Türkiye illerini solar-iklim profiline göre cluster et

### Run

```text
scripts/14_cluster_turkiye_provinces.py
```

### Input

```text
outputs/13_turkiye_province_final_feature_table.csv
```

### Output

```text
outputs/14_turkiye_province_cluster_model_selection.csv
outputs/14_turkiye_province_clusters.csv
outputs/14_turkiye_province_cluster_profiles.csv
outputs/figures/14_turkiye_province_cluster_scatter_map.png
outputs/figures/14_turkiye_province_cluster_pca.png
```

Kümeleme için kullanılan feature'lar:

```text
gti_mean_Wm2
temperature_penalty_over_25C_mean
relative_humidity_mean_percent
cloud_cover_mean_percent
wind_speed_10m_mean_kmh
ann_predicted_pv_performance_mean
seasonal_variability_index
```

Bu aşama makaledeki yeni katkılardan biridir:

```text
cluster-aware PV suitability assessment
```

---

## Aşama 15 — Cluster-wise ve Türkiye geneli MCDM ranking

### Run

```text
scripts/15_clusterwise_mcdm_turkiye_provinces.py
```

### Input

```text
outputs/14_turkiye_province_clusters.csv
outputs/02_ann_xai_feature_weights.csv
```

### Output

```text
outputs/15_turkiye_province_mcdm_climate_decision_matrix.csv
outputs/15_turkiye_province_mcdm_augmented_decision_matrix.csv
outputs/15_turkiye_province_national_rankings_climate_only.csv
outputs/15_turkiye_province_national_rankings_performance_augmented.csv
outputs/15_turkiye_province_clusterwise_rankings_climate_only.csv
outputs/15_turkiye_province_clusterwise_rankings_performance_augmented.csv
outputs/15_turkiye_province_top_candidates_by_cluster.csv
outputs/15_turkiye_province_mcdm_weights.csv
outputs/figures/15_turkiye_province_top20_ranking.png
outputs/figures/15_turkiye_province_top_candidates_by_cluster.png
```

Bu aşamada iki karar matrisi üretilir:

### 1. Climate-only ranking

V3 ile aynı ana meteorolojik kriterlere dayanır:

```text
GTI proxy
sıcaklık cezası
nem
bulutluluk
rüzgâr
```

### 2. Performance-augmented ranking

Buna ek olarak şunları da kullanır:

```text
ANN predicted PV performance
seasonal variability
applicability-domain risk
```

Bu ikinci versiyon Türkiye ili seçimi için daha pratik bir aday tarama sonucudur.

---

# Tek komutla çalıştırma

Internet ve NASA POWER fetch dahil tam V4 için:

```text
scripts/run_all_v4_with_fetch.py
```

NASA verisini zaten indirdiysen ve sadece analizleri çalıştırmak istiyorsan:

```text
scripts/run_all_v4_no_fetch_existing_meteo.py
```

---

# Makale contribution ifadesi

V4 ile çalışma şu contribution'a dönüşür:

```text
A capacity-normalized, leakage-safe, outlier-screened multi-XAI, applicability-domain controlled, cluster-aware hybrid MCDM framework for province-level PV suitability assessment in Türkiye.
```

Türkçe:

```text
Türkiye'de il düzeyinde GES uygunluk değerlendirmesi için kapasite-normalize, veri sızıntısına karşı denetlenmiş, aykırı XAI etkilerinden arındırılmış, applicability-domain kontrollü, cluster-duyarlı hibrit MCDM çerçevesi.
```

---

# Önemli sınırlamalar

1. İl koordinatları il merkezi/province-center proxy olarak alınmıştır. Bu nedenle sonuçlar parsel bazlı değil, makro il taramasıdır.
2. NASA `ALLSKY_SFC_SW_DWN` yatay yüzey radyasyonudur; V3'teki GTI değişkenine proxy olarak eşlenmiştir. Daha kesin panel düzlemi ışınımı için PVGIS veya pvlib ile tilt correction yapılabilir.
3. Gerçek arazi seçimi için eğim, arazi kullanımı, trafo/yol uzaklığı ve korunan alanlar gibi GIS katmanları V5 aşamasına eklenmelidir.
