# K-mer GWAS
Kmers were counted with KMC following recomendations from (https://github.com/voichek/kmersGWAS). Using these kmer counts I performed the GWAS in R with Gapit (https://github.com/jiabowang/GAPIT)

# Counting k-mers with KMC

```
IDX=$LSB_JOBINDEX
name=`sed -n ${IDX}p <read.list`
mkdir ${name}
cd ${name}
printf '%s\n' ${data}/${name}.R1.trim.fq.gz  ${data}/${name}.R2.trim.fq.gz > FILES_${name} 
sed -i 's/\s\+/\n/g' FILES_${name} 

[path]/tools/KMC_June2021/bin/kmc -k31 -t16 -m64 -ci2 -cs10000 @FILES_${name}  output_kmc_cannon ./ 1> kmc_cannon.1 2> kmc_cannon.2 
[path]/KMC_June2021/bin/kmc -k31 -t16 -m64 -ci0 -cs10000 @FILES_${name}  output_kmc_all ./ 1> kmc_all.1 2> kmc_all.2 
[path]/kmerGWAS/bin/kmers_add_strand_information -c output_kmc_canon -n output_kmc_all -k 31 -o kmers_with_strand 1> kmc_strand.1 

rm ${name}/*.kmc*
#rm FILES_${name}

```
# K-mer data converted to plink file 
Converted k-mer data into a plink file coding k-mers presence/absence as two homozygous variants and removing k-mers with a minor frequency of less than 0.05

```
/bin/kmers_table_to_bed -t kmers_table -k 31 -p phenotypes.pheno --maf 0.05 --mac 5 -b 10000000 -o output_file
```

The k-mer plink(.bed) output was imported into plink (v1.9, https://www.cog-genomics.org/plink/) and converted to a vcf file using `--recode vcf`

  
# GWAS with R and GAPIT

The vcf file was split into 99 smaller files, then converted to hapmap file using R::vcfR (https://github.com/knausb/vcfR) and then GAPIT GWAS was run on each of these smaller files.

To split the vcf file, first the header was removed and saved to another file, then the file was split into smaller files contiaing 20,000 k-mers and the header was attached to each of these smaller files.

```
head -7 kmer.vcf > header.vcf
sed 1,7d kmer.vcf > vcf_noheader.txt
split -d -l 200000 vcf_noheader.txt sub && for X in sub*; do { cat header.vcf "$X"; } > split_files/$X.txt; done

```


Then all Convert vcf to hapmap 

```
vcf_file <-  "kmer.vcf"

vcf <- read.vcfR(vcf_file, verbose = FALSE)
myHapMap <- vcfR2hapmap(vcf)
write.table(myHapMap, file = "kmer.hmp.txt", sep = "\t", row.names = FALSE, col.names = FALSE) 

```





# Mapping k-mers to reference genome
