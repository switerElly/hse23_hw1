# hse23_hw1

### Создаю ссылки и выбираю случайные чтения

```bash
ls /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}

seqtk sample -s629 oil_R1.fastq 5000000 > sub1.fastq
seqtk sample -s629 oil_R2.fastq 5000000 > sub2.fastq
seqtk sample -s629 oilMP_S4_L001_R1_001.fastq 1500000 > matep1.fastq
seqtk sample -s629 oilMP_S4_L001_R2_001.fastq 1500000 > matep2.fastq
```

### Оцениваю файлы через fastqc и делаю отчет с multiqc

```bash
mkdir fastqc
mkdir multiqc
ls sub* matep* | xargs -tI{} fastqc -o fastqc {}
multiqc -o multiqc fastqc
```

### Обрезаю чтения с platanus

```bash
platanus_trim sub*
platanus_internal_trim matep*
```

### Оцениваю обрезанные файлы через fastqc и делаю отчет с multiqc

```bash
mkdir fastqc_trimmed
mkdir multiqc_trimmed
ls sub* matep*| xargs -tI{} fastqc -o fastqc_trimmed {}
multiqc -o multiqc_trimmed fastqc_trimmed
```

### Перехожу в screen и запускаю сбор контиг

```bash
screen
time platanus assemble -o Poil -t 4 -f sub1.fastq.trimmed sub2.fastq.trimmed 2> assemble.log
```

### Перехожу в screen и запускаю сбор скаффолдов

```bash
screen
time platanus scaffold -o Poil -t 4 -c Poil_contig.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matep1.fastq.int_trimmed matep2.fastq.int_trimmed 2> scaffold.log
```

### Перехожу в screen и уменьшаю число промежутков

```bash
screen
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matep1.fastq.int_trimmed matep2.fastq.int_trimmed 2> gapclose.log
```

### Скачиваю файлы через scp, например вот так:

```bash
scp -i bioinf -P 5222 aavarfolomeeva@89.175.46.92:~/multiqc/multiqc_report.html .
```

### Записываю в отдельный файл последовательность самого длинного скаффолда

```bash
more Poil_scaffold.fa
echo scaffold1_len3833690_cov231 > tmp2.txt
seqtk subseq Poil_scaffold.fa tmp2.txt > scaf_long.fasta
```

### Записываю в отдельный файл последовательность самого длинного скаффолда

```bash
more Poil_gapClosed.fa
echo scaffold1_cov231 > _tmp.txt
seqtk subseq Poil_gapClosed.fa _tmp.txt > longest.fasta
```
