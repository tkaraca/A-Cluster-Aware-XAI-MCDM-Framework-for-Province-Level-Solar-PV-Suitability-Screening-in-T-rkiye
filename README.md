# PV XAI-ANN-MCDM V6 — Solar proxy robustness + supplementary k=6 maps

Bu sürüm V5 üzerine iki yayın-savunması revizyonu ekler:

1. **NASA radyasyon değişkeni doğru adlandırıldı.** Türkiye 81 il tarafında NASA POWER `ALLSKY_SFC_SW_DWN` değişkeni artık açıkça `solar_radiation_proxy_Wm2` olarak tutulur. Eski `global_tilted_irradiance (W/m²)` adı sadece V3/V5 kod uyumluluğu için alias olarak kalır. Makalede bu değişken **horizontal surface shortwave solar radiation proxy** olarak yazılmalıdır; exact GTI/POA olarak yazılmamalıdır.

2. **Solar radiation proxy robustness analizi eklendi.** `17_solar_radiation_proxy_robustness.py`, santral tarafında `shortwave_radiation (W/m²)` ile ANN eğitir ve Türkiye il tarafında NASA `solar_radiation_proxy_Wm2` ile tahmin/ranking üretir. Bu analiz, GTI-vs-horizontal proxy uyumsuzluğu için supplementary robustness kontrolüdür.

3. **Ek figürler eklendi.** Cluster centroid haritası, V5 top aday haritası, k=6 supplementary cluster haritası ve main-vs-solar-proxy rank karşılaştırma figürü üretilecek.

---

## İlk tam çalışma

NASA POWER verisini ilk kez indireceksen çalıştır:

```text
scripts/run_all_v6_with_fetch.py
```

Bu script sırasıyla şu aşamaları çalıştırır:

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
10_prepare_turkiye_province_candidates.py
11_fetch_nasa_power_hourly.py
12_build_province_feature_table.py
13_train_ann_predict_turkiye_provinces.py
14_cluster_turkiye_provinces.py
15_clusterwise_mcdm_turkiye_provinces.py
16_cluster_k_sensitivity_turkiye_provinces.py
17_solar_radiation_proxy_robustness.py
```

## NASA verisi zaten varsa

Şu dosya zaten varsa:

```text
data/turkiye_province_nasa_power_hourly_combined.csv
```

şunu çalıştır:

```text
scripts/run_all_v6_no_fetch_existing_meteo.py
```

Bu script NASA veri çekme adımı olan `11_fetch_nasa_power_hourly.py` dosyasını atlar.

---

## V6 yeni çıktı dosyaları

### NASA radiation proxy audit

```text
outputs/12_turkiye_province_radiation_proxy_audit.csv
```

Bu dosya makalede kullanılacak isimlendirmeyi netleştirir:

```text
province_feature_column = solar_radiation_proxy_Wm2
source_parameter = ALLSKY_SFC_SW_DWN
publication_label_recommended = horizontal surface shortwave solar radiation proxy
legacy_compatibility_alias = global_tilted_irradiance (W/m²)
```

### Solar proxy robustness

```text
outputs/17_solar_proxy_ann_model_metrics.csv
outputs/17_solar_proxy_prediction_summary.csv
outputs/17_solar_proxy_applicability_domain.csv
outputs/17_solar_proxy_national_rankings_performance_priority.csv
outputs/17_solar_proxy_clusterwise_rankings_performance_priority.csv
outputs/17_solar_proxy_top_candidates_performance_priority.csv
outputs/17_solar_proxy_robustness_summary.csv
```

`17_solar_proxy_robustness_summary.csv` en kritik dosyadır. Burada şunlara bak:

```text
top10_overlap_count
top10_jaccard
spearman_rank_corr_all_provinces
main_top10
solar_proxy_top10
```

Ana V5 top aday portföyü ile solar-proxy robustness portföyü büyük ölçüde örtüşüyorsa, radyasyon proxy sınırlılığına karşı daha güçlü savunma elde edilir.

---

## V6 yeni figürleri

```text
outputs/figures/14_turkiye_province_cluster_coordinate_map_labeled.png
outputs/figures/15_v5_performance_priority_top_candidates_coordinate_map.png
outputs/figures/16_k6_cluster_coordinate_map.png
outputs/figures/17_solar_proxy_top20_ranking.png
outputs/figures/17_main_vs_solar_proxy_rank_comparison.png
```

Not: Bu haritalar polygon tabanlı GIS haritası değil, il merkezi/centroid koordinatlarıyla üretilen screening haritalarıdır. Makalede bu açıkça yazılmalıdır.

---

## Excel raporu

Tüm ana CSV tablolarını tek Excel'e almak için en son çalıştır:

```text
scripts/06_export_excel_report.py
```

Üretilen dosya:

```text
outputs/pv_xai_mcdm_v6_summary_report.xlsx
```

---

## Makalede kullanılması önerilen ifade

Türkiye il düzeyi radyasyon değişkeni için:

```text
NASA POWER ALLSKY_SFC_SW_DWN was used as a horizontal surface shortwave solar radiation proxy for province-level preliminary screening. The original plant-level training data included global tilted irradiance; therefore, an additional shortwave-radiation proxy robustness analysis was conducted to assess the sensitivity of the candidate province portfolio to the solar-input definition.
```

Türkçesi:

```text
Türkiye il düzeyi ön taramada NASA POWER ALLSKY_SFC_SW_DWN değişkeni yatay yüzey kısa dalga güneş radyasyonu proxy'si olarak kullanılmıştır. Orijinal santral eğitim verisinde global tilted irradiance bulunduğundan, güneş girdisi tanımına duyarlılığı değerlendirmek amacıyla shortwave-radiation proxy tabanlı ek bir sağlamlık analizi yapılmıştır.
```
