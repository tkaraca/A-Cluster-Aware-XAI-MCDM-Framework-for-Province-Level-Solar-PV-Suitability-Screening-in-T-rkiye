# PV XAI-ANN-MCDM Pipeline V3 — Final Revision

Bu sürüm, Kahramanmaraş hariç 4 şehirli ana analiz için hazırlanmıştır. Kahramanmaraş ana analizden çıkarıldı çünkü önceki koşuda kapasiteye normalize edilmiş üretim değerleri fiziksel tutarlılık açısından şüpheliydi.

## V3'te gelen ana revizyonlar

1. **Ana analiz Kahramanmaraş hariçtir.**
   - `scripts/00_config.py` içinde:
   - `EXCLUDE_CITIES = ["Kahramanmaraş"]`

2. **Final XAI ağırlığı revize edildi.**
   - Eski sürümde finite-difference ve ANN connection weights final ensemble içinde rüzgârı gereğinden fazla yükseltebiliyordu.
   - V3 final ağırlığı artık şu kaynaklardan gelir:
     - SHAP
     - Permutation importance
     - Mutual information
     - Correlation
   - Finite-difference ve ANN connection weights sadece diagnostic/supplementary olarak raporlanır.

3. **Ana MCDM ensemble revize edildi.**
   - Ana sıralama: `MARCOS + TOPSIS + EDAS`
   - CoCoSo: robustness/supplementary olarak ayrı raporlanır.

4. **Ek robustness scriptleri eklendi.**
   - `07_common_period_sensitivity.py`
   - `08_xai_method_sensitivity.py`
   - `09_repeated_seed_stability.py`

5. **Excel raporu V3 çıktılarını kapsayacak şekilde güncellendi.**
   - `outputs/pv_xai_mcdm_v3_summary_report.xlsx`

---

## Scriptleri çalıştırma sırası

Scriptleri tek tek Run etmek için önerilen sıra:

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
06_export_excel_report.py
```

Tek seferde çalıştırmak istersen:

```text
run_all.py
```

---

## Aşama 1 — Veri hazırlama

### Script

```text
scripts/01_prepare_dataset.py
```

### Kullandığı dosyalar

```text
data/five_cities_clean_day_combined_all_columns.csv
inputs/city_coordinates.csv
inputs/installed_capacities_kWp.csv
```

### Yaptığı işlem

- Kahramanmaraş'ı veri setinden çıkarır.
- Koordinatları ekler.
- Kurulu güç ile normalize target üretir.
- `target_for_ann = target_hourly_energy_kWh_sum_available / installed_capacity_kWp`
- `temperature_penalty_over_25C` değişkenini üretir.

### Ürettiği dosyalar

```text
outputs/01_prepared_hourly_dataset.csv
outputs/01_city_summary_metrics.csv
outputs/01_data_quality_report.csv
```

### Makale tablosu

- Table: Dataset and site characteristics
- Table: Data quality and descriptive statistics

---

## Aşama 2A — Data leakage audit

### Script

```text
scripts/02a_leakage_audit.py
```

### Kullandığı dosya

```text
outputs/01_prepared_hourly_dataset.csv
```

### Ürettiği dosyalar

```text
outputs/02a_leakage_audit_report.csv
outputs/02a_train_validation_test_time_ranges_by_city.csv
outputs/02a_feature_screening_report.csv
outputs/figures/02a_train_validation_test_timeline_by_city.png
outputs/figures/02a_feature_target_correlation_heatmap.png
```

### Makale kullanımı

- Table: Data leakage audit results
- Figure: Temporal train-validation-test split by city
- Figure: Feature-target correlation heatmap

---

## Aşama 2B — Baseline model karşılaştırması

### Script

```text
scripts/02b_train_baseline_models.py
```

### Kullandığı dosya

```text
outputs/01_prepared_hourly_dataset.csv
```

### Ürettiği dosyalar

```text
outputs/02b_model_comparison_metrics.csv
outputs/02b_model_comparison_metrics_by_city.csv
outputs/02b_leave_one_city_out_metrics.csv
outputs/figures/02b_model_comparison_test_rmse.png
outputs/figures/02b_model_comparison_test_r2.png
outputs/figures/02b_leave_one_city_out_randomforest_rmse.png
```

### Makale kullanımı

- Table: Baseline regression model comparison
- Table: City-wise baseline model performance
- Table: Leave-one-city-out spatial validation
- Figure: Baseline RMSE comparison
- Figure: Baseline R² comparison

---

## Aşama 2C — ANN eğitimi ve V3 XAI ağırlığı

### Script

```text
scripts/02c_train_ann_extract_xai_weights.py
```

### Kullandığı dosya

```text
outputs/01_prepared_hourly_dataset.csv
```

### Feature set

```text
global_tilted_irradiance (W/m²)
temperature_2m (°C)
relative_humidity_2m (%)
cloud_cover (%)
wind_speed_10m (km/h)
```

### Target

```text
target_for_ann
```

### Final V3 XAI ağırlığı

Final dosya:

```text
outputs/02_ann_xai_feature_weights.csv
```

Bu dosya şu kaynakların outlier-screened consensus ortalamasından oluşur:

```text
SHAP + Permutation Importance + Mutual Information + Correlation
```

Diagnostic olarak ayrıca şunlar raporlanır:

```text
ANN connection weights
Finite-difference sensitivity
```

### Ürettiği performans dosyaları

```text
outputs/02c_ann_model_metrics.csv
outputs/02c_ann_model_metrics_by_city.csv
outputs/02c_ann_test_predictions.csv
outputs/02c_ann_high_performance_classification_metrics.csv
```

### Ürettiği XAI dosyaları

```text
outputs/02c_xai_weight_comparison.csv
outputs/02c_outlier_screened_consensus_xai_weights.csv
outputs/02_ann_xai_feature_weights.csv
outputs/02c_shap_feature_weights.csv
outputs/02c_permutation_feature_weights.csv
outputs/02c_mutual_info_feature_weights.csv
outputs/02c_correlation_feature_weights.csv
outputs/02c_ann_connection_weights.csv
outputs/02c_finite_difference_sensitivity_weights.csv
```

### Ürettiği figürler

```text
outputs/figures/02c_ann_observed_vs_predicted.png
outputs/figures/02c_ann_residual_distribution.png
outputs/figures/02c_xai_weight_comparison.png
outputs/figures/02c_ann_test_rmse_by_city.png
outputs/figures/02c_high_performance_confusion_matrix.png
```

### Makale kullanımı

- Table: ANN train-validation-test performance
- Table: City-wise ANN test performance
- Table: XAI criterion-weight comparison
- Table: Final outlier-screened ANN-XAI criterion weights
- Figure: Observed vs predicted
- Figure: Residual distribution
- Figure: XAI weight comparison

---

## Aşama 3 — MCDM karar matrisi ve hibrit ağırlık

### Script

```text
scripts/03_build_decision_matrix.py
```

### Kullandığı dosyalar

```text
outputs/01_prepared_hourly_dataset.csv
outputs/02_ann_xai_feature_weights.csv
```

### Ürettiği dosyalar

```text
outputs/03_mcdm_decision_matrix.csv
outputs/03_merec_objective_weights.csv
outputs/03_hybrid_weights_alpha_0_50.csv
outputs/03_criterion_directions.csv
```

### Hibrit ağırlık

```text
w_final = alpha * w_XAI + (1 - alpha) * w_MEREC
alpha = 0.50
```

### Makale kullanımı

- Table: MCDM decision matrix
- Table: MEREC objective weights
- Table: Hybrid XAI-MEREC weights

---

## Aşama 4 — Ana MCDM sıralaması

### Script

```text
scripts/04_run_mcdm_ranking.py
```

### Ana yöntemler

```text
MARCOS + TOPSIS + EDAS
```

### Supplementary robustness

```text
MARCOS + TOPSIS + EDAS + CoCoSo
```

### Ürettiği dosyalar

```text
outputs/04_mcdm_method_scores_alpha_0_50.csv
outputs/04_mcdm_rankings_alpha_0_50.csv
outputs/04_mcdm_rankings_alpha_0_50_with_cocoso_robustness.csv
```

### Makale kullanımı

- Table: Main MCDM ranking
- Table: Supplementary CoCoSo robustness ranking

---

## Aşama 5 — Alpha sensitivity

### Script

```text
scripts/05_sensitivity_analysis.py
```

### Alpha senaryoları

```text
0.00, 0.25, 0.50, 0.75, 1.00
```

### Ürettiği dosyalar

```text
outputs/05_sensitivity_alpha_rankings.csv
outputs/05_sensitivity_alpha_rankings_with_cocoso_robustness.csv
outputs/05_rank_stability_summary.csv
outputs/05_sensitivity_hybrid_weights_by_alpha.csv
outputs/05_sensitivity_method_scores_by_alpha.csv
outputs/figures/05_mcdm_rank_sensitivity_heatmap.png
outputs/figures/05_mcdm_rank_sensitivity_heatmap_with_cocoso.png
```

### Makale kullanımı

- Table: Alpha sensitivity rankings
- Table: Rank stability summary
- Figure: MCDM rank sensitivity heatmap

---

## Aşama 7 — Common-period sensitivity

### Script

```text
scripts/07_common_period_sensitivity.py
```

### Amacı

Şehirlerin farklı tarih aralıklarına sahip olması sıralamayı etkiliyor mu diye kontrol eder. Tüm şehirlerin ortak zaman penceresini bulur ve MCDM karar matrisini bu ortak dönem için tekrar hesaplar.

### Ürettiği dosyalar

```text
outputs/07_common_period_window.csv
outputs/07_common_period_decision_matrix.csv
outputs/07_common_period_merec_weights.csv
outputs/07_common_period_hybrid_weights_alpha_0_50.csv
outputs/07_common_period_method_scores_alpha_0_50.csv
outputs/07_common_period_rankings_alpha_0_50.csv
outputs/07_common_period_rankings_alpha_0_50_with_cocoso_robustness.csv
outputs/figures/07_common_period_rank_comparison.png
```

### Makale kullanımı

- Table: Common-period sensitivity ranking
- Figure: Full-period vs common-period ranking comparison

---

## Aşama 8 — XAI method sensitivity

### Script

```text
scripts/08_xai_method_sensitivity.py
```

### Amacı

Nihai sıralama hangi XAI ağırlık yöntemine duyarlı mı diye kontrol eder.

### Karşılaştırılan XAI yöntemleri

```text
final_outlier_screened_consensus
shap_only
permutation_only
mutual_information_only
correlation_only
diagnostic_ann_connection_only
diagnostic_finite_difference_only
```

### Ürettiği dosyalar

```text
outputs/08_xai_method_sensitivity_rankings.csv
outputs/08_xai_method_sensitivity_weights.csv
outputs/08_xai_method_sensitivity_method_scores.csv
outputs/figures/08_xai_method_sensitivity_heatmap.png
```

### Makale kullanımı

- Table: Ranking sensitivity to XAI weighting method
- Figure: XAI method sensitivity heatmap

---

## Aşama 9 — Repeated-seed stability

### Script

```text
scripts/09_repeated_seed_stability.py
```

### Amacı

ANN eğitimi ve permutation importance sonuçları random seed değişiminden çok etkileniyor mu diye kontrol eder.

### Ürettiği dosyalar

```text
outputs/09_repeated_seed_ann_metrics.csv
outputs/09_repeated_seed_ann_metrics_summary.csv
outputs/09_repeated_seed_permutation_weights.csv
outputs/09_repeated_seed_xai_weight_stability.csv
outputs/figures/09_repeated_seed_ann_metric_stability.png
outputs/figures/09_repeated_seed_xai_weight_stability.png
```

### Makale kullanımı

- Table: Repeated-seed ANN metric stability
- Table: Repeated-seed permutation importance stability
- Figure: ANN metric stability across seeds
- Figure: XAI weight stability across seeds

---

## Aşama 6 — Excel raporu

### Script

```text
scripts/06_export_excel_report.py
```

### Ürettiği dosya

```text
outputs/pv_xai_mcdm_v3_summary_report.xlsx
```

Bu dosya ana CSV tablolarını tek Excel içinde toplar.

---

## Makalede kullanılabilecek ana contribution cümlesi

```text
This study proposes a capacity-normalized, leakage-safe, outlier-screened multi-XAI and hybrid-MCDM framework for PV site suitability assessment using real operational PV generation and meteorological data.
```

Türkçe:

```text
Bu çalışma, gerçek PV üretim ve meteoroloji verilerini kullanarak kapasiteye göre normalize edilmiş, veri sızıntısına karşı denetlenmiş, outlier-screened çoklu-XAI ve hibrit-MCDM tabanlı bir GES lokasyon uygunluk değerlendirme çerçevesi önermektedir.
```
