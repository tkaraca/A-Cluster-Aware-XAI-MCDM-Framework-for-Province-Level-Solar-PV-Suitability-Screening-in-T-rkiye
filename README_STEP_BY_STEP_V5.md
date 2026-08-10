# PV XAI-ANN-MCDM V5 — Performance-Priority Türkiye Province Screening

Bu sürüm V4 sonuç analizinde görülen iki problemi düzeltmek için hazırlanmıştır:

1. Türkiye geneli national ranking tarafında düşük sıcaklık cezası fazla ödüllendiriliyordu.
2. ANN-predicted PV performance ve applicability-domain risk, performance-augmented sıralamada yeterince etkili değildi.

V5 şu revizyonları ekler:

- **Performance-priority MCDM mode**
- **Applicability-domain risk minimum ağırlığı**
- **Temperature penalty ağırlık cap'i**
- **Outlier-screened XAI consensus ağırlığı korunur**
- **CoCoSo ana ensemble dışında robustness olarak kalır**
- **k=2,3,4,6 cluster sensitivity**
- **safe-candidate listesi**: yüksek extrapolation-risk ve yüksek seasonal-variability uyarısı olmayan adaylar

---

## Ana çalıştırma dosyaları

İlk kez NASA POWER verisi indirilecekse:

```text
scripts/run_all_v5_with_fetch.py
```

NASA verisi zaten varsa ve şu dosya mevcutsa:

```text
data/turkiye_province_nasa_power_hourly_combined.csv
```

şunu çalıştır:

```text
scripts/run_all_v5_no_fetch_existing_meteo.py
```

Eski V4 run dosyaları alias olarak bırakıldı. Onları çalıştırırsan V5 scriptlerine yönlenir:

```text
scripts/run_all_v4_with_fetch.py
scripts/run_all_v4_no_fetch_existing_meteo.py
```

---

## Script sırası

Tam V5 pipeline şu sırayla çalışır:

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
11_fetch_nasa_power_hourly.py       # sadece with_fetch scriptinde
12_build_province_feature_table.py
13_train_ann_predict_turkiye_provinces.py
14_cluster_turkiye_provinces.py
15_clusterwise_mcdm_turkiye_provinces.py
16_cluster_k_sensitivity_turkiye_provinces.py
```

Opsiyonel Excel rapor için en son:

```text
06_export_excel_report.py
```

---

## V5 performans-priority MCDM mantığı

V5 national/province screening için yeni mod:

```text
performance_priority
```

Karar kriterleri:

```text
ann_predicted_pv_performance_mean       benefit
gti_mean_Wm2                             benefit
seasonal_variability_index               cost
applicability_domain_risk                cost
temperature_penalty_over_25C_mean        cost
cloud_cover_mean_percent                 cost
relative_humidity_mean_percent           cost
wind_speed_10m_mean_kmh                  benefit
```

Blok ağırlıkları:

```text
Performance block      50%
Risk/stability block   25%
Climate penalty block  25%
```

Bloklar:

```text
Performance block:
- ann_predicted_pv_performance_mean
- gti_mean_Wm2

Risk/stability block:
- seasonal_variability_index
- applicability_domain_risk

Climate penalty block:
- temperature_penalty_over_25C_mean
- cloud_cover_mean_percent
- relative_humidity_mean_percent
- wind_speed_10m_mean_kmh
```

V5'te ayrıca şu guardrail'ler var:

```text
temperature_penalty_over_25C_mean maksimum ağırlık: 0.20
applicability_domain_risk minimum ağırlık: 0.10
```

Bu sayede national ranking düşük sıcaklık cezasına aşırı kaymaz ve modelin extrapolation-risk verdiği iller daha güçlü cezalandırılır.

---

## En önemli V5 output dosyaları

### V3/plant-data model doğrulama çıktıları

```text
outputs/02c_ann_model_metrics.csv
outputs/02c_ann_model_metrics_by_city.csv
outputs/02_ann_xai_feature_weights.csv
outputs/09_repeated_seed_ann_metrics_summary.csv
outputs/09_repeated_seed_xai_weight_stability.csv
```

### Türkiye il feature ve prediction çıktıları

```text
outputs/12_turkiye_province_feature_summary.csv
outputs/13_turkiye_province_prediction_summary.csv
outputs/13_turkiye_province_applicability_domain.csv
outputs/13_turkiye_province_final_feature_table.csv
```

### Cluster çıktıları

```text
outputs/14_turkiye_province_cluster_model_selection.csv
outputs/14_turkiye_province_clusters.csv
outputs/14_turkiye_province_cluster_profiles.csv
```

### V5 national ve clusterwise MCDM çıktıları

```text
outputs/15_turkiye_province_mcdm_performance_priority_decision_matrix.csv
outputs/15_turkiye_province_national_rankings_performance_priority.csv
outputs/15_turkiye_province_clusterwise_rankings_performance_priority.csv
outputs/15_turkiye_province_national_rankings_performance_priority_with_cocoso_robustness.csv
outputs/15_turkiye_province_clusterwise_rankings_performance_priority_with_cocoso_robustness.csv
outputs/15_turkiye_province_top_candidates_performance_priority.csv
outputs/15_turkiye_province_top_candidates_performance_priority_safe.csv
outputs/15_turkiye_province_mcdm_weights.csv
```

### k-sensitivity çıktıları

```text
outputs/16_cluster_k_sensitivity_model_selection.csv
outputs/16_cluster_k_sensitivity_assignments.csv
outputs/16_cluster_k_sensitivity_profiles.csv
outputs/16_cluster_k_sensitivity_clusterwise_rankings.csv
outputs/16_cluster_k_sensitivity_top_candidates.csv
```

### Figürler

```text
outputs/figures/15_v5_performance_priority_top20.png
outputs/figures/15_v5_performance_priority_top_candidates_by_cluster.png
outputs/figures/15_v5_performance_priority_risk_vs_rank.png
outputs/figures/16_cluster_k_sensitivity_top_candidates_heatmap.png
```

---

## Makale için önerilen sonuç kullanımı

Ana sonuç olarak şunu kullan:

```text
outputs/15_turkiye_province_national_rankings_performance_priority.csv
```

Cluster bazlı yorum için:

```text
outputs/15_turkiye_province_clusterwise_rankings_performance_priority.csv
outputs/15_turkiye_province_top_candidates_performance_priority.csv
```

CoCoSo kontrolü için:

```text
outputs/15_turkiye_province_national_rankings_performance_priority_with_cocoso_robustness.csv
```

k değeri duyarlılığı için:

```text
outputs/16_cluster_k_sensitivity_top_candidates.csv
```

Safe aday listesi için:

```text
outputs/15_turkiye_province_top_candidates_performance_priority_safe.csv
```

Bu safe liste, yüksek applicability-domain risk veya yüksek seasonal variability uyarısı olmayan adayları öne çıkarır.

---

## Dikkat edilmesi gereken yorum

V5 hâlâ province-level screening yapar. Bu, gerçek arazi/site-level GES yer seçimi değildir. İl düzeyi makro taramadır. Gerçek lokasyon seçimi için ileride şu GIS kriterleri eklenmelidir:

```text
eğim
bakı
arazi kullanımı
şebeke/trafo uzaklığı
yol uzaklığı
korunan alan/tarım/orman kısıtları
kar/flood/deprem riski
```

