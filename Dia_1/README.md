## Download dos dados da aula
Os arquivos necessários para esta prática estão disponíveis no Google Drive.
Pasta com os dados:  
https://drive.google.com/drive/folders/1wFpFSaT4xzVpjIs0ACVK4b2fnjpvHUsx

# Aula Prática 1 — Montagem de Genoma  
**Long reads + Short reads**

---

## 1. Instalação dos programas

Todos os programas serão instalados em um ambiente **conda**.  
Na pasta do GitHub há um arquivo chamado `genome_assembly_course.yml` que contém as instruções para a criação desse ambiente.

```bash
conda env create -f genome_assembly_course.yml
conda activate genome_assembly_course
```

---

## 2. Organização dos dados

É importantíssimo tomar cuidado com a organização dos dados.  
A pasta `Aula_1` contém as subpastas `subsampled` e `reference`, que possuem os arquivos utilizados durante a aula.  
Vamos criar pastas para armazenar os resultados das análises.

```bash
mkdir -p results/{qc,profiling,assembly,polish,eval}
```

---

## 3. Qualidade dos reads

O primeiro passo em qualquer workflow de montagem é sempre a checagem da qualidade do sequenciamento.  
Estamos usando reads de *Arabidopsis thaliana* obtidos por duas plataformas: reads longos PacBio HiFi e reads curtos Illumina.

### 3.1 Long reads — NanoPlot

```bash
NanoPlot \
  --fastq subsampled/Long/at_hifi_30x.fastq \
  -o results/qc/nanoplot \
  --threads 4
```

Arquivos para checar em `results/qc/nanoplot/`:

- `LengthvsQualityScatterPlot_dot.png`
- `NanoStats.txt`

---

### 3.2 Short reads — FastQC e MultiQC

```bash
fastqc subsampled/Short/ind_1/SRR1560657_1_sub.fastq.gz subsampled/Short/ind_1/SRR1560657_2_sub.fastq.gz \
       subsampled/Short/ind_2/SRR1581142_1_sub.fastq.gz subsampled/Short/ind_2/SRR1581142_2_sub.fastq.gz \
       -o results/qc/fastqc -t 4

multiqc results/qc -o results/qc/multiqc
```

Arquivos para checar:

- Relatórios `.html` em `results/qc/fastqc`
- Relatório geral em `results/qc/multiqc`

---

## 4. Profiling do genoma

### 4.1 Contagem de k-mers (Jellyfish)

O profiling permite investigar atributos importantes do genoma usando reads curtos, baseando-se na contagem de k-mers.

```bash
jellyfish count \
  -C -m 21 -s 1G -t 4 \
  <(zcat subsampled/Short/ind_1/SRRXXXXXX_1_sub.fastq.gz subsampled/Short/ind_1/SRRXXXXXX_2_sub.fastq.gz) \
  -o results/profiling/reads.jf

jellyfish histo -t 4 results/profiling/reads.jf > results/profiling/kmer.histo

genomescope2 \
  -i results/profiling/kmer.histo \
  -o results/profiling/genomescope \
  -k 21
```

Arquivos para checar em `results/profiling/genomescope/`:

- `summary.txt`
- `transformed_linear_plot.png`

---

## 5. Montagem com long reads — Hifiasm

```bash
hifiasm \
  -o results/assembly/at \
  -t 8 \
  subsampled/Long/at_hifi_30x.fastq
```

Conversão de GFA para FASTA:

```bash
awk '/^S/{print ">"$2"\n"$3}' results/assembly/at.bp.hap1.p_ctg.gfa > results/assembly/hap1.fasta
awk '/^S/{print ">"$2"\n"$3}' results/assembly/at.bp.hap2.p_ctg.gfa > results/assembly/hap2.fasta
```

---

## 6. Avaliação da montagem

### 6.1 Estatísticas básicas — SeqKit

```bash
seqkit stats results/assembly/hap1.fasta results/assembly/hap2.fasta
seqkit stats reference/A_thaliana_ref.fasta
```

---

### 6.2 Avaliação estrutural — QUAST

```bash
quast results/assembly/hap1.fasta -o results/eval/quast_1
quast results/assembly/hap2.fasta -o results/eval/quast_2
quast reference/A_thaliana_ref.fasta -o results/eval/quast_ref
```

---

### 6.3 Completude gênica — BUSCO

```bash
busco -i results/assembly/hap1.fasta -l embryophyta_odb10 -m genome -o results/assembly/busco/busco_hap1 -c 8
busco -i results/assembly/hap2.fasta -l embryophyta_odb10 -m genome -o results/assembly/busco/busco_hap2 -c 8
busco -i reference/A_thaliana_ref.fasta -l embryophyta_odb10 -m genome -o results/assembly/busco/busco_ref -c 8
```

---

## 7. Isolando os cinco maiores contigs

```bash
seqkit sort -l -r results/assembly/hap1.fasta | seqkit head -n 5 > results/assembly/hap1_top5.fasta
seqkit sort -l -r results/assembly/hap2.fasta | seqkit head -n 5 > results/assembly/hap2_top5.fasta
seqkit sort -l -r reference/A_thaliana_ref.fasta | seqkit head -n 5 > results/assembly/ref_top5.fasta
```

```bash
seqkit stats results/assembly/hap1_top5.fasta results/assembly/hap2_top5.fasta
seqkit stats results/assembly/ref_top5.fasta
```

```bash
busco -i results/assembly/hap1_top5.fasta -l embryophyta_odb10 -m genome -o results/assembly/busco/busco_hap1_top5 -c 8
busco -i results/assembly/hap2_top5.fasta -l embryophyta_odb10 -m genome -o results/assembly/busco/busco_hap2_top5 -c 8
busco -i results/assembly/ref_top5.fasta -l embryophyta_odb10 -m genome -o results/assembly/busco/busco_ref_top5 -c 8
```
