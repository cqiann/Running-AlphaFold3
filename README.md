# Running-AlphaFold3
Example .sh file:

```
#!/bin/bash
#SBATCH --job-name=REPLACE WITH JOB NAME
#SBATCH --account=beagle3-exusers
#SBATCH --partition=beagle3
#SBATCH --time=10:00:00
#SBATCH --cpus-per-task=16
#SBATCH --gres=gpu:2
#SBATCH --constraint=a100

module load singularity

BIND_PATHS="/software/alphafold3.0-el8-x86_64/databases,/software/alphafold3.0-el8-x86_64/params,/software/alphafold3.0-el8-x86_64/singularity,/home/USERNAME,/scratch/beagle3/christineqian"

singularity exec --nv \
  -B "$BIND_PATHS" \
  --env CUDA_VISIBLE_DEVICES=0,1,NVIDIA_VISIBLE_DEVICES=0,1 \
  /software/alphafold3.0-el8-x86_64/alphafold3.sif \
  python /app/alphafold/run_alphafold.py \
  --json_path=/scratch/beagle3/christineqian/alphafold3/42r.3c_premsa.json \
  --db_dir=/software/alphafold3.0-el8-x86_64/databases \
  --output_dir=/scratch/beagle3/christineqian/alphafold3_output \
  --model_dir=/software/alphafold3.0-el8-x86_64/params \
  --flash_attention_implementation=triton \
  --run_data_pipeline=True \
  --run_inference=True \
  --jackhmmer_n_cpu=8 \
  --nhmmer_n_cpu=8
```
