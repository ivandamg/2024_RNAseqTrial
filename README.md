# 2024_RNAseqTrial
Summary statistics of RNAseq trial.  Look at coverage of plant-bacteria, completeness, etc


1. Clean reads with fastp

   
         cd /data/users/imateusgonzalez/2024_RNAseqTrial/01_RawData

         mkdir ../02_TrimmedData/


         for FILE in $(ls *1.fastq.gz); do echo $FILE; sbatch --partition=pshort_el8 --job-name=$(echo $FILE | cut -d'_' -f1,2)fastp --time=0-02:00:00 --mem-per-cpu=24G --ntasks=1 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1,2)_fastp.out --error=$(echo $FILE | cut -d'_' -f1,2)_fastp.error --mail-type=END,FAIL --wrap " cd /data/users/imateusgonzalez/2024_RNAseqTrial/01_RawData ; module load FastQC; ~/00_Software/fastp --in1 $FILE --in2 $(echo $FILE | cut -d'_' -f1,2)_R2.fastq.gz --out1 ../02_TrimmedData/$(echo $FILE | cut -d'_' -f1,2)_1_trimmed.fastq.gz --out2 ../02_TrimmedData/$(echo $FILE | cut -d'_' -f1,2)_2_trimmed.fastq.gz -h ../02_TrimmedData/$(echo $FILE | cut -d',' -f1,2)_fastp.html --thread 4; fastqc -t 4 ../02_TrimmedData/$(echo $FILE | cut -d'_' -f1,2)_1_trimmed.fastq.gz; fastqc -t 4 ../02_TrimmedData/$(echo $FILE | cut -d'_' -f1,2)_2_trimmed.fastq.gz"; sleep  1; done

3. Index reference Medicago

       sbatch --partition=pshort_el8 --job-name=StarIndex --time=0-01:00:00 --mem-per-cpu=64G --ntasks=1 --cpus-per-task=1 --output=StarIndex.out --error=StarIndex.error --mail-type=END,FAIL --wrap "cd /data/users/imateusgonzalez/2024_RNAseqTrial/00_ReferenceGenomes; module load STAR/2.7.10a_alpha_220601-GCC-10.3.0; STAR --runThreadN 1 --runMode genomeGenerate --genomeDir /data/users/imateusgonzalez/2024_RNAseqTrial/00_ReferenceGenomes --genomeFastaFiles GCF_003473485.1_MtrunA17r5.0-ANR_genomic.fna --sjdbGTFfile GCF_003473485.1_MtrunA17r5.0-ANR_genomic.gff --sjdbOverhang 99 --genomeSAindexNbases 10"


4. Map reads to medicago

          for FILE in $(ls *_1_trimmed.fastq.gz ); do echo $FILE; sbatch --partition=pibu_el8 --job-name=$(echo $FILE | cut -d'_' -f1,2)_1STAR --time=0-04:00:00 --mem-per-cpu=128G --ntasks=8 --cpus-per-task=1 --output=$(echo $FILE | cut -d'_' -f1,2)_STAR.out --error=$(echo $FILE | cut -d'_' -f1,2)_STAR.error --mail-type=END,FAIL --wrap "module load STAR/2.7.10a_alpha_220601-GCC-10.3.0; cd /data/users/imateusgonzalez/2024_RNAseqTrial/02_TrimmedData; STAR --runThreadN 8 --genomeDir /data/users/imateusgonzalez/2024_RNAseqTrial/00_ReferenceGenomes --readFilesIn $FILE $(echo $FILE | cut -d'_' -f1,2)_2_trimmed.fastq.gz --readFilesCommand zcat --outFileNamePrefix $(echo $FILE | cut -d'_' -f1,2)_MappedMedicago --outSAMtype BAM SortedByCoordinate --outReadsUnmapped Fastx --limitBAMsortRAM 1065539232; mv Unmapped.out.mate1 $(echo $FILE | cut -d'_' -f1,2)_UnmappedMedicago_R1.fastq; mv Unmapped.out.mate1 $(echo $FILE | cut -d'_' -f1,2)_UnmappedMedicago_R2.fastq "; sleep  1; done


5. Index reference rhizobia

       sbatch --partition=pshort_el8 --job-name=StarIndex --time=0-01:00:00 --mem-per-cpu=64G --ntasks=1 --cpus-per-task=1 --output=StarIndex.out --error=StarIndex.error --mail-type=END,FAIL --wrap "cd /data/users/imateusgonzalez/2024_RNAseqTrial/00_ReferenceGenomes; module load STAR/2.7.10a_alpha_220601-GCC-10.3.0; STAR --runThreadN 1 --runMode genomeGenerate --genomeDir /data/users/imateusgonzalez/2024_RNAseqTrial/00_ReferenceGenomes --genomeFastaFiles GCF_003473485.1_MtrunA17r5.0-ANR_genomic.fna --sjdbGTFfile GCF_003473485.1_MtrunA17r5.0-ANR_genomic.gff --sjdbOverhang 99 --genomeSAindexNbases 10"


