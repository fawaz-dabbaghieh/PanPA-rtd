Other Info
===========
Some extra information about using **PanPA** and the output GAF format.

GAF Format
----------
The GAF (Graph Alignment Format) was described here `here <https://github.com/lh3/gfatools/blob/master/doc/rGFA.md>`_. For **PanPA**, a couple more tags were added to keep all important information related to the alignments.

When aligning amino acid sequences against amino acid graphs, the tag ``gid`` gives the target graph where the query sequenced aligned.

when aligning DNA sequences using the argument ``--dna``, where the frameshift aware alignment algorithm is used, another tag is added ``DNA`` which has two values, froward and reverse, indicating if the DNA query sequence was aligned in the forward or reverse complement direction.

DNA to DNA
----------
PanPA was mainly designed to align amino acid and DNA sequences against amino acid graphs. It is possible though to also generate the index and graphs from MSA sequences in DNA alphabet. However, when aligning DNA query sequences back to these graphs, you do not need to use the ``--dna`` argument, as this argument tries to align DNA against amino acids and uses the frameshift aware alignment algorithm. However, you just need to omit the ``--dna`` argument, and use ``--sub_matrix dna`` which is basically edit distance (1 for a match, 0 for mismatch). Therefore, you might want to also change the gap penalty with ``--gap_score``` argument. In this mode however, **PanPA** will not try to also align the reverse complement, so the user might want to provide this in the query sequence FASTA file.

For example ``PanPA align -d graphs_from_dna_seqs/ -r query_dna_sequences.fasta -o query_dna_sequences.gaf --sub_matrix dna --gap_score -1 -c 10``
