# Aula prática (06/02):
## Anotação estrutural do genoma
### Anotação de DNA Repetitivo com RepeatModeler e RepeatMasker
1. Organizar arquivos 
Rodar dentro da pasta annotation (dentro de Aula_2):
```bash
mkdir repeats
```
2. Construir biblioteca de repetições (RepeatModeler)
Para a prática, nós usamos o RepeatModeler para criar uma base de dados de repeats específica de *A. thaliana* dentro da pasta annotation/repeats. Este passo é computacionalmente pesado, então não precisa rodar na aula.:

```bash
BuildDatabase -name at_db ../reference/GCA_000001735.2_TAIR10.1_genomic.fna
```
```bash
RepeatModeler -database at_db -threads 4
```
O RepeatModeler gera vários arquivos referentes à anotação dentro de pastas com um código representando a corrida. Para a próxima etapa da prática, usaremos apenas o arquivo *consensi.fa.classified* cuja cópia já está disponível na pasta annotation enviada à vocês. O arquivo contém sequências consenso dos repeats encontrados em *A. thaliana*, já classificados de acordo com a nomenclatura usual. O RepeatMasker usará este arquivo como referência para anotar o genoma completo.

3. Mascarar o genoma (RepeatMasker)
Agora que temos uma base de dados customizada, podemos usar o RepeatMasker para anotar e mascarar nosso genoma.
```bash
RepeatMasker \
  -pa 4 \
  -lib ../consensi.fa.classified \
  -xsmall \
  -gff \
  ../reference/GCA_000001735.2_TAIR10.1_genomic.fna
```
O RepeatMasker gera os arquivos de output na mesma pasta onde está o genoma de referência utilizado. Para esta prática, os arquivos foram copiados para a spoilers_anno/repeats. Navegue até ela e vamos conferir alguns dos outputs mais importantes:
```bash
head GCA_000001735.2_TAIR10.1_genomic.fna.masked
```
```bash
head GCA_000001735.2_TAIR10.1_genomic.fna.out
```
```bash
head GCA_000001735.2_TAIR10.1_genomic.fna.out.gff
```
```bash
grep -v "^#" GCA_000001735.2_TAIR10.1_genomic.fna.tbl
```
## Anotação estrutural de genes 
Ative o ambiente:
```bash
conda activate genome_assembly_course
```
Agora, instale o gffcompare dentro desse ambiente:
```bash
conda install -c conda-forge -c bioconda gffcompare -y
```
###  Helixer  - software de anotação de novo de genomas eucarióticos
Artigo - https://doi.org/10.1093/bioinformatics/btaa1044

Site - https://www.plabipd.de/helixer_main.html

Helixer é um software de anotação estrutural de genomas baseado em *deep learning*. Diferente de métodos clássicos, ele aprende padrões biológicos diretamente das sequências. Ab initio assistido por aprendizado de máquina. Não precisa de RNA-seq ou proteínas (mas pode ser refinado depois).

###  EviAnn  - software de anotação de genomas eucarióticos baseado em evidências
Artigo - https://www.biorxiv.org/content/10.1101/2025.05.07.652745v2 

GitHUB - alekseyzimin/EviAnn_release: This is the standalone version of the EviAnn pipeline 

É baseado em evidências (RNA-seq + proteínas de espécies próximas suporte evolutivo + genoma FASTA).
IMPORTANTE: Atenção mascarar genoma antes da anotação!  
1. Organizar arquivos:
A pasta data/annotation (drive) contém as pastas reference (genoma de referência), rna-seq (reads zipados de rna-seq) e prot (referência de proteínas)
As pastas para as análises devem ser criadas dentro de annotation com o comando:
```bash
mkdir -p results_anno_dri/{trimmed,logs,index,bam,enviann}
```
2. Trimar (Trim Galore) — single-end
Arquivos single-end. Rodar (em background com log):
```bash
for srr in SRR3581356 SRR3581681 SRR3581693 SRR3581703 SRR3581705 SRR3581706 SRR3581708; do
  nohup trim_galore \
    -q 20 \
    --cores 4 \
    -o results_anno_dri/trimmed \
    rna-seq/${srr}.fastq.gz \
    > results_anno_dri/logs/${srr}.trim_galore.out 2>&1 &
done
```  
3. Mapear evidências de RNA-Seq (HISAT2)
   
3.1 Indexar genoma
Construir o índice do genoma (só 1 vez)
```bash
hisat2-build reference/GCA_000001735.2_TAIR10.1_genomic.fna results_anno_dri/index/hisat2_masked_genome
```
Isso cria vários arquivos .ht2 dentro de results_anno/index/

3.2 Alinhamento com HISAT2 
Comando para rodar com todas as amostras de RNA-Seq (não rodar em aula):
```bash
for srr in SRR3581356 SRR3581693 SRR3581681 SRR3581705 SRR3581706 SRR3581708 SRR3581703
do
    echo "Aligning $srr..."
```
```bash
    hisat2 -p 8 \
        -x results_anno/index/hisat2_masked_genome \
        -U results_anno/trimmed/${srr}_trimmed.fq.gz \
        2> results_anno/logs/${srr}.hisat2.log \
    | samtools view -@ 4 -bS - \
    > results_anno/bam/${srr}.bam
done
```
RODAR apenas 1 bam pra aula pŕatica (escolher um para o lugar do SRRXXXXXXX):

```bash
hisat2 -p 8 \
    -x results_anno_dri/index/hisat2_masked_genome \
    -U results_anno_dri/trimmed/SRRXXXXX_trimmed.fq.gz \
    2> results_anno_dri/logs/SRRXXXXXX.hisat2.log \
| samtools view -@ 4 \
> results_anno_dri/bam/SRRXXXXXX.bam
```

Depois disso é necessário ordenar (sorting) e indexar o arquivo BAM
```bash
samtools sort -@ 4 \
    -o results_anno/bam/SRRXXXXXXX.sorted.bam \
    results_anno/bam/SRRXXXXXXX.bam
```
```bash
 samtools index results_anno/bam/SRRXXXXXXX.sorted.bam
```
4.  Evidência Proteínas de espécie próxima 
Entrar em: https://www.uniprot.org/ 
Buscar por Brassicaceae
Fazer download de todas as proteínas “Reviewed(Swiss-Prot)
O arquivo já está baixado na pasta annotation/prot

5.  Rodar EviAnn
Para rodar o EviAnn é necessário criar um arquivo de texto com uma lista de BAMs para usarmos como referência extrínseca de rna-seq. 
```bash
nano rna_evidence.txt
```
Vamos usar os BAMs da aula, substitua o SRRXXXXXXX com o BAM que você fez o index. Escreva no nano:
```bash
results_anno/bam/SRRXXXXXXX.sorted.bam bam
```
Agora vá para a pasta results_anno/enviann e rode:
```bash
eviann.sh \
  -g ../../reference/GCA_000001735.2_TAIR10.1_genomic.fna \
  -r rna_evidence.txt \
  -p ../../prot/Brassicaceae_SwissProt.fasta \
  -t 8
```
6. Comparar com anotação oficial 
Atividade: Baixar o gff oficial TAIR 10 https://drive.google.com/file/d/1lXypcEqUxkoGR1rDYl8Uy89l9B9q3pNn/view?usp=drive_link
```bash
gffcompare -r ATH_original_genomic.chromfix.gff3 \
           ../TAIR10_helixer.gff \
           results_anno/enviann/GCA_000001735.2_TAIR10.1_genomic.fna.gff
```
#Resultado em gffcmp.stats
Referência
~59.396 mRNAs em 36.963 loci
Representa um conjunto rico e complexo (TAIR do assembly)

7. Galaxy web server  
Galaxy é uma plataforma online de bioinformática que permite rodar análises complexas sem usar linha de comando.
História Galaxy : **https://usegalaxy.org/u/leeg-01/h/ecologia-molecular**
- É como um laboratório virtual
- Tudo roda em servidor
- Você usa por interface gráfica
- Guarda histórico das análises
Acessar o Galaxy: https://usegalaxy.org 
Criar conta → login
Enviar arquivos gff3 de todas as anotações incluindo do artigo original 
Transformar arquivos GFF3 em Fasta de proteínas 

Na barra de ferramentas, digitar:
###  BUSCO: Assess genome assembly and annotation completeness
Configurar 
Input    **proteínas FASTA**
Mode    **proteins**
Lineage dataset    **embryophyta_odb10 (plantas)**
Clique em **Run**
Interpretar resultado

33.741 out of 33741 consensus transcripts written in gffcmp.combined.gtf
O gffcompare gerou uma anotação integrada
**Atividade:  Comparando as duas anotações estruturais geradas nesta aula prática com os resultados reportados no artigo de referência (abaixo), qual pipeline apresentou melhor desempenho e por quê? Discuta quais métricas suportam essa conclusão (ex.: BUSCO, número de genes/transcritos, completude/fragmentação), e indique qual pipeline você recomendaria para análises futuras, justificando sua escolha.**

**Anotação funcional de genes** 
Galaxy
Buscar por ferramenta **InterProScan (domínios proteicos)**
Configurar
Input	**proteínas** FASTA
Applications	**todas** (ou Pfam, SMART, TIGRFAM)
Output	**TSV**
Clique em **Run**
```bash
cat  InterProScan.tabular | cut -f6 | sort -u
```
Saída tabular padrão do InterProScan, que integra:
-	domínios proteicos
-	famílias
-	assinaturas (ProSite, Pfam, SMART, etc.)
-	termos GO
-	vias metabólicas (MetaCyc, Reactome)
-	Interpretação biológica do exemplo: Essa proteína tem 642 aa; contém um domínio de ligação a ATP; pertence à família das proteínas quinases; participa de vias de sinalização (Reactome); função molecular bem definida (ATP binding);

**Atividade: Escolha um domínio PFAM, descreva quantas proteinas você encontra para esse dominio e qual a função para a planta.**

Observações importantes :
Cada proteína pode contar mais de uma via;
Isso não é enriquecimento estatístico;
É frequência bruta;
Para enriquecimento real, seria necessário:
Background
Testes estatísticos (GO enrichment)

**REVIGO**
O REVIGO é uma ferramenta online usada para resumir e organizar listas de termos de Gene Ontology (GO), especialmente após análises de enriquecimento funcional. Em estudos genômicos e transcriptômicos, é comum obter listas extensas de termos GO, muitas vezes redundantes ou semanticamente muito semelhantes. O REVIGO resolve esse problema ao calcular a similaridade semântica entre os termos e remover aqueles repetitivos ou pouco informativos, agrupando-os em categorias mais representativas. Como resultado, ele gera visualizações como gráficos de dispersão, treemaps e tabelas resumidas que facilitam a interpretação biológica dos dados. 
Site : **http://revigo.irb.hr/**

 
