# Montagem, Anotação e Análise de Variantes Genômicas

## Curso Prático Hands-on

### Visão geral

Este repositório contém os materiais e fluxos de trabalho de um curso prático focado em montagem de genomas, anotação estrutural e funcional, e identificação de variantes genéticas em plantas, utilizando *Arabidopsis thaliana* como sistema modelo.

O curso cobre todo o pipeline de bioinformática, desde os dados brutos de sequenciamento (leituras longas e curtas), passando pela montagem do genoma e sua avaliação, anotação de elementos repetitivos, predição de genes por abordagens ab initio e baseadas em evidências, anotação funcional, e, por fim, identificação de SNPs e análises de genética de populações.

Todas as análises são realizadas por meio de ferramentas de linha de comando em ambientes Linux e, quando apropriado, complementadas por plataformas web como o Galaxy.

### Público-alvo e conhecimentos prévios
Este curso é destinado a:
Estudantes de pós-graduação, pós-doutorandos e pesquisadores das áreas de genômica, biologia molecular, ecologia e evolução
Bioinformatas e técnicos que trabalham com dados genômicos
Estudantes interessados em aprender pipelines práticos de análise genômica

### Conhecimentos prévios esperados
Para melhor aproveitamento do curso, espera-se que os participantes:
Tenham familiaridade com ambientes Linux e linha de comando
Possuam conhecimentos básicos de genômica (DNA, RNA, genes, SNPs)
Tenham noções básicas de R e/ou Python (recomendado, mas não obrigatório)

### Programa
Dia 1 (05/02): Montagem e Avaliação do Genoma
+ Montagem do genoma utilizando leituras longas e curtas
+ Instalação de ferramentas bioinformáticas utilizando conda
+ Organização de dados e estruturação do fluxo de trabalho
+ Controle de qualidade das leituras de sequenciamento
+ Leituras longas: NanoPlot
+ Leituras curtas: FastQC e MultiQC
+ Perfil genômico por análise de k-mers (Jellyfish, GenomeScope)
+ Montagem do genoma com leituras PacBio HiFi usando hifiasm
+ Conversão de arquivos GFA para FASTA
+ Avaliação da montagem:
+ Estatísticas básicas (SeqKit)
+ Qualidade e contiguidade da montagem (QUAST)
+ Avaliação de completude (BUSCO)
+ Comparação com o genoma de referência publicado de *A. thaliana*

Dia 2 (06/02): 
Anotação Estrutural do Genoma
+ Anotação de elementos repetitivos e predição gênica
+ Identificação e anotação de DNA repetitivo
+ RepeatModeler
+ RepeatMasker
+ Estratégias de mascaramento do genoma
+ Anotação estrutural de genes com Helixer (anotação ab initio baseada em aprendizado profundo)
+ Anotação estrutural de genes com EviAnn (anotação baseada em evidências usando RNA-seq e homologia proteica)
+ Comparação de anotações com GFF3
+ Introdução ao Galaxy para avaliação da anotação com BUSCO 
