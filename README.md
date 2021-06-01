# Codger

Codger is a toolkit for dealing with the explicit codon alignments produced by [DVORFS](https://github.com/ilevantis/dvorfs).

Explicit codon alignments are fasta alignment files where the first record/row is named "CODONS" and specified the beginning of each codon in the alignment using either "N" or "n". "N" marks a "reference" codon (i.e. a column that is present in the consensus/reference sequences or likely informative). "n" marks a non-reference codon (i.e. an insertion compared with the consensus/reference sequences or uninformative etc...). Each codon must be padded to a width that is a multiple of 3.

## Install

```shell
cd ~/bin
wget https://raw.githubusercontent.com/ilevantis/codger/master/codger
chmod +x codger
```

**dependencies:** python version >= 3.3

## Example Usage

Run dvorfs on chromosome 1 of human genome to look for ERV RT domains using pHMMs from GyDB.

Then align the resulting sequences by translating to amino acids, aligning using using MAFFT, and realigning the nucleotide sequences according to this amino acid alignment.

Finally, trimming out columns which have too many ambiguous codons.

Note: Use "-" to specify input from STDIN.

```shell
dvorfs -f GRCh38.p13_chr1.fasta --hmm2 GyDB_RTs.hmm2 -p 24 --filter no-overlap

codger translate dvorfs.alis/*.ali.fa \
| mafft-einsi \
> chr1_RTs.ali.pep.fa

codger aaalign -a chr1_RTs.ali.pep.fa dvorfs.alis/*.ali.fa \
| codger setref --remove-ambigs -p 0.75 - \
| codger trim - \
> chr1_RTs.trimmed.ali.fa

```

The format for the explicit codon alignment is designed to make it easy to manually edit in GUI alignment viewers/editors such as [AliView](http://www.ormbunkar.se/aliview/). A number of workflows with particularly disrupted EVEs may entail manually adjusting an alignment and manually editing codon-columns. i.e. editing the "CODONS" entry to manually set which codon-columns are marked as reference/non-reference, or adjusting the width of existing codon-columns, or adding in new codon-columns.


## codger translate
Translate and combine a single or multiple codon alignment files into an amino acid alignment.
When multiple codon aligned fastas are given, sequences in the output AA alignment will not be
properly aligned to all other sequences.
```
usage: codger translate [-h] [-o OUT] [-e] cdn_fastas [cdn_fastas ...]
  cdn_fastas            One or more explicit codon aligned fastas.
  -o OUT, --out OUT     Output file name.
  -e, --extended-chars  Tranlsate frameshifted codons into specific characters:
                        ? - bp deletion; & - bp insertion; ! - multi bp frameshifting
                        insertion.
```


## codger aaalign
Align multiple codon alignment files based on a corresponding AA alignment.
```
usage: codger aaalign [-h] [-o OUT] -a AA cdn_fastas [cdn_fastas ...]
  cdn_fastas         One or more explicit codon aligned fastas.
  -o OUT, --out OUT  Output file name.
  -a AA, --aa AA     Amino acid alignment corresponding to the codon alignment files.
```


## codger setref
Modify the reference codon-column row based on percent of ambiguous codons in a column or
based on reference sequences.
```
usage: codger setref [-h] [-o OUT] [-r REFS] [-k] [-a] [-p AMBIG_PC] cdn_fasta
  cdn_fasta             An explicit codon aligned fasta.
  -o OUT, --out OUT     Output file name.
  -r REFS, --refs REFS  A file listing sequence IDs (one per line) of sequences to be used as
                        reference sequences in the alignment. A fasta file can also be used,
                        in which case all IDs in the provided fasta file determine reference
                        sequences.
  -k, --keep-refs       All codon-columns containing a codon in a reference sequence are
                        marked as reference codon-columns.
  -a, --remove-ambigs   Codon-columns cotaining higher than X proportion of ambiguous codons
                        are marked as non-reference. If --refs is given, proportion is
                        calculated excluding reference sequence rows.
  -p AMBIG_PC, --ambig-pc AMBIG_PC
                        Threshold proportion of ambiguous codons to mark codon-column as non-
                        reference. Default: 1.0
```


## codger trim
Remove codon-columns marked as non-reference from the alignment.
```
usage: codger trim [-h] [-o OUT] cdn_fasta
  cdn_fasta          An explicit codon aligned fasta.
  -o OUT, --out OUT  Output file name.
```


## codger chop
Chop codon-columns wider than 3bp down to size. Codons wider than 3bp in a codon-column will
be replaced with 'NNN'.
```
usage: codger chop [-h] [-o OUT] cdn_fasta
  cdn_fasta          An explicit codon aligned fasta.
  -o OUT, --out OUT  Output file name.
```


## codger thin
Remove all sequences containing fewer than P codons marked as reference codons in the
alignment.
```
usage: codger thin [-h] [-o OUT] -l LENGTH cdn_fasta
  cdn_fasta              An explicit codon aligned fasta.
  -o OUT, --out OUT      Output file name.
  -l L, --length LENGTH  Minimum number of codons marked as reference for a sequence to be
                         kept.
```


## codger filter
Exclude or include sequences based on their sequence ID.
```
usage: codger filter [-h] [-o OUT] --ids IDS (-i | -e) cdn_fasta
  cdn_fasta          An explicit codon aligned fasta.
  -o OUT, --out OUT  Output file name.
  --ids IDS          A file listing sequence IDs (one per line) to include or exclude from the
                     alignment. A fasta file can also be used, in which case all IDs in the
                     provided fasta file are used as IDs to include/exclude.
  -i, --include      Only sequence IDs in the ID file are included in the output.
  -e, --exclude      Sequence IDs in the ID file are excluded in the output.
```
