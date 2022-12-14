# Workflow_qiime2

## 1. Importing sequences  
- Revisar los archivos que se generaron de la secuenciación. En este caso se debió hacer un manifest en el que se coloca el sample ID, forward o reverse y la ubicación. Una vez realizado, se procede al siguiente comando:  

`qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path manifest.tsv --input-format PairedEndFastqManifestPhred33 --output-path 01_demux1.qza` 

- Se realizó el primer vistazo de taxonomía, sin filtrados para observar posibles contaminantes  

## 2. Quality control  
- Una vez detectados los features, se procede a optimizar los parámetros para el quality control. Para esto difrentes se realizaron diferentes juegos de valores para los parámetros de recorte.  

`qiime dada2 denoise-paired --i-demultiplexed-seqs 01_demux1.qza --output-dir first_analysis/02_dada2_270 --p-trim-left-f 2 --p-trim-left-r 2 --p-trunc-len-f 290 --p-trunc-len-r 270 --p-n-threads 32`  

- Todo se registró en un Excel (se colocará link de acceso al drive donde está ubicado el Excel)  
- 

