# Trimming and Mapping of Whole Genome Sequence Reads

NB These command lines are snipets from submission scripts optimised for our HPC server, they provide a summary of the commands and the arguments used. 

# Adaptor removal and trimming
Reads were trimmed with ```trimmomatic v0.35``` to remove adaptor sequence and low quality sequence. Reads were trimmed if the base quality at the beginning or end of the read was <3, and only reads of more than 50 bp were retained.

```
trimmomatic PE -threads 2 -phred33 ${data}/${name}_R1.fastq.gz ${data}/${name}_R2.fastq.gz ${out}/${name}.R1.trim.fq.gz ${out}/logs/${name}.R1.un.fq.gz ${out}/${name}.R2.trim.fq.gz ${out}/logs/${name}.R2.un.fq.gz ILLUMINACLIP:${adaptor}:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:50 > ${out}/logs/${name}.trimmo.log  2> ${out}/logs/${name}.trimmo.err
```

# Mapping with bwa to the reference genome
Reads were mapped to the reference genome (CB_CbHevensen-1_hq-v2) using ```bwa mem v0.7.17```
An example command line was 
```
bwa mem ${Ref} ${data}${name}_1.trim.fq.gz ${data}${name}_2.trim.fq.gz -M -R "@RG\tID:${name}\tSM:${name}\tPL:Illumina" -t ${proc} > ${TMPDIR}/${name}.sam
```

The sam files were converted to bams using ```sambamba v0.6.6```
```
sambamba view -t ${proc} -S ${TMPDIR}/${name}.sam -f bam -l 0 -o /dev/stdout|sambamba sort /dev/stdin -o /dev/stdout t ${proc} -l 0 -m 10GB --tmpdir ${TMPDIR} > ${TMPDIR}/${name}.sort.bam
```

Mapping statistics and filtering were performed using ```sambamba v0.6.6``` as follows
```
sambamba flagstat ${TMPDIR}/${name}.sort.bam > ${path_out}stats/${name}
```

Low quality alignments (Qual >10) were removed and duplicates were marked as follows 
```
sambamba view  -F "mapping_quality >= "${Qual} ${TMPDIR}/${name}.sort.bam -o ${TMPDIR}/${name}_sort_${Qual}.bam -t ${proc} -l 0 -f bam 
gatk MarkDuplicates --TMP_DIR=${TMPDIR} --INPUT=${TMPDIR}/${name}_sort_${Qual}.bam --OUTPUT=${path_out}${name}_sort_${Qual}_dup.bam --METRICS_FILE=${path_out}stats_dup/${name} --VALIDATION_STRINGENCY=LENIENT --MAX_FILE_HANDLES_FOR_READ_ENDS_MAP=1024
sambamba index ${path_out}${name}_sort_${Qual}_dup.bam
```

Mapping statistics after quality filtering and marking duplicates were outputted
```
sambamba flagstat ${path_out}${name}_sort_${Qual}_dup.bam > ${path_out}statsQ${Qual}/${name}
```

Mean coverage for each individual was estimated using ``` bedtools v2.27.1```
```
genomeCoverageBed -ibam ${path_out}${name}_sort_${Qual}_dup.bam -d | awk '{ total += $3 } END { print total/NR }' >  ${path_out}statsQ${Qual}/${name}_cov
```
