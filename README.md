# Workflow_qiime2

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

`qiime dada2 denoise-paired --i-demultiplexed-seqs 01_demux1.qza --output-dir first_analysis/02_dada2_270 --p-trim-left-f 2 --p-trim-left-r 2 --p-trunc-len-f 290 --p-trunc-len-r 270 --p-n-threads 32`  

- Se generaron los .qzv para su visualización  

`qiime dada2 denoise-paired --i-demultiplexed-seqs 01_demux1.qza --output-dir first_analysis/02_dada2_270 --p-trim-left-f 2 --p-trim-left-r 2 --p-trunc-len-f 290 --p-trunc-len-r 270 --p-n-threads 32`

- Todo se registró en un Excel (Link de acceso al drive donde está ubicado el Excel)  [Onedrive_datos](https://tecmx-my.sharepoint.com/personal/a00826712_tec_mx//_layouts/15/onedrive.aspx?login_hint=A00826712%40tec%2Emx&id=%2Fpersonal%2Fa00826712%5Ftec%5Fmx%2FDocuments%2F16S%5FDIANA) 
- 

