# CoreCheck
a coreference-aware dual-granularity fact verification framework for complex claims.

# Gold evidence

cd CorefCheck
mkdir -p output/normalization output/refinement output/decomposition output/verification
python Gold_evidence/S1_coreference_normalization.py \
  --model_dir Qwen2.5-32B-Instruct \
  --input_path data/hover/hover_2_hops.json \
  --output_path output/normalization/hover_2_hops.json \
  --device_map auto \
  --dtype float16 \
  --max_new_tokens 512 \
  --max_input_length 30000 \
  --temperature 0 \
  --save_every 20

python Gold_evidence/S2_coreference_refinement.py \
  --input_path output/normalization/hover_2_hops.json \
  --output_path output/refinement/hover_2_hops.json \
  --model_dir Qwen2.5-32B-Instruct \
  --device_map auto \
  --dtype float16 \
  --max_new_tokens 512 \
  --max_input_length 30000 \
  --temperature 0 \
  --max_rewrite_rounds 2 \
  --save_every 20

python Gold_evidence/S3_claim_decomposition.py \
  --model_dir Qwen2.5-32B-Instruct \
  --input_path output/refinement/hover_2_hops.json \
  --output_path output/decomposition/hover_2_hops.json \
  --device_map auto \
  --dtype float16 \
  --max_new_tokens 512 \
  --validation_max_new_tokens 512 \
  --max_input_length 30000 \
  --max_attempts 3 \
  --min_content_coverage 0.72 \
  --max_subclaims 20 \
  --save_every 20 \
  --resume

python Gold_evidence/S4_claim_verification.py \
  --input_path output/decomposition/hover_2_hops.json \
  --output_path output/verification/hover_2_hops.json \
  --model deepseek-r1:14b \
  --refute_threshold 0.50 \
  --strong_refute_threshold 0.85 \
  --save_every 10 \
  --resume

python Gold_evidence/S5_risk_calibration.py \
  --input_path output/verification/hover_2_hops.json \
  --output_dir hover_test_results \
  --apply_thresholds thresholds/hover_best_thresholds.json

# Open evidence
Open-evidence retrieval follows the same approach as BiDeV.
After obtaining the corresponding open-retrieved evidence:
python S1_open_evidence_retrieval.py \
  --input_file "${INPUT_FILE}" \
  --output_file "${RETRIEVED_FILE}" \
  --index_path "${INDEX_PATH}" \
  --top_k 10

python S2_evidence_relevance_filtering.py \
  --input_file "${RETRIEVED_FILE}" \
  --output_file "${FILTERED_FILE}" \
  --model_path "${FILTER_MODEL}" \
  --gpus "${GPU_IDS}"

python open_claim_verification.py \
  --input_path "${FILTERED_FILE}" \
  --output_path "${VERIFICATION_FILE}" \
  --metrics_path "${METRICS_FILE}" \
  --model deepseek-r1:14b \
  --ollama_url "${OLLAMA_URL}" \
  --save_every 10 \
  --resume

