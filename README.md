# hse23_hw1

## Часть работы в консоли

### Создаю ссылки и выбираю случайные чтения

```bash
ls /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}

seqtk sample -s629 oil_R1.fastq 5000000 > S1.fastq
seqtk sample -s629 oil_R2.fastq 5000000 > S2.fastq
seqtk sample -s629 oilMP_S4_L001_R1_001.fastq 1500000 > M1.fastq
seqtk sample -s629 oilMP_S4_L001_R2_001.fastq 1500000 > M2.fastq
```

### Оцениваю файлы через fastqc и делаю отчет с multiqc

```bash
mkdir fastqc
mkdir multiqc
ls S* M* | xargs -tI{} fastqc -o fastqc {}
multiqc -o multiqc fastqc
```

### Обрезаю чтения с platanus

```bash
platanus_trim S*
platanus_internal_trim M*
```

### Оцениваю обрезанные файлы через fastqc и делаю отчет с multiqc

```bash
mkdir fastqc_trimmed
mkdir multiqc_trimmed
ls S* M*| xargs -tI{} fastqc -o fastqc_trimmed {}
multiqc -o multiqc_trimmed fastqc_trimmed
```

### Перехожу в screen и запускаю сбор контиг

```bash
screen
time platanus assemble -o Poil -t 4 -f S1.fastq.trimmed S2.fastq.trimmed 2> assemble.log
```

### Перехожу в screen и запускаю сбор скаффолдов

```bash
screen
time platanus scaffold -o Poil -t 4 -c Poil_contig.fa -IP1 S1.fastq.trimmed S2.fastq.trimmed -OP2 M1.fastq.int_trimmed M2.fastq.int_trimmed 2> scaffold.log
```

### Перехожу в screen и уменьшаю число промежутков

```bash
screen
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 S1.fastq.trimmed S2.fastq.trimmed -OP2 M1.fastq.int_trimmed M2.fastq.int_trimmed 2> gapclose.log
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


## Часть с кодом

### Контиги

```bash
f = open("contigs.fasta")
a = f.readlines()
n = ln = 0
lng = int(a[0][a[0].find('len') + 3:a[0].rfind('_')])
lst = []
for i in a:
    if '>' in i:
        n += 1
        ln += int(i[i.find('len') + 3:i.rfind('_')])
        ltmp = int(i[i.find('len') + 3:i.rfind('_')])
        lst.append(ltmp)
        if ltmp > lng:
          lng = ltmp
lst.sort()
lst.reverse()
summ = j = ind =  0
while summ <= ln/2:
    summ += lst[j]
    ind = j
    j+=1

print(f'Количество контигов: {n}')
print(f'Длина контигов: {ln}')
print(f'Самый длинный контиг: {lng}')
print(f'N50: {lst[ind]}')
```
Количество контигов: 601

Длина контигов: 3923107

Самый длинный контиг: 179307

N50: 47993

### Скаффолды

```bash
import re
f = open("scaffolds.fasta")
a = f.readlines()
n = ln = mx = sm = 0
lng = a[0].split('len')[1].split('_cov')[0]
lst = []
for i in a:
    if '>' in i:
        n += 1
        ln += int(i[i.index('len') + 3:i.index('_cov')])
        ltmp = int(i[i.index('len') + 3:i.index('_cov')])
        lst.append(ltmp)
    for j in i:
        if j == "N":
            sm += 1
lst.sort()
lst.reverse()
summ = j = ind =  0
while summ <= ln/2:
    summ += lst[j]
    ind = j
    j+=1

print(f'Количество скаффолдов: {n}')
print(f'Длина скаффолдов: {ln}')
print(f'Самый длинный скаффолд: {lng}')
print(f'N50: {lst[ind]}')
print(f'Общая длина гэпов до подрезания: {sm}')
```
Количество скаффолдов: 67

Длина скаффолдов: 3872516

Самый длинный скаффолд: 3833690

N50: 3833690

Общая длина гэпов до подрезания: 6851

### Скаффолды после удаления гэпов

```bash
f = open("after_gap.fa")
a = f.readlines()
sm = 0
for i in a:
    for j in i:
        if j == "N":
            sm += 1
print(f'Общая длина гэпов после подрезания: {sm}')
```
Общая длина гэпов после подрезания: 1530

### Количество гэпов для самого длинного скаффолда до подрезания

```bash
!grep 'N' scaffolds.fasta | wc -l
```

### Количество гэпов для самого длинного скаффолда после подрезания

```bash
!grep 'N' after_gap.fa | wc -l
```

## Отчеты из Multiqc

##### У меня немного поломался отчет, так как изначальные файлы, с которыми я работала не до конца проанализировались, и я запистила все на новых. Поэтому то, что нас интересует это S1, S2, M1, M2, остальные не нужны.

## для исходных чтений
![Image alt](https://github.com/switerElly/hse23_hw1/blob/main/imges/Screenshot%20from%202023-10-08%2017-41-38.png)
![Image alt](https://github.com/switerElly/hse23_hw1/blob/main/imges/fastqc_adapter_content_plot%20(1).png)
![Image alt](https://github.com/switerElly/hse23_hw1/blob/main/imges/fastqc_per_sequence_quality_scores_plot%20(1).png)

## для урезанных чтений
![Image alt](https://github.com/switerElly/hse23_hw1/blob/main/imges/Screenshot%20from%202023-10-08%2017-43-13.png)
![Image alt](https://github.com/switerElly/hse23_hw1/blob/main/imges/fastqc_adapter_content_plot%20(2).png)
![Image alt](https://github.com/switerElly/hse23_hw1/blob/main/imges/fastqc_per_sequence_quality_scores_plot%20(2).png)
