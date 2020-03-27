# Instalação

Via CONDA

```
wget https://data.qiime2.org/distro/core/qiime2-2020.2-py36-linux-conda.yml
conda env create -n qiime2-2020.2 --file qiime2-2020.2-py36-linux-conda.yml
OPTIONAL CLEANUP
rm qiime2-2020.2-py36-linux-conda.yml
```

Após a instalação, ativar ambiente conda

```
source activate qiime2-2020.2
```
Nota: Se não der certo, digite `/usr/local/miniconda3/bin/conda init` e reinicie a seção. 

# Noções básicas sobre arquivos QIIME2

O QIIME2 usa dois tipos diferentes de arquivos:
- .qza são os que contêm dados e metadados de uma análise. Na verdade, é apenas um arquivo zip que contém um diretório especialmente formatado com dados e metadados. Você pode ver que tipo de dados estão contidos em um arquivo de dados com o comando `qiime tools peek filename.qza.`
- .qzv são arquivos para visualizações. Todos os arquivos QIIME2 podem ser visualizados usando um navegador on-line disponível em https://view.qiime2.org .

# Importando dados
Podemos importar os dados de diversas maneiras. Aqui vou exemplificar a importação de single reads multiplex. Para isso, crie um diretório “sequences” e baixes os dois arquivos ###########

Após o Download dos arquivos utilize esse comando para importar os dados:
 
```
qiime tools import \
  --type EMPSingleEndSequences \
  --input-path sequences \
  --output-path sequences.qza
```
Agora, precisamos saber quais sequências estão associadas a cada amostras.  Para isso, utilizaremos as informações do `sample-metadata.tsv` ###########

```
qiime demux emp-single \
  --i-seqs sequences.qza \
  --m-barcodes-file sample-metadata.tsv \
  --m-barcodes-column barcode-sequence \
  --o-per-sample-sequences demux.qza \
  --o-error-correction-details demux-details.qza
```

Para visualizar a quantidade de sequências por amostra utilizaremos o `qiime demux summarize`
 
```
qiime demux summarize --i-data demux.qza --o-visualization demux.qzv
```

Pronto! Seus dados estão prontos para serem Analisados!!


# DADA2
Aqui, utilizaremos o DADA2 ***falar mais coisas do DADA

```
qiime dada2 denoise-single \
  --i-demultiplexed-seqs demux.qza \
  --p-trim-left 0 \
  --p-trunc-len 120 \
  --o-representative-sequences rep-seqs-.qza \
  --o-table table.qza \
  --o-denoising-stats stats.qza
````

Para Visualizar
```
qiime metadata tabulate --m-input-file stats-dada2.qza   --o-visualization stats-dada2.qzv
```

# Pausa para reflexão
*Importância da qualidade de sequenciamento. `fastqc`
*Tipos de importação

# Geração de árvore filogenética
```
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences rep-seqs.qza \
  --o-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza \
  --o-rooted-tree rooted-tree.qza
```

# Taxonomia
Os tutoriais disponíveis no QIIME2 sugerem a taxonomia GreenGenes, aqui utilizaremos a taxonomia SILVA v. 132. Esse banco de dados foi atualizado em 2018/2019, enquanto o GreenGenes teve sua última atualização em 2012/2013.

```
qiime feature-classifier classify-sklearn \
  --i-classifier database/silva-132-99-nb-classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv

qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization taxa-bar-plots.qzv
```

# CORE
```
qiime feature-table core-features --i-table table.qza --o-visualization CORE/core.qzv
```

# Diversidade
###escrever algo
```
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny rooted-tree.qza \
  --i-table table.qza \
  --p-sampling-depth 1103 \
  --m-metadata-file sample-metadata.tsv \
  --output-dir diversity-results

```

##escrever algo

```
qiime diversity alpha-group-significance \
  --i-alpha-diversity diversity-results/faith_pd_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization diversity-results/faith-pd-group-significance.qzv

qiime diversity alpha-group-significance \
  --i-alpha-diversity diversity-results/evenness_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization diversity-results/evenness-group-significance.qzv
```

#Alfa rarefação
```
qiime diversity alpha-rarefaction \
  --i-table table.qza \
  --i-phylogeny rooted-tree.qza \
  --p-max-depth 4000 \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization diversity-results/alpha-rarefaction.qzv
```

# Filtragem de dados
Antes de passar para as análises estatísticas, uma abordagem muito útil para nossas análises é a filtragem de dados de acordo com nosso interesse científico.
Por exemplo, seria mais interessante analisármos o CORE microbioma por partes do corpo. 

```
qiime feature-table filter-samples \
  --i-table table.qza \
  --m-metadata-file sample-metadata.tsv \
  --p-where "[body-site]='gut'" \
  --o-filtered-table gut-table.qza
````

# Estatística
#####escrever algo
PERMANOVA
##escrever algo
```
qiime diversity beta-group-significance \
  --i-distance-matrix diversity-results/unweighted_unifrac_distance_matrix.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column body-site \
  --o-visualization statistics/unweighted-unifrac-body-site-significance.qzv \
  --p-pairwise

qiime diversity beta-group-significance \
  --i-distance-matrix diversity-results/unweighted_unifrac_distance_matrix.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column subject \
  --o-visualization statistics/unweighted-unifrac-subject-group-significance.qzv \
  --p-pairwise
```

ANCON
#escrever algo
````
qiime composition add-pseudocount \
  --i-table gut-table.qza \
  --o-composition-table statistics/comp-gut-table.qza

qiime composition ancom \
  --i-table  statistics/comp-gut-table.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column subject \
  --o-visualization  statistics/ancom-subject.qzv

qiime taxa collapse \
  --i-table gut-table.qza \
  --i-taxonomy taxonomy.qza \
  --p-level 6 \
  --o-collapsed-table gut-table-l6.qza

qiime composition add-pseudocount \
  --i-table gut-table-l6.qza \
  --o-composition-table statistics/comp-gut-table-l6.qza

qiime composition ancom \
  --i-table statistics/comp-gut-table-l6.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column subject \
  --o-visualization statistics/l6-ancom-subject.qzv
 ``` 
