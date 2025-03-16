# Running-AlphaFold3
Create the following .sh file, replace $JOBNAME, $USERNAME, $PATH_TO_INPUT_FILE with the job name you want, your Midway username, and the path to your AlphaFold3 input file :

```
#!/bin/bash
#SBATCH --job-name=$JOBNAME
#SBATCH --account=beagle3-exusers
#SBATCH --partition=beagle3
#SBATCH --time=10:00:00
#SBATCH --cpus-per-task=16
#SBATCH --gres=gpu:2
#SBATCH --constraint=a100

module load singularity

BIND_PATHS="/software/alphafold3.0-el8-x86_64/databases,/software/alphafold3.0-el8-x86_64/params,/software/alphafold3.0-el8-x86_64/singularity,/home/$USERNAME,/scratch/beagle3/$USERNAME"

singularity exec --nv \
  -B "$BIND_PATHS" \
  --env CUDA_VISIBLE_DEVICES=0,1,NVIDIA_VISIBLE_DEVICES=0,1 \
  /software/alphafold3.0-el8-x86_64/alphafold3.sif \
  python /app/alphafold/run_alphafold.py \
  --json_path=$PATH_TO_YOUR_INPUT_FILE \
  --db_dir=/software/alphafold3.0-el8-x86_64/databases \
  --output_dir=/scratch/beagle3/$USERNAME/alphafold3_output \
  --model_dir=/software/alphafold3.0-el8-x86_64/params \
  --flash_attention_implementation=triton \
  --run_data_pipeline=True \
  --run_inference=True \
  --jackhmmer_n_cpu=8 \
  --nhmmer_n_cpu=8
```
