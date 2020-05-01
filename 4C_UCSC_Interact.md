Create UCSC Interact Files Colored by Viewpoint
================

The UCSC interact format is a compact notation to display chromatin interaction data directly in the genome browser. Arcs are used to represent interactions that appear within the view. A squared connector represents interactions on the same chromosome but outside of the view. It is possible to represent interchromosomal interactions, these appear as a short connector labeled with the name of the interacting chromosome- coordinates can be found by clicking on the connector.

The example provided by UCSC shows 4 arcs representing interacting loci (given as 1bp points) with different colors and values.

    track type=interact name="interact Example One" description="An interact file" interactDirectional=true maxHeightPixels=200:100:50 visibility=full
    browser position chr12:40,560,500-40,660,499 
    #chrom  chromStart  chromEnd  name  score  value  exp  color  sourceChrom  sourceStart  sourceEnd  sourceName  sourceStrand  targetChrom  targetStart  targetEnd  targetName  targetStrand
    chr12    40572709      40618813        rs7974522/LRRK2/muscleSkeletal  0       0.624   muscleSkeletal  #7A67EE  chr12 40572709        40572710  rs7974522       .       chr12   40618812        40618813  LRRK2   +
    chr12    40579899      40618813        rs17461492/LRRK2/muscleSkeletal  0       0.624   muscleSkeletal  #7A67EE chr12 40579899        40579900  rs17461492       .       chr12   40618812        40618813  LRRK2   +
    chr12    40614433      40618813        rs76904798/LRRK2/nerveTibial  0       0.625   nerveTibial  #FFD700       chr12 40614433        40614434  rs76904798       .       chr12   40618812        40618813  LRRK2   +
    chr12    40618812      40652520        rs2723264/LRRK2/lung  0       1.839   lung     #9ACD32  chr12      40652519        40652520  rs2723264       .       chr12   40618812        40618813  LRRK2   +

The first version of an interact file that I made to represent the WT mouse E11.5 face 4C data used 1bp loci, the midpoint of the restriction fragment used as the viewpoint and the midpoint of the interaction given at 10kb resolution. A more accurate (but maybe not as streamlined) visualization is to use the full coordinates.

I provide examples for both from my own data below. The examples assume that you are interested in plotting interactions on the same chromosome as your viewpoints and the bed file of interactions are already filtered for significant interactions on the same chromosome. No method of assigning score or value is given for the examples.

1bp loci example
----------------

### 1. Make a viewpoint\_format.txt file

The columns are:
viewpoint\_name midpoint color

    vp1 50940037    #FF0000
    vp2 50941402    #00FF04
    vp3 51171223    #FF00F4 
    vp4 51773845    #0C8900
    vp5 52009462    #1500FF
    vp6 52204597    #803582

Midpoint is the midpoint of the restriction fragment used as the viewpoint. An alternative to midpoint plotting is given later. The colors can be in RGB, hexadecimal or html color name. If given in hexadecimal it must be preceeded by the pound sign. Make sure there are no headers or extra lines at the top and bottom.

### 2. Write the scripts that make the interact files

Use the viewpoint\_format.txt file to plug in details about the viewpoints. Have the interaction data output from the 4C analysis in bed format. My data files happen to be named Face\_\*\_3C-seq\_lifted\_to\_mm10.bed, change that in line 3 if needed.

    #write_make_interact_files.sh
    cat viewpoint_format.txt | awk '{
            print "cat Face_"$1"_3C-seq_lifted_to_mm10.bed | awk \x27\{print \$1\"\\t\"\(\$2+\$3\)/2\"\\t"$2"\\t"$1"/3C-seq/Face\\t0\\t1\\tFacevBrain\\t"$3"\\t\"$1\"\\t"$2"\\t"$2+1"\\t"$1"\\t.\\t\"\$1\"\\t\"\(\$2+\$3\)/2\"\\t\"\(\(\$2+\$3\)/2\)+1\"\\t\"\$1\":\"\$2\"-\"\$3\"\\t.\"\}\x27 CONVFMT=\x27\%.0f\x27 > Face_3C\-seq_"$1"_mm10.tmp" \
            "\necho \x22track type=interact name=\\\"Face "$1"\\\" description=\\\"Significant interactions in E11.5 Face (E11.5 Brain control)\\\" interactDirectional=true maxHeightPixels=200:100:50 visibility=full\x22 > Face_3C\-seq_"$1"_mm10.interact" \
            "\necho \"browser position chr6:49,720,000-52,880,000\x22 >> Face_3C\-seq_"$1"_mm10.interact" \
            "\ncat Face_3C-seq_"$1"_mm10.tmp | awk \x27\{if \(\$2<\$3\) print \$0\; else print \$1\"\\t\"\$3\"\\t\"\$2\"\\t\"\$4\"\\t\"\$5\"\\t\"\$6\"\\t\"\$7\"\\t\"\$8\"\\t\"\$9\"\\t\"\$10\"\\t\"\$11\"\\t\"\$12\"\\t\"\$13\"\\t\"\$14\"\\t\"\$15\"\\t\"\$16\"\\t\"\$17\"\\t\x22\$18\}\x27 >> Face_3C\-seq_"$1"_mm10.interact"}' > make_interact_files.sh

This writes the make\_interact\_files.sh. Example for three viewpoints below:

    cat Face_vp1_3C-seq_lifted_to_mm10.bed | awk '{print $1"\t"($2+$3)/2"\t50940037\tvp1/3C-seq/Face\t0\t1\tFacevBrain\t#ff0000\t"$1"\t50940037\t50940038\tvp1\t.\t"$1"\t"($2+$3)/2"\t"(($2+$3)/2)+1"\t"$1":"$2"-"$3"\t."}' > Face_3C-seq_vp1_mm10.tmp
    echo "track type=interact name=\"Face vp1\" description=\"Significant interactions in E11.5 Face (E11.5 Brain control)\" interactDirectional=true maxHeightPixels=200:100:50 visibility=full" > Face_3C-seq_vp1_mm10.interact
    echo "browser position chr6:49,720,000-52,880,000" >> Face_3C-seq_vp1_mm10.interact
    cat Face_3C-seq_vp1_mm10.tmp | awk '{if ($2<$3) print $0; else print $1"\t"$3"\t"$2"\t"$4"\t"$5"\t"$6"\t"$7"\t"$8"\t"$9"\t"$10"\t"$11"\t"$12"\t"$13"\t"$14"\t"$15"\t"$16"\t"$17"\t"$18}' >> Face_3C-seq_vp1_mm10.interact

    cat Face_vp2_3C-seq_lifted_to_mm10.bed | awk '{print $1"\t"($2+$3)/2"\t50941402\tvp2/3C-seq/Face\t0\t1\tFacevBrain\t#00FF040\t"$1"\t50941402\t50941403\tvp2\t.\t"$1"\t"($2+$3)/2"\t"(($2+$3)/2)+1"\t"$1":"$2"-"$3"\t."}' > Face_3C-seq_vp2_mm10.tmp
    echo "track type=interact name=\"Face vp2\" description=\"Significant interactions in E11.5 Face (E11.5 Brain control)\" interactDirectional=true maxHeightPixels=200:100:50 visibility=full" > Face_3C-seq_vp2_mm10.interact
    echo "browser position chr6:49,720,000-52,880,000" >> Face_3C-seq_vp2_mm10.interact
    cat Face_3C-seq_vp2_mm10.tmp | awk '{if ($2<$3) print $0; else print $1"\t"$3"\t"$2"\t"$4"\t"$5"\t"$6"\t"$7"\t"$8"\t"$9"\t"$10"\t"$11"\t"$12"\t"$13"\t"$14"\t"$15"\t"$16"\t"$17"\t"$18}' >> Face_3C-seq_vp2_mm10.interact

    cat Face_vp3_3C-seq_lifted_to_mm10.bed | awk '{print $1"\t"($2+$3)/2"\t51171223\tvp3/3C-seq/Face\t0\t1\tFacevBrain\t#FF00F40\t"$1"\t51171223\t51171224\tvp3\t.\t"$1"\t"($2+$3)/2"\t"(($2+$3)/2)+1"\t"$1":"$2"-"$3"\t."}' > Face_3C-seq_vp3_mm10.tmp
    echo "track type=interact name=\"Face vp3\" description=\"Significant interactions in E11.5 Face (E11.5 Brain control)\" interactDirectional=true maxHeightPixels=200:100:50 visibility=full" > Face_3C-seq_vp3_mm10.interact
    echo "browser position chr6:49,720,000-52,880,000" >> Face_3C-seq_vp3_mm10.interact
    cat Face_3C-seq_vp3_mm10.tmp | awk '{if ($2<$3) print $0; else print $1"\t"$3"\t"$2"\t"$4"\t"$5"\t"$6"\t"$7"\t"$8"\t"$9"\t"$10"\t"$11"\t"$12"\t"$13"\t"$14"\t"$15"\t"$16"\t"$17"\t"$18}' >> Face_3C-seq_vp3_mm10.interact

### 3. Then run it

    chmod +x make_interact_files.sh

    ./make_interact_files.sh

This will result in interact format files. Example from vp1:

    track type=interact name="Face vp1" description="Significant interactions in E11.5 Face (E11.5 Brain control)" interactDirectional=true maxHeightPixels=200:100:50 visibility=full
    browser position chr6:49,720,000-52,880,000
    chr6    50940037    50955213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50940037    50940038    vp1 .   chr6    50955213    50955214    chr6:50950213-50960213  .
    chr6    50940037    50945213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50940037    50940038    vp1 .   chr6    50945213    50945214    chr6:50940213-50950213  .
    chr6    50940037    51165213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50940037    50940038    vp1 .   chr6    51165213    51165214    chr6:51160213-51170213  .
    chr6    50940037    50975213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50940037    50940038    vp1 .   chr6    50975213    50975214    chr6:50970213-50980213  .
    chr6    50924860    50940037    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50940037    50940038    vp1 .   chr6    50924860    50924861    chr6:50919860-50929860  .
    chr6    50940037    50965213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50940037    50940038    vp1 .   chr6    50965213    50965214    chr6:50960213-50970213  .
    chr6    50940037    50985213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50940037    50940038    vp1 .   chr6    50985213    50985214    chr6:50980213-50990213  .
    chr6    50914860    50940037    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50940037    50940038    vp1 .   chr6    50914860    50914861    chr6:50909860-50919860  .

Full-length loci example
------------------------

### 1. Make the viewpoint\_format\_full.txt

The columns are:
viewpoint\_name start end color

    vp1 50939860  50940213  #FF0000
    vp2 50941269  50941535  #00FF04
    vp3 51171048  51171397  #FF00F4 
    vp4 51173671  51174019  #0C8900
    vp5 52009286  52009637  #1500FF
    vp6 52204449  52204744  #803582

### 2. Write the scripts that make the interact files

    #write_make_interact_full_files.sh
    cat viewpoint_format_full.txt | awk '{
            print "cat Face_"$1"_3C-seq_lifted_to_mm10.bed | awk \x27\{print \$1\"\\t\"\$2\"\\t"$3"\\t"$1"/3C-seq/Face\\t0\\t1\\tFacevBrain\\t"$4"\\t\"$1\"\\t"$2"\\t"$3"\\t"$1"\\t.\\t\"\$1\"\\t\"\$2\"\\t\"\$3\"\\t\"\$1\":\"\$2\"-\"\$3\"\\t.\"\}\x27 CONVFMT=\x27\%.0f\x27 > Face_3C\-seq_"$1"_mm10_full.tmp" \
            "\necho \x22track type=interact name=\\\"Face "$1" full loci\\\" description=\\\"Significant interactions in E11.5 Face (E11.5 Brain control)\\\" interactDirectional=true maxHeightPixels=200:100:50 visibility=full\x22 > Face_3C\-seq_"$1"_mm10_full.interact" \
            "\necho \"browser position chr6:49,720,000-52,880,000\x22 >> Face_3C\-seq_"$1"_mm10_full.interact" \
            "\ncat Face_3C-seq_"$1"_mm10_full.tmp | awk \x27\{if \(\$2<\$3\) print \$0\; else print \$1\"\\t\"\$3\"\\t\"\$2\"\\t\"\$4\"\\t\"\$5\"\\t\"\$6\"\\t\"\$7\"\\t\"\$8\"\\t\"\$9\"\\t\"\$10\"\\t\"\$11\"\\t\"\$12\"\\t\"\$13\"\\t\"\$14\"\\t\"\$15\"\\t\"\$16\"\\t\"\$17\"\\t\x22\$18\}\x27 >> Face_3C\-seq_"$1"_mm10_full.interact"}' > make_interact_full_files.sh

### 3. Run it

Sample output:

    track type=interact name="Face vp1 full loci" description="Significant interactions in E11.5 Face (E11.5 Brain control)" interactDirectional=true maxHeightPixels=200:100:50 visibility=full
    browser position chr6:49,720,000-52,880,000
    chr6    50940213    50950213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50939860    50940213    vp1 .   chr6    50950213    50960213    chr6:50950213-50960213  .
    chr6    50940213    50940213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50939860    50940213    vp1 .   chr6    50940213    50950213    chr6:50940213-50950213  .
    chr6    50940213    51160213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50939860    50940213    vp1 .   chr6    51160213    51170213    chr6:51160213-51170213  .
    chr6    50940213    50970213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50939860    50940213    vp1 .   chr6    50970213    50980213    chr6:50970213-50980213  .
    chr6    50919860    50940213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50939860    50940213    vp1 .   chr6    50919860    50929860    chr6:50919860-50929860  .
    chr6    50940213    50960213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50939860    50940213    vp1 .   chr6    50960213    50970213    chr6:50960213-50970213  .
    chr6    50940213    50980213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50939860    50940213    vp1 .   chr6    50980213    50990213    chr6:50980213-50990213  .
    chr6    50909860    50940213    vp1/3C-seq/Face 0   1   FacevBrain  #FF0000 chr6    50939860    50940213    vp1 .   chr6    50909860    50919860    chr6:50909860-50919860  .

Values (q-value, p-value, read number, etc.) can be included in the data bed files and substitued with a minor change to the code. Values do not influence the height of the arc or the thickness of the line. The convention seems to be that the arc height is related to the distance between the two anchors, to avoid having overlapping arcs if possible. Also, in the case of representing 4C data where Directional=true, dashed arcs lead to points downstream of viewpoint and solid arcs to those upstream.
