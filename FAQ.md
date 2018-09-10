# FAQ for IntelligentOCR project. (Why the hell did you do what you did?)

## Extraction of pdf into memory
### Why calling command-line scripts instead of using wrappers?
This script has been thought inside a server with quad-core processor and 4GB of RAM
full of running processes. Then my focus has been to RAM saving. 
This is why I preferred to use the CL script so I could process a page at a 
once - and manage them as a generator of pillow images instead of a list.

### Do images need to be beautiful?
Yes, they do. In fact, every OCR program has this pre-processing part. 
However, using neural networks and deep learning to find tables led me to 
personally manage this part. For this reason I introduced a de-skewing of all pages 
before feeding the NN.

More steps are coming: the next one is de-noising of dirty backgrounds - maybe
introducing 
[these](https://www.kaggle.com/c/denoising-dirty-documents/kernels)
projects.

## Inference
### Is this a black box?
What the neural network returns is a list of boxes and scores. In my opinion, it is not wise to 
use them without understanding what the NN is _seeing_.

I observed some pattern inside the insure policies I had to analyzed:
* tables are always single-column;
* they mostly contain number and text;
* every page hardly contains more then 4 tables each.

Since I could not prepare a specific dataset for this kind of pages due to lack of time, 
the inference is not so precise:
* it mistakes big tables and considering it as a set of ones; 
* scores rarely goes over 80%;

For this reason I decided to interpret boxes:
1. First step is to **consider the first 10 boxes** (`MAX_NUM_BOXES`) with **score over 0.4** (`MIN_SCORE`): this let me 
consider very imprecise boxes, which have some _kind of idea_ of where the boxes are;
2. Then **merge all vertical overlapping boxes**: if a box is overlapping another one there is good probability
that both of them are looking at the same table. So merging them - considering the min `y_min` and the max `y_max` of
the two - let me increase accuracy;
3. Crop widely: since tables are in single-column with no inline text I could consider only 2 parameters out of 4
(`y_min` and `y_max` and not `x_min` and `x_max`). This let me reduce inference error.

Then **no**, it is not a black box at all.

I have made those tests with my other project
[TableTrainNet](https://github.com/mawanda-jun/TableTrainNet)
and applied the results to this one.

### Do images _still_ need to be beautiful?
Not so much indeed.
The pre-trained neural network has been trained over normal, 24-bit depth 3-channels images.
It is not simple for it to understand that tables are _objects_, since for it those are lines that have not so much
correlation with others.
For this reason it is useful to apply
[this](https://www.researchgate.net/publication/320243569_Table_Detection_Using_Deep_Learning)
transformation, that let my NN perform much better.