# Running AlphaFold3
Create the following .sh file, replace $USERNAME, $PATH_TO_INPUT_FILE with your Midway username and the path to your AlphaFold3 input file. 
When defining `BIND_PATHS`, make sure to include the path that contains your AlphaFold3 input file. For example, if I have the input file in `/scratch/beagle3/christineqian/$some_sub_directory`, I would include `/scratch/beagle3/christineqian/` in the BIND_PATHS so AlphaFold3 can have access to my input file.

```
#!/bin/bash
#SBATCH --job-name=alphafold3
#SBATCH --account=beagle3-exusers
#SBATCH --partition=beagle3
#SBATCH --time=10:00:00
#SBATCH --cpus-per-task=16
#SBATCH --gres=gpu:2
#SBATCH --constraint=a100

module load singularity

BIND_PATHS="/software/alphafold3.0-el8-x86_64/databases,/software/alphafold3.0-el8-x86_64/params,/software/alphafold3.0-el8-x86_64/singularity,/home/$USERNAME,/$PATH_THAT_CONTAINS_YOUR_INPUT_FILE"

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
The input file should be a JSON file with the following top-level structure:
```
{
  "name": "Job name goes here",
  "modelSeeds": [1, 2],  # At least one seed required.
  "sequences": [
    {"protein": {...}},
    {"rna": {...}},
    {"dna": {...}},
    {"ligand": {...}}
  ],
  "bondedAtomPairs": [...],  # Optional
  "userCCD": "...",  # Optional
  "dialect": "alphafold3",  # Required
  "version": 2  # Required
}
```
Fields for Protein
```
{
  "protein": {
    "id": "A",
    "sequence": "PVLSCGEWQL",
    "modifications": [
      {"ptmType": "HY3", "ptmPosition": 1},
      {"ptmType": "P1L", "ptmPosition": 5}
    ],
    "unpairedMsa": ...,  # Mutually exclusive with unpairedMsaPath.
    "unpairedMsaPath": ...,  # Mutually exclusive with unpairedMsa.
    "pairedMsa": ...,  # Mutually exclusive with pairedMsaPath.
    "pairedMsaPath": ...,  # Mutually exclusive with pairedMsa.
    "templates": [...]
  }
}
```
The fields specify the following:

* id: str | list[str]: An uppercase letter or multiple letters specifying the unique IDs for each copy of this protein chain. The IDs are then also used in the output mmCIF file. Specifying a list of IDs (e.g. ["A", "B", "C"]) implies a homomeric chain with multiple copies.
* sequence: str: The amino-acid sequence, specified as a string that uses the 1-letter standard amino acid codes.
* modifications: list[ProteinModification]: An optional list of post-translational modifications. Each modification is specified using its CCD code and 1-based residue position. In the example above, we see that the first residue won't be a proline (P) but instead HY3.
* unpairedMsa: str: An optional multiple sequence alignment for this chain. This is specified using the A3M format (equivalent to the FASTA format, but also allows gaps denoted by the hyphen - character). 
* unpairedMsaPath: str: An optional path to a file that contains the multiple sequence alignment for this chain instead of providing it inline using the unpairedMsa field. The path can be either absolute, or relative to the input JSON path. The file must be in the A3M format, and could be either plain text, or compressed using gzip, xz, or zstd.
* pairedMsa: str: The AlphaFold3 Documentation recommend not using this optional field and using the unpairedMsa for the purposes of pairing.
* pairedMsaPath: str: An optional path to a file that contains the multiple sequence alignment for this chain instead of providing it inline using the pairedMsa field. The path can be either absolute, or relative to the input JSON path. The file must be in the A3M format, and could be either plain text, or compressed using gzip, xz, or zstd.
* templates: list[Template]: An optional list of structural templates.

An example JSON file for predicting a 42-residue ABeta dimer with MSA and with template:
```
{
  "name": "42r.2c_notemp",
  "sequences": [
    {
      "protein": {
        "id":["A", "B"],
        "sequence": "DAEFRHDSGYEVHHQKLVFFAEDVGSNKGAIIGLMVGGVVIA",
      }
    }
  ],
  "modelSeeds": [1, 2, 3],
  "dialect": "alphafold3",
  "version": 2
}
```

