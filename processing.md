Processing genome data from the Broad
```
#convert to fastq for assembly with samtools v1.7
conda activate samtools 

#for one file
samtools fastq -1 test1.fq -2 test2.fq  /hpcstor6/scratch01/p/patrick.kearns/genomes/2023-08-15_256samples_33K.bam

conda activate samtools
#a for loop for all files
#assumes you're in the folder where the genomes are and you have a output folder (mkdir fastq) called fastq
for i in *.bam; do samtools fastq -1 fastq/$i-1.fq -2 fastq/$i-2.fq $i; done

#assemble with SPADES, see sh file as its done on a per-genome basis

#rename the bins by folder name, assumes your in the folder where all the genomes are located
## Specify the extension 
target_extension=".fasta"

# Function to process files
process_file() {
    local filepath="$1"
    local foldername="$2"
    local filename=$(basename "$filepath")
    
    # Append folder name to the file
    new_filename="${foldername}_${filename}"
    
    # Rename the file
    mv "$filepath" "$(dirname $filepath)/$new_filename"
}

# Main loop to find and process files
find . -type f -name "*$target_extension" | while read filepath; do
    foldername=$(basename "$(dirname "$filepath")")
    process_file "$filepath" "$foldername"
done

#######find and copy all files into a new folder, assumes your in the folder where all the genomes are located
#tell it the extension for each genome
mkdir aggregated_genomes
target_extension="_contigs.fasta"

# Specify the destination folder where the files will be copied
destination_folder="./aggregated_genomes"

# Create the destination folder if it doesn't exist
mkdir -p "$destination_folder"

# Main loop to find and copy files
find . -type f -name "*$target_extension" -exec cp {} "$destination_folder" \;

#remove extra files
rm *final*

#run checkM on the genomes
cd /hpcstor6/scratch01/p/patrick.kearns/genomes
mkdir checkM_results
checkm lineage_wf ./aggregated_genomes ./checkM_results -x fasta
```
