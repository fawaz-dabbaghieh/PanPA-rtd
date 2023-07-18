Subcommands
===========

**PanPA** is separated into 3 subcommands that can be run independently:

1. Building GFA graphs from set of input MSAs with ``build_gfa``
2. Indexing the set of MSAs which corresponds to the set of produced GFAs with ``build_index``
3. Aligning query DNA or AA sequences back to the graphs produced with ``align`` 

Building GFAs
-------------
In this step, PanPA can take as input a directory with MSAs in FASTA format, a text file with a list of input MSAs with each file in one line, or simply input MSAs as command arguments and outputs the corresponding graphs of each MSA in an output directory. The idea here is that there's a 1 to 1 correspondence between the MSAs and the graphs, so later on, the index built from the MSAs can be used to also match against the graphs.

.. code-block::
	:caption: build_gfa input arguments

	usage: PanPA build_gfa [-h] [-f IN_FILES [IN_FILES ...]] [-l IN_LIST] [-d IN_DIR] [-c CORES] [-o OUT_DIR]

	optional arguments:
	  -h, --help			show this help message and exit
	  -f IN_FILES [IN_FILES ...], --fasta_files IN_FILES [IN_FILES ...]
							Input MSA(s) in fasta format, one or more file space-separated
	  -l IN_LIST, --fasta_list IN_LIST
							a text file with all input MSAs paths each on one new line
	  -d IN_DIR, --in_dir IN_DIR
							Directory path containing one or more amino acid MSA in FASTA format (gzipped allowed)
	  -c CORES, --cores CORES
							Numbers of cores to use for aligning
	  -o OUT_DIR, --out_dir OUT_DIR
							Output directory where the index files and graphs from the MSAs are stored


Indexing MSAs
-------------
The subommands ``build_index`` takes the set of MSAs as input, and for each sequence in the MSA, seeds are extracted, the user can specify two types of seeds here, 1) *k*-mers or 2) *(w,k)*-minimizers using the argument ``--seeding_alg`` which takes either ``k_mers`` or ``wk_min``, then
the user can to specify the *k* size with ``-k, --kmer_size`` and w size with ``-w, --window`` or keep the default values. The user also needs to give an output file name/location.

The ``--seed_limit`` argument takes an integer, which specifies a limit to how many MSAs (or graphs) can one seed belong to.
E.g. one *k*-mer can be present in all MSAs given, the user can specify a limit on that, and the matches are ordered
based on how many times that seed was present in that MSA and the top ``n`` will be taken. If the user chooses to keep all hits, then ``0`` is given to this argument and all seed hits will be kept in the index, i.e. with ``0``, all matching MSAs/Graphs will be used for alignments

.. code-block::
	:caption: build_index input arguments

	usage: PanPA build_index [-h] [-f IN_FILES [IN_FILES ...]] [-l IN_LIST] [-d IN_DIR] [-o OUT_INDEX]
									  [--seeding_alg SEEDING_ALG] [-k K-MER] [-w WINDOW] [--seed_limit SEED_LIMIT]

	optional arguments:
	  -h, --help			show this help message and exit
	  -f IN_FILES [IN_FILES ...], --fasta_files IN_FILES [IN_FILES ...]
							Input MSA(s) in fasta format, one or more file space-separated
	  -l IN_LIST, --fasta_list IN_LIST
		  				  a text file with all input MSAs paths each on one new line
	  -d IN_DIR, --in_dir IN_DIR
							Directory path containing one or more amino acid MSA in FASTA format (gzipped allowed)
	  -o OUT_INDEX, --out_index OUT_INDEX
							The output index file name
	  --seeding_alg SEEDING_ALG
							Seeding algorithm. Choices: k_mers, wk_min. Default: k_mers
	  -k K-MER, --kmer_size K-MER
							K-mer size for indexing the sequencing. Default: 5
	  -w WINDOW, --window WINDOW
							Window size when using w,k-minimizers instead of k-mers for indexing. Default:8
	  --seed_limit SEED_LIMIT
							Indicates how many graphs can a seed belong to. Default: 5, give 0 for no limit

Align Query Sequences
---------------------
For aligning query sequences to the graphs, you need to give three main inputs to the subcommand ``align``:

1- The index that was built with ``build_index`` subcommand
2- The input graphs that were built from the same set of MSAs that were used for building the index, the set of grpahs which can be a directory, a text file with list, or given directly in the command.
3- The query sequences in FASTA. If DNA sequences are given, then the user needs to use the flag ``--dna`` which will then run the frameshift aware alignment algorithm on both the forward and reverse complement of each DNA query sequence.

The user can also specify the substitution matrix to use for the alignment, or print a list of possible matrices with ``--sub_matrix_list``. The user can also specify a certain gap score with ``--gap_score``, a cutoff on alignment id with ``--min_id_score``, and can set a limit to how many graphs to align to with ``--seed_limit``.
This step can be made faster by giving more cores.
If DNA sequences were aligned to graphs that were build from DNA sequences, then please read about this in the Other Info section.

The output alignment are in GAF format. To learn more about this format please check `here <https://github.com/lh3/gfatools/blob/master/doc/rGFA.md>`_, moreover, the Other Info section has some extra information about the output file.

.. code-block::
	:caption: align input arguments

	usage: PanPA align [-h] [-g IN_FILES [IN_FILES ...]] [-l IN_LIST] [-d GRAPHS] [--index INDEX]
					   [-r SEQS] [--dna] [-c CORES] [--sub_matrix SUB_MATRIX] [--sub_matrix_list]
					   [-o GAF] [--gap_score GAP_SCORE] [--min_id_score MIN_ID_SCORE]
					   [--seed_limit SEED_LIMIT]

	options:
	  -h, --help			show this help message and exit
	  -g IN_FILES [IN_FILES ...], --gfa_files IN_FILES [IN_FILES ...]
							Input GFA graphs, one or more file space-separated
	  -l IN_LIST, --gfa_list IN_LIST
							a text file with all input graphs paths each on one new line
	  -d GRAPHS, --in_dir GRAPHS
							Path to directory with GFA files
	  --index INDEX		 Path to pickled index file generated in the build step
	  -r SEQS, --seqs SEQS  The input sequences to align in fasta format
	  --dna				 Give this flag if the query sequences are DNA and not AA
	  -c CORES, --cores CORES
							Numbers of cores to use for aligning
	  --sub_matrix SUB_MATRIX
							Substitution matrix to use for alignment, default: blosum62
	  --sub_matrix_list	 When given, a list of possible substitution matrices will be given
	  -o GAF, --out_gaf GAF
							Output alignments file path
	  --gap_score GAP_SCORE
							The gap score to use for the alignment, default: -3
	  --min_id_score MIN_ID_SCORE
							minimum alignment identity score for the alignment to be outputted, [0,1]
	  --seed_limit SEED_LIMIT
							How many graphs can each seed from the query sequence have hits to,
							default: 3


Align to Single Graph
---------------------
The user can avoid the extracting seeds step when aligning sequences if the user wants to align to a specific target graph by using the ``align_single`` subcommand.

In this subcommand, the user needs to provide a target graph in GFA format, the graph has to be a DAG (directed and acyclic), and query sequences in FASTA format.

.. code-block::
	:caption: align to single target

	usage: PanPA align_single [-h] [-g IN_GRAPH] [-r SEQS] [--dna] [-c CORES] [--sub_matrix SUB_MATRIX]
							  [--sub_matrix_list] [-o GAF] [--gap_score GAP_SCORE]
							  [--min_id_score MIN_ID_SCORE]

	options:
	  -h, --help			show this help message and exit
	  -g IN_GRAPH, --gfa_files IN_GRAPH
							Input GFA graph to align against
	  -r SEQS, --seqs SEQS  The input sequences to align in fasta format
	  --dna				 Give this flag if the query sequences are DNA and not AA
	  -c CORES, --cores CORES
							Numbers of cores to use for aligning
	  --sub_matrix SUB_MATRIX
							Substitution matrix to use for alignment, default: blosum62
	  --sub_matrix_list	 When given, a list of possible substitution matrices will be given
	  -o GAF, --out_gaf GAF
							Output alignments file path
	  --gap_score GAP_SCORE
							The gap score to use for the alignment, default: -3
	  --min_id_score MIN_ID_SCORE
							minimum alignment identity score for the alignment to be outputted, [0,1]
