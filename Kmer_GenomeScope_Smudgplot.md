

# KMC 
Kmers (19mer) were counted from trimmed sequence reads using KMC
https://github.com/refresh-bio/KMC

Example submission script looping through all samples in the file 'read.list'.

```
IDX=$LSB_JOBINDEX
name=`sed -n ${IDX}p </read.list`
mkdir tmp_${name}
printf '%s\n' ${data}/${name}.R1.trim.fq.gz  ${data}/${name}.R2.trim.fq.gz > FILES_${name} 
sed -i 's/\s\+/\n/g' FILES_${name} 
/cluster/project/gdc/shared/tools/KMC_June2021/bin/kmc -k19 -t16 -m64 -ci1 -cs10000 @FILES_${name}  kmc/${name} tmp_${name}
/cluster/project/gdc/shared/tools/KMC_June2021/bin/kmc_tools transform kmc/${name} histogram ${out}/${name}_k19.hist -cx10000

awk -v OFS=' ' '{ print $1, $2 }' ${out}/${name}_k19.hist > ${out}/${name}_k19_sp.hist
```


# Genome Scope
Genome scope was used to estimate the heteroyzgosity using kmers
https://github.com/schatzlab/genomescope

```
while read p; do 
Rscript ~/bin/genomescope.R ./kmc_hist/${p} 19 150 ./gscope_out/tmp/
mv ./gscope_out/tmp/plot.png ./gscope_out/${p}.png 
mv ./gscope_out/tmp/summary.txt ./gscope_out/${p}.summary

done<hist_file.list

```

# Smudgeplot
Smudgeplot was used to visualise the ploidy, estiamte genome size and coverage. 

```
while read p; do 
L=$(smudgeplot.py cutoff ${data}/${p}_k19_sp.hist L)
U=$(smudgeplot.py cutoff ${data}/${p}_k19_sp.hist U)
smudgeplot.py plot ${out}/${p}_L"$L"_U"$U"_coverages.tsv -o ${out}/${p}
done<kmc_file.list
```

