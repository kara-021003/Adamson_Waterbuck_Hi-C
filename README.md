# Adamson_Waterbuck_Hi-C
Reproducible Workflow for Hi-C Variant Calling and Variant Heatmap Visualisation

Step 1: Generate a 1kb H5 file

Follow https://github.com/Farre-lab/Kirkland_Bovidae/tree/main/Hi-C workflow
Modify BinSize to 1kb in: https://github.com/Farre-lab/Kirkland_Bovidae/blob/main/Hi-C/4_HiCExplorer1.sh 
To generate a 1kb Hi-C matrix


Step 2: Perform Normalization of 1kb H5 file

Follow: https://github.com/Farre-lab/Kirkland_Bovidae/blob/main/Hi-C/5_HiCExplorer2.sh


Step 3: Perform Correction of Normalized 1kb H5 file

Follow: https://github.com/Farre-lab/Kirkland_Bovidae/blob/main/Hi-C/6_HiCExplorer3.sh


Step 4: Convert Normalized, Corrected H5 file to 1kb COOL file

hicConvertFormat --matrices {normalized_corrected.h5} --outFileName {output_1Kb.cool}  --inputFormat h5 --outputFormat cool


Step 5: Convert 1kb COOL file to MCOOL file with cooler zoomify

cooler zoomify {output_1kb.cool} --resolutions 1000,5000,10000,25000,50000,100000,250000,500000,1000000,2500000 -o {output.mcool}


Step 6: Perform Cooler Balance on MCOOL file

cooler balance {output.mcool}::/resolutions/1000
Repeat for the following resolutions: 5000, 10000, 25000, 50000, 100000, 250000, 500000, 1000000 and 2500000 (These are the same resolutions generated in the mcool file)


Step 7: Structural Variant Calling with EagleC2
. This step generates a structural variant call file (`SV_calls.txt`) and a log file.

predictSV --mcool {output.mcool} --resolutions 50000,100000,250000,500000 --prob-cutoff-1 0.3 --prob-cutoff-2 0.3 -O {file_name} -g other --balance-type ICE -p 8 --intra-extend-size 1,1,1,1 --inter-extend-size 1,1,1,1


Step 8: Hi-C Variant Heatmap Visualisation 
. Generates heatmap of input Hi-C variant coordinates

plot-SVbreaks --cool-uri {output.mcool}::resolutions/250000 \
                --balance-type ICE --breakpoint-coords chr1,pos1,chr2,pos2 \
                --window-width 5 -O {output.png} --dpi 800


Step 9 (OPTIONAL): Visualising Heatmap of Whole Chromosome Interactions 
. Firstly, a 500kb resolution h5 file needs to be generated - this follows the following script exactly: https://github.com/Farre-lab/Kirkland_Bovidae/blob/main/Hi-C/4_HiCExplorer1.sh

. Visualising Whole Chromosome Interactions - Heatmap Generation:

hicPlotMatrix -m {500kb.h5} --dpi 800 --chromosomeOrder chr1 chr2 -o {output.png} --log1p

OR optionally

plot-interSVs \
  --cool-uri {output.mcool}::resolutions/500000 \
  --sv-file {SV_calls.txt} \
  -C chr1 chr2 \
  -O {output.png} \
  --balance-type ICE \
  --dpi 800