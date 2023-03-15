# INSTRUCCIONES 
## Paquetes usados en Python

* matplotlib.pyplot
* numpy
* pandas
* requests
* subprocess
* sys
* biopython

## Corrida del código

El código presenta 3 secciones. La primera consiste en la descarga de los archivos fasta usados en el flujo de trabajo. La base de datos de donde se extrae ambos archivos es Uniprot.
La segunda parte consiste el uso de ejecutables BLAST+ para poder hacer el alineamiento del proteoma humano con las proteínas relacionadas a melanoma en *Homo sapiens*. Luego se uso las secuencias que se alinearan con el proteoma y se las uso de input para correr Interproscan.

### Uniprot código
* Input : Tener una definición del query que se quiere encontrar
* Código :
  ```uniprot_api_url  = "https://rest.uniprot.org/uniprotkb/stream"
  uniprot_api_args = {"compressed" : "false",
                    "format"     : "fasta",
                    "query"      : "(HIV virus) AND (reviewed:true) AND (organism_id:9606)"}
  uniprot_query_seqs = requests.get(uniprot_api_url,params=uniprot_api_args).text
  uniprot_query_file = open("uniprot_sequences_query.fasta", "wt")
  uniprot_query_file.write(uniprot_query_seqs)
  uniprot_query_file.close()``` 
* Output: Archivo fasta con las secuencias de los resultados del query
### Blast +
* Input : Las secuencias de proteínas relacionadas a melanoma y el proteoma humano en formato multifasta.
* Uso de la version 2.2.26+ (https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.2.26/ncbi-blast-2.2.26+-x64-win64.tar.gz)
  * Primero se uso el ejecutable makeblastdb.exe para generar la base de datos en base al proteoma humano.
  * Luego se uso el ejecutable blastp.exe para realizar el alineamiento de las proteínas relacionadas a melanoma.
* El código usado puede ser directamente a tráves de una consola linux (blastp_align.sh) o el notebook de Python
* Output :
  * Base de datos con datos con prefijo Homo_sapiens 
  *Archivo tsv obtenido mediante blastp 

### Interproscan
* Input : String que contiene las secuencias que tuvieron un alineamiento mediante blastp
* Código : 
  ```submit_data = {"email":"ana.romani1@unmsm.edu.pe",
               "title":"proteins_melanoma",
               "goterms":"false",
               "pathways":"false",
               "stype":"p",
               "sequence":query_str}
  submit_url   = "https://www.ebi.ac.uk/Tools/services/rest/iprscan5/run"
  progress_url = "https://www.ebi.ac.uk/Tools/services/rest/iprscan5/status"
  results_url  = "https://www.ebi.ac.uk/Tools/services/rest/iprscan5/result"
  submit_headers   = {"Accept":"text/plain"}
  progress_headers = {"Accept":"text/plain"}
  results_headers  = {"Accept":"text/tab-separated-values"}
  submit_request = requests.post(submit_url,data=submit_data,headers=submit_headers)
  results_log_request = requests.get(results_url+"/"+submit_job_id+"/log",headers=results_headers)
  results_tsv_request = requests.get(results_url+"/"+submit_job_id+"/tsv",headers=results_headers)
  results_tsv_str = StringIO(results_tsv_request.text)
  results_column_names = ["sequence","md5","length","database","accession","description","start","end","evalue","post_processed","date","entry","name"]
  results_df = pd.read_csv(results_tsv_str,sep="\t",names=results_column_names)
  ```
* Output : Tabla con las predicciones de los dominios 

<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Licencia Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br />Esta obra está bajo una <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Licencia Creative Commons Atribución-NoComercial 4.0 Internacional</a>.
