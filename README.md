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

- Una vez establecido el valor se corre el siguiente comando para eliminar aquellos features con una frecuencia menor a ___:  

`qiime feature-table filter-features --i-table 02_dada12_16_285/table.qza --p-min-frequency 10 --o-filtered-table freq-filt-table.qza`    

#### 2.2.2 Eliminar features
- Para eliminar los features que corresponden a mitocondrias, cloroplastos, Archaea y Cyanobacteria, se be primeramente generar un clasificador para asignación taxonómica. Se seleccionó la base de datos de SILVA_138.1 por ser la más acualizada.  

- Para esto desde el computador se debe descargar el archivo con las secuencias fasta: "SILVA_138.1_SSURef_NR99_tax_silva.fasta.gz" de [SILVA_138_fasta](https://www.arb-silva.de/no_cache/download/archive/release_138_1/Exports/); y el archivo con los IDs de la taxonomía: "tax_slv_ssu_138.1.txt.gz" de [taxonomy_silva](https://www.arb-silva.de/no_cache/download/archive/release_138_1/Exports/taxonomy/).  

- Una vez descargados, se procede a convertirlos a artifacto de qiime2:  

`qiime tools import --type 'FeatureData[Sequence]' --input-path SILVA_138.1_SSURef_NR99_tax_silva.fasta.gz --output-path SILVA_138.1_99_seqs.qza`  

`qiime tools import --type 'FeatureData[Taxonomy]' --input-format HeaderlessTSVTaxonomyFormat --input-path tax_slv_ssu_138.1.txt --output-path ref-taxonomy.qza`  

- Ahora se debe extraer de la base de datos la sección que se obtuvo de la secuenciación. NOTA: Necesario buscar los primers y el largo amplificado (documento resumen de secuenciación), la longitud mínima y máxima de las lecturas de calidad (se observa en rep-seqs12_16_285.qzv).  

`qiime feature-classifier extract-reads --i-sequences SILVA_138.1_99_seqs.qza --p-f-primer CCTAYGGGGYGCWGCAG --p-r-primer GACTACHVGGGTATCTAATCC --p-trunc-len 301 --p-min-length 273 --p-max-length 481 --o-reads ref-seqs.qza`  

- Ahora se procede a eliminar Chloroplast, Mitochondria, Cyanobacteria, Archaea (especies que no deben de estar ahí):  

### 2.3 Batch effect
