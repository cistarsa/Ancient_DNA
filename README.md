# Ancient_DNA
Determination of allele frequency change, among other popgen features, over time in CPB

## on CHTC, use wegmann lab [ATLAS](https://bitbucket.org/wegmannlab/atlas/wiki/Installing%20and%20Running%20ATLAS)
### also, build and push Docker image for portability:
  on mac:
  ```bash
    Zachary-Cohens-MacBook-2:ATLAS zacharycohen$ docker build -t kingcohn1/atlas:wgmlab .
    Zachary-Cohens-MacBook-2:ATLAS zacharycohen$ sudo docker push kingcohn1/atlas:wgmlab
```
Dockerfile:
```
FROM ubuntu:18.04 AS builder

# Install Dependencies
RUN apt-get update && apt-get -y install \
    cmake \
    g++ \
    gcc \
    git \
    libarmadillo-dev \
    libblas-dev \
    libboost-dev \
    liblapack-dev

WORKDIR /tmp
RUN git clone --depth 1 https://bitbucket.org/WegmannLab/atlas.git && \
    cd atlas && make

FROM ubuntu:18.04

LABEL description="Atlas ancient DNA toolkit container" \
      author="Mahesh Binzer-Panchal" \
      version="0.9.9" \
      tool_author="Vivian Link, Athanasios Kousathanas, Krishna Veeramah, Christian Sell, Amelie Scheu, Daniel Wegmann" \
      doi="https://doi.org/10.1101/105346"

RUN apt-get update && apt-get -y install \
    libarmadillo-dev \
    libblas-dev \
    libboost-dev \
    liblapack-dev

COPY --from=builder /tmp/atlas/atlas /usr/local/bin/atlas

CMD [ "atlas" ]

```

for large data sets:
### STEP1
```
#!/bin/bash

## use bwa-mem2 precompiled binaries
curl -L https://github.com/bwa-mem2/bwa-mem2/releases/download/v2.0pre2/bwa-mem2-2.0pre2_x64-linux.tar.bz2 \
  | tar jxf -

#./bwa-mem2-2.0pre2_x64-linux/bwa-mem2 index Curated_Assembly.fa

ls -lhs
#tar -xzf 1*tgz

tar -xzf atlas_2.tgz

tar -xzf curated*tgz

mv curated_run/* ./
ls -lhs

for f in `ls -d 1* | sed 's/^1_//g'`; do for z in 1_"$f"/*R1*; do for j in 1_"$f"/*R2*; do if [[ "$z" ==  *"$f"* ]] && [[ "$j" == *"$f"* ]]; then for header in `zcat $z | head -n 1`; do for id in `echo $header | head -n 1 | cut -f 1-4 -d":" | sed 's/@//' | sed 's/:/_/g'`; do for sm in `echo $header | head -n 1 | grep -Eo "[ATGCN]+$"`; do bwa-mem2-2.0pre2_x64-linux/bwa-mem2 mem -R "@RG\tID:$f\tSM:$id"_"$sm\tLB:$id"_"$sm\tPL:ILLUMINA" Curated_Assembly.fa "$z" "$j" > "$f".sam; done; done; done;fi; done; done ; done


#sed -ie 's/\\t/\t/g' *sam

#git clone https://github.com/samtools/htslib

#git clone https://github.com/samtools/samtools
#cd samtools
#make
#cp samtools ../samtools_exec

#cd ../

./samtools view -S -b *sam > "$f".bam

./samtools sort *bam -o sorted_"$f".bam

./samtools index sorted_"$f".bam

./samtools view -H sorted*bam | grep "@RG" | awk '{print $2}' | grep "^ID:I" | sed 's/ID://g' | sort -u >> rgs_"$f".txt

mkdir "$f"_bams_atlas
```

