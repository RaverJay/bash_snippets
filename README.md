# bash_snippets
bash scripts for daily use

### counts
    #!/bin/bash
    # output occurences in descending order
    # SK
    sort <&0 | uniq -c | sort -nr
    
### texit
    #!/bin/bash
    # pdflatex either all .tex files or only <file>[.[tex]]

    if [ $# == 0 ]; then
        T=(*.tex)
        if [ "${T[0]}" == "*.tex" ]; then
            echo 'No .tex files here!' >&2
            exit 1
        fi
    else
        T=("${1%.*}.tex")
        if [ ! -f "$T" ]; then
            echo "No file named $T here!" >&2
            exit 1
        fi
    fi
    for F in ${T[@]}; do
        B=${F%.*}
        pdflatex "$B"
        if [ -f "${B}.aux" ]; then
            bibtex "$B" && pdflatex "$B"
        fi
        pdflatex "$B"
    done
    
### bamify
    #!/bin/bash
    # bamify - select mapped reads, sort, convert to bam and index
    # by Sebastian Krautwurst

    if [ -z "$1" ]
    then
        echo "No input sam file(s) given!"
        exit
    fi

    while [ ! -z "$1" ]
    do
        echo -n "$1"
        BAM="${1%.sam}".bam
        samtools view -h -F4 "$1" | samtools sort | samtools view -hb > "$BAM" && samtools index "$BAM"
        echo " bamified to $BAM"
        shift
    done
    
### plotify
    #!/bin/bash
    # plotify - minimap2 some reads ($2,$3,...) to some reference ($1) and plot the coverage
    # by Sebastian Krautwurst

    if [ -z "$1" ]
    then
        echo "No input given!"
        exit 1
    elif [ -z "$2" ]
    then
        echo "No input reads given!"
        exit 1
    fi

    REF="$1"
    shift

    while [ ! -z "$1" ]
    do
        echo "Plotify $1 ..."
        BN=$(basename "$1")
        BAM="${BN}".bam
        minimap2 -ax map-ont "$REF" "$1" | samtools view -h -F4 | samtools sort | samtools view -hb > "$BAM" && samtools index "$BAM"
        echo "Mapped $1 and bamified to $BAM"
        plotCoverageExons.py -o "${BAM%.*}.pdf" "$BAM"
        echo "Plotted coverage to ${BAM%.*}.pdf"
        shift
    done
