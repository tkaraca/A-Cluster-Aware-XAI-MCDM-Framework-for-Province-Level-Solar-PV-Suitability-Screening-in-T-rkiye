# V3 Final Revision Notes

- Main dataset excludes Kahramanmaraş: `EXCLUDE_CITIES = ["Kahramanmaraş"]`.
- Final ANN-XAI weights no longer use finite-difference or ANN connection weights directly.
- Final weights are `SHAP + Permutation + Mutual Information + Correlation` consensus.
- Main MCDM ensemble is `MARCOS + TOPSIS + EDAS`.
- CoCoSo remains in supplementary robustness outputs.
- New robustness scripts:
  - `07_common_period_sensitivity.py`
  - `08_xai_method_sensitivity.py`
  - `09_repeated_seed_stability.py`
- Final Excel report: `outputs/pv_xai_mcdm_v3_summary_report.xlsx`.
