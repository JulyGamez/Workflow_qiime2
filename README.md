# Workflow_qiime2-2022.8  
Es necesario instalar la versión qiime2-2022.8 (conda environment - seguir documentación de QIIME2).  
NOTA: Siempre tener cuidado de la ubicación de los archivos.  

## 1. Importing sequences  
- Revisar los archivos que se generaron de la secuenciación. En este caso se debió hacer un manifest en el que se coloca el sample ID, forward o reverse y la ubicación. Una vez realizado, se procede al siguiente comando:  

`qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path manifest.tsv --input-format PairedEndFastqManifestPhred33 --output-path 01_demux1.qza` 

- Para observar la calidad general del archivo se genera un archivo .qzv capaz de visualizarse en: [QIIME2 VIEW](https://view.qiime2.org/)  

`qiime demux summarize --i-data 01_demux1.qza --o-visualization demux.qzv`  

- Se procedió con el manual [Moving pictures](https://docs.qiime2.org/2022.8/tutorials/moving-pictures/) y se realizó el primer vistazo de taxonomía (proceso no mostrado), sin filtrados para observar posibles contaminantes.  

## 2. Quality control 
### 2.1 El mejor Trimming
- Una vez detectados los features y observar el estado actual de las secuencias basándonos en phred, se procede a optimizar los parámetros para el quality control. Para esto se realizaron diferentes juegos de valores para los parámetros de recorte.  
 
- El siguiente comando es un ejemplo de corrida con unos parámetros (se seleccionan dependiendo del 01_demux1.qzv - interactive plot)  

`qiime dada2 denoise-paired --i-demultiplexed-seqs 01_demux1.qza --output-dir first_analysis/02_dada12_16_285 --p-trim-left-f 12 --p-trim-left-r 16 --p-trunc-len-f 285 --p-trunc-len-r 250 --p-n-threads 32`  

- Se generaron los .qzv para su visualización  

`qiime feature-table tabulate-seqs --i-data first_analysis/02_dada12_16_285/representative_sequences.qza --o-visualization first_analysis/02_dada12_16_285/rep-seqs12_16_285.qzv`  

`qiime feature-table summarize --i-table first_analysis/02_dada12_16_285/table.qza --o-visualization first_analysis/02_dada12_16_285/table_dada12_16_285.qzv --m-sample-metadata-file /mnt/Documents/A00826712/16S_CITOCINAS/metadata.tsv`  

`qiime metadata tabulate --m-input-file first_analysis/02_dada12_16_285/denoising_stats.qza --o-visualization first_analysis/02_dada12_16_285/demux-filter-stats12_16_285.qzv`  

- Todo se registró en un Excel "TRIMM_ANALISIS" (Link de acceso al drive donde está ubicado el Excel)  [Onedrive_datos](https://tecmx-my.sharepoint.com/personal/a00826712_tec_mx//_layouts/15/onedrive.aspx?login_hint=A00826712%40tec%2Emx&id=%2Fpersonal%2Fa00826712%5Ftec%5Fmx%2FDocuments%2F16S%5FDIANA)  
- A partir de los datos recabados se seleccinó el TRIMM9. Con estos parámetros 12, 16, 285, 250 se obtuvo una cantidad elevada de features y se conservaron gran parte de los reads  

### 2.2 Filtrado por frecuencia y features  
#### 2.2.1 Definir frecuencia mínima por feature  
- Una vez seleccionado el procesamiento, se debe realizar el filtrado de los features que pueden ser considerados como contaminación, por lo que debemos establecer el valor mínimo de frecuencia de cada feature. Para esto se usará "thumb rule" (fórmula que permite definir valor mínimo esperado con base en el número de muestras)  

- Una vez establecido el valor se corre el siguiente comando para eliminar aquellos features con una frecuencia menor a m=18 (frecuencia por feature) en k=17 (muestras):  

`qiime feature-table filter-features --i-table /mnt/Documents/A00826712/16S_CITOCINAS/first_analysis/02_dada12_16_285/table.qza --p-min-frequency 17 --p-min-samples 18 --o-filtered-table freq-filt-table.qza`  

#### 2.2.2 Eliminar features
- Para eliminar los features que corresponden a mitocondrias, cloroplastos, Archaea y Cyanobacteria, se be primeramente generar un clasificador para asignación taxonómica. Se seleccionó la base de datos de SILVA_138.1 por ser la más acualizada.  

#### 2.2.2.1 Entrenar clasificador  

- Es necesario instalar [RESCRIPT](https://github.com/bokulich-lab/RESCRIPt) en el ambiente de conda de qiime2-2022.8  

`conda install -c conda-forge -c bioconda -c qiime2 -c defaults xmltodict`  

`pip install git+https://github.com/bokulich-lab/RESCRIPt.git`  

- Ingresar a [SILVA_138 database for QIIME2](https://forum.qiime2.org/t/processing-filtering-and-evaluating-the-silva-database-and-other-reference-sequence-data-with-rescript/15494) y seguir los pasos para poder descargar una base de datos de SILVA compatible con QIIME2.  

`qiime rescript get-silva-data --p-version '138.1' --p-target 'SSURef_NR99' --p-include-species-labels --o-silva-sequences silva-138.1-ssu-nr99-rna-seqs.qza --o-silva-taxonomy silva-138.1-ssu-nr99-tax.qza`  
  
`qiime rescript reverse-transcribe --i-rna-sequences silva-138.1-ssu-nr99-rna-seqs.qza --o-dna-sequences silva-138.1-ssu-nr99-seqs.qza`  

- Ahora se debe extraer de la base de datos la sección que se obtuvo de la secuenciación. NOTA: Necesario buscar los primers y el largo amplificado (documento resumen de secuenciación), la longitud mínima y máxima de las lecturas de calidad (se observa en rep-seqs12_16_285.qzv).  

`qiime feature-classifier extract-reads --i-sequences silva-138.1-ssu-nr99-seqs.qza --p-f-primer CCTAYGGGGYGCWGCAG --p-r-primer GACTACHVGGGTATCTAATCC --p-trunc-len 301 --p-min-length 273 --p-max-length 481 --p-n-jobs 32 --o-reads ref-seqs.qza`  

Opción 1  
`qiime rescript evaluate-fit-classifier --i-sequences ref-seqs.qza --i-taxonomy silva-138.1-ssu-nr99-tax.qza --o-classifier silva99-classifier.qza --o-observed-taxonomy silva_predicted-taxonomy.qza --o-evaluation silva_taxonomy-fit-classifier-evaluation.qzv`  

Opción 2  
`qiime feature-classifier fit-classifier-naive-bayes --i-reference-reads ref-seqs.qza --i-reference-taxonomy silva-138.1-ssu-nr99-tax.qza --o-classifier classifier.qza`  

- Se debe realizar una prueba del clasificador, para esto se corrió el siguiente comando:  

`qiime feature-classifier classify-sklearn --i-classifier classifier.qza --i-reads 02_dada12_16_285/representative_sequences.qza --o-classification taxonomy_silva_clas.qza`  

- Ahora se procede a eliminar Chloroplast, Mitochondria, Cyanobacteria, Archaea (especies que no deben de estar ahí) de la tabla previamente filtrada para feature:  

`qiime taxa filter-table --i-table freq-filt-table.qza --i-taxonomy taxonomy_silva_clas.qza --p-exclude Mitochondria,Chloroplast,Cyanobacteria,Archaea --o-filtered-table table_freq_tax_filtered.qza`  

Se realiará un filtrado manual por ID, basándonos en lo que aparece en el control "mock" (todos los IDs que no deben estar presentes en la leche serán removidos). Para esto se deberá exportar la tabla filtrada a biom y de ahí a tsv. El filtrado se realizará en excel, por lo que es necesario exportarlo a nuestro computador  

`qiime tools export --input-path table_freq_tax_filtered.qza --output-path table_biom`  
`biom convert -i table_biom/feature-table.biom -o table_freq_tax_filtered.tsv --to-tsv`  


``  
### 2.3 Batch effect  
- Después de observar los gráficos de abundancias relativas ordenados por batch (fecha de extracción) e identificar visualmente una gran diferencia entre muestras y la presencia de Streptomyces, se decidió eliminar el feature Streptomyces y separar las muestras en dos sets: Num y G.  

`qiime taxa filter-table --i-table feature-table_filt4-.qza --i-taxonomy taxonomy_silva_clas.qza --p-exclude Streptomyces --o-filtered-table feature-table_filt4S.qza`  

`qiime feature-table filter-samples --i-table feature-table_filt4S.qza --m-metadata-file metadata_complete3_NUM.txt --o-filtered-table feature-table_filt4S_NUM.qza`  

`qiime feature-table filter-samples --i-table feature-table_filt4S.qza --m-metadata-file metadata_complete3G.txt --o-filtered-table feature-table_filt4S_G.qza`  

- Para visualizar las tablas y determinar si todo va en orden:  

`qiime metadata tabulate --m-input-file feature-table_filt4S_NUM.qza --o-visualization feature-table_filt4S_NUM.qzv`  

`qiime metadata tabulate --m-input-file feature-table_filt4S_G.qza --o-visualization feature-table_filt4S_G.qzv`  

- Para el caso del set G se realizó un filtrado extra para el batch. Para esto se exportó en tsv la feature-table_filt4S_G y se modificó en excel. Los features eliminados se colocarán en el excel. Posteriormente se exportó feature_table_filt5_G.tsv a qiime2 como qza:  

`biom convert -i feature_table_filt5_G.tsv -o feature_table_filt5_G.biom --table-type "Table" --to-hdf5`  

`qiime tools import --input-path feature_table_filt5_G.biom --type FeatureTable[Frequency] --output-path feature_table_filt5_G.qza`  

### 2.4 Análisis de diversidad  
#### 2.4.1 Generación de árbol para análisis filogenéticos  
- Se debe realizar un árbol con las secuencias representativas generadas para poder seguir con los análisis filogenéticos:  

`qiime phylogeny align-to-tree-mafft-fasttree --i-sequences representative_sequences.qza --o-alignment aligned-rep-seqs.qza --o-masked-alignment masked-aligned-rep-seqs.qza --o-tree unrooted-tree.qza --o-rooted-tree rooted-tree.qza`  

#### 2.4.2 Alpha y beta diversidad  

- Con el árbol generado y las tablas filtradas se procede a realizar los análisis de diversidad. NOTA: Para determinar el --p-sampling-depth se requiere determinar el conteo mínimo de reads por muestra y valorar si se desea mantener muestra o lecturas para otorgar el valor para la rarefacción. En nuestro caso nuestras lecturas mínima y máxima por muestra eran de 5821 y 105787 respectivamente. Colocamos 7712 así perdíamos solo dos muestras del grupo control (1A28 y 1A35).  

`qiime diversity core-metrics-phylogenetic --i-phylogeny rooted-tree.qza --i-table feature-table5.qza --p-sampling-depth 7712 --m-metadata-file metadata3.tsv --output-dir core-metrics-results`  

`qiime diversity alpha-group-significance --i-alpha-diversity faith_pd_vector.qza --m-metadata-file /mnt/Documents/A00826712/16S_CITOCINAS/first_analysis/02_dada12_16_285/metadata3.tsv --o-visualization faith-pd-significance.qzv`  

`qiime diversity alpha-group-significance --i-alpha-diversity evenness_vector.qza --m-metadata-file /mnt/Documents/A00826712/16S_CITOCINAS/first_analysis/02_dada12_16_285/metadata3.tsv --o-visualization evenness-significance.qzv`  

`qiime diversity alpha-group-significance --i-alpha-diversity shannon_vector.qza --m-metadata-file /mnt/Documents/A00826712/16S_CITOCINAS/first_analysis/02_dada12_16_285/metadata3.tsv --o-visualization shannon-significance.qzv`  

`qiime diversity alpha-group-significance --i-alpha-diversity observed_features_vector.qza --m-metadata-file /mnt/Documents/A00826712/16S_CITOCINAS/first_analysis/02_dada12_16_285/metadata3.tsv --o-visualization observed-features-significance.qzv`  

- Para poder analizar estadísticamente los resultados de alpha diversidad se deben exportar los resultados:  

`qiime metadata tabulate --m-input-file /mnt/Documents/A00826712/16S_CITOCINAS/first_analysis/02_dada12_16_285/metadata3.tsv --m-input-file faith_pd_vector.qza --m-input-file shannon_vector.qza --m-input-file evenness_vector.qza --m-input-file observed_features_vector.qza --o-visualization combined_metadata.qzv`  

- Para estadísticos de la diversidad beta:  

`qiime diversity beta-group-significance --i-distance-matrix core-metrics-results/weighted_unifrac_distance_matrix.qza --m-metadata-file metadata3.tsv --m-metadata-column Group --p-method permanova --p-pairwise --o-visualization core-metrics-results/weighted_unifrac_Group_perm.qzv`  

`qiime diversity beta-group-significance --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza --m-metadata-file metadata3.tsv --m-metadata-column Group --p-method permanova --p-pairwise --o-visualization core-metrics-results/unweighted_unifrac_Group_perm.qzv`  

 `qiime diversity beta-group-significance --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza --m-metadata-file metadata3.tsv --m-metadata-column Group --p-method anosim --p-pairwise --o-visualization core-metrics-results/unweighted_unifrac_Group_anosim.qzv`  
 
`qiime diversity beta-group-significance --i-distance-matrix core-metrics-results/weighted_unifrac_distance_matrix.qza --m-metadata-file metadata3.tsv --m-metadata-column Group --p-method anosim --p-pairwise --o-visualization core-metrics-results/weighted_unifrac_Group_anosim.qzv`  
  
- Para realizar BIPLOTS de beta diversidad [TUTORIAL BIPLOT](https://forum.qiime2.org/t/how-to-use-pcoa-biplot/7953/2)
- En core-metrics-results tenemos algunos archivos necesarios: x_distance_matrix y rarefied_table.  

`qiime diversity pcoa --i-distance-matrix weighted_unifrac_distance_matrix.qza --o-pcoa weighted_pcoa_matrix.qza`  

- Necesario obtener una tabla de frecuencia relativa a partir de rarefied_table.qza  

`qiime feature-table relative-frequency --i-table rarefied_table.qza --o-relative-frequency-table rarefied_relative_table.qza`  

- Se genera el pcoa-biplot con el pcoa y la rarefied_relative_table

`qiime diversity pcoa-biplot --i-pcoa weighted_pcoa_matrix.qza --i-features rarefied_relative_table.qza --o-biplot weighted_biplot_matrix.qza`  

- Se genera la visualización del biplot con emperor. Se deben colocar la cantidad de features que deseamos aparezcan (aparecen solo IDs, en caso de querer nombres, es necesario realizar un taxa collapse al nivel taxonomico deseado a partir de la frequency table inicial (feature_table5))

`qiime emperor biplot --i-biplot weighted_biplot_matrix.qza --m-sample-metadata-file /mnt/Documents/A00826712/16S_CITOCINAS/first_analysis/02_dada12_16_285/metadata3.tsv --m-feature-metadata-file /mnt/Documents/A00826712/16S_CITOCINAS/first_analysis/02_dada12_16_285/taxonomy_silva_clas.qza --p-number-of-features 8 --o-visualization weighted_emperor_biplot.qzv`  




