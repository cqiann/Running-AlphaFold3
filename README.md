# Running AlphaFold3
Create the following .sh file, replace any $ as appropriate (Except for $BIND_PATHS, keep that as $BIND_PATHS). 

When defining `BIND_PATHS`, make sure to include the path that contains your AlphaFold3 input file. For example, if I have the input file in `/scratch/beagle3/christineqian/$some_sub_directory`, I would include the higher level directory, ie. `/scratch/beagle3/christineqian/` in the BIND_PATHS so AlphaFold3 can have access to my input file.

`--json_path` is where you would provide the path of your input file, which will be in JSON format. Say if I had an input file named `42r.1c.json` in `/scratch/beagle3/christineqian/alphafold3/`, then I would specify the field as: `--json_path=/scratch/beagle3/christineqian/alphafold3/42r.1c.json`

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

BIND_PATHS="/software/alphafold3.0-el8-x86_64/databases,/software/alphafold3.0-el8-x86_64/params,/software/alphafold3.0-el8-x86_64/singularity,/home/$YOUR_MIDWAY_USERNAME,/$PATH_TO_A_HIGHER_LEVEL_DIRECTORY_THAT_CONTAINS_THE_INPUT_FILE"

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

Here is an example input JSON file for predicting a complex containing
>MHC|chain A
IKEEHTIIQAEFYLLPDKRGEFMFDFDGDEIFHVDIEKSETIWRLEEFAKFASFEAQGALANIAVDKANLDVMKERSNNTPDANVAPEVTVLSRSPVNLGEPNILICFIDKFSPPVVNVTWLRNGRPVTEGVSETVFLPRDDHLFRKFHYLTFLPSTDDFYDCEVDHWGLEEPLRKHWEFEEKTLLPETKESR

>MHC|chain B
GSGGGGSRPWFLEYCKSECHFYNGTQRVRLLVRYFYNLEENLRFDSDVGEFRAVTELGRPDAENWNSQPEFLEQKRAEVDTVCRHNYEIFDNFLVPRRVEPTVTVYPTKTQPLEHHNLLVCSVSDFYPGNIEVRWFRNGKEEKTGIVSTGLVRNGDWTFQTLVMLETVPQSGEVYTCQVEHPSLTDPVTVEWKAQSTSAQNK

>TCR|chain C
MTGFLKALLLVLCLRPEWIKSQQKTGGQQVKQSSPSLTVQEGGILILNCDYENDMFDYFAWYKKY
PDNSPTLLISVRSNVDKREDGRFTVFLNKSGKHFSLHITASQPEDTAVYLCAAGGFNTGNYKYVF
GAGTRLKVIAHIQNPEPAVYQLKDPRSQDSTLCLFTDFDSQINVPKTMESGTFITDKTVLDMKAMD
SKSNGAIAWSNQTSFTCQDIFKETNATYPSSDVPCDATLTEKSFETDMNLNFQNLSVMGLRILLLK
VAGFNLLMTLRLWSS

>TCR|chain D
MNKWVFCWVTLCLLTVETTHGDGGIITQTPKFLIGQEGQKLTLKCQQNFNHDTMYWYRQDSGKG
LRLIYYSITENDLQKGDLSEGYDASREKKSSFSLTVTSAQKNEMAVFLCASSIGTGGNERLFFGHG
TKLSVLEDLRNVTPPKVSLFEPSKAEIANKQKATLVCLARGFFPDHVELSWWVNGKEVHSGVSTD
PQAYKESNYSYCLSSRLRVSATFWHNPRNHFRCQVQFHGLSEEDKWPEGSPKPVTQNISAEAW
GRADCGITSASYHQGVLSATILYEILLGKATLYAVLVSGLVLMAMVKKKNS

>peptide|chain E
INVELSHLGKKK

with template and MSA:

```
{
  "name": "TCR_complex",
  "sequences": [
    {
      "protein": {
        "id":["A"],
        "sequence": "IKEEHTIIQAEFYLLPDKRGEFMFDFDGDEIFHVDIEKSETIWRLEEFAKFASFEAQGALANIAVDKANLDVMKERSNNTPDANVAPEVTVLSRSPVNLGEPNILICFIDKFSPPVVNVTWLRNGRPVTEGVSETVFLPRDDHLFRKFHYLTFLPSTDDFYDCEVDHWGLEEPLRKHWEFEEKTLLPETKESR",
      }
    },
    {
      "protein": {
        "id":["B"],
        "sequence": "GSGGGGSRPWFLEYCKSECHFYNGTQRVRLLVRYFYNLEENLRFDSDVGEFRAVTELGRPDAENWNSQPEFLEQKRAEVDTVCRHNYEIFDNFLVPRRVEPTVTVYPTKTQPLEHHNLLVCSVSDFYPGNIEVRWFRNGKEEKTGIVSTGLVRNGDWTFQTLVMLETVPQSGEVYTCQVEHPSLTDPVTVEWKAQSTSAQNK",
      }
    },
    {
      "protein": {
        "id":["C"],
        "sequence": "MTGFLKALLLVLCLRPEWIKSQQKTGGQQVKQSSPSLTVQEGGILILNCDYENDMFDYFAWYKKYPDNSPTLLISVRSNVDKREDGRFTVFLNKSGKHFSLHITASQPEDTAVYLCAAGGFNTGNYKYVFGAGTRLKVIAHIQNPEPAVYQLKDPRSQDSTLCLFTDFDSQINVPKTMESGTFITDKTVLDMKAMDSKSNGAIAWSNQTSFTCQDIFKETNATYPSSDVPCDATLTEKSFETDMNLNFQNLSVMGLRILLLKVAGFNLLMTLRLWSS",
      }
    },
    {
      "protein": {
        "id":["D"],
        "sequence": "MNKWVFCWVTLCLLTVETTHGDGGIITQTPKFLIGQEGQKLTLKCQQNFNHDTMYWYRQDSGKGLRLIYYSITENDLQKGDLSEGYDASREKKSSFSLTVTSAQKNEMAVFLCASSIGTGGNERLFFGHGTKLSVLEDLRNVTPPKVSLFEPSKAEIANKQKATLVCLARGFFPDHVELSWWVNGKEVHSGVSTDPQAYKESNYSYCLSSRLRVSATFWHNPRNHFRCQVQFHGLSEEDKWPEGSPKPVTQNISAEAWGRADCGITSASYHQGVLSATILYEILLGKATLYAVLVSGLVLMAMVKKKNS",
      }
    },
    {
      "protein": {
        "id":["E"],
        "sequence": "INVELSHLGKKK",
      }
    },
  ],
  "modelSeeds": [1, 2, 3],
  "dialect": "alphafold3",
  "version": 1
}
```
For `modelSeeds: list[int]:` If you provide n random seeds, you will get n predicted structures, each with the respective random seed. You must provide at least one random seed. So in the example JSON file `modelSeeds: [1, 2, 3]` gives three predicted structures.
