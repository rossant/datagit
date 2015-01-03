datagit
=======

Random notes about data science workflows with Python, pandas, git, and the Jupyter notebook

## Context

Imagine you're analyzing a big dataset, using Python, pandas, and the IPython/Jupyter notebook. You have a `data/` directory where you store the original, raw datasets. Since these are pretty messy and complicated, you first create a `cleaning.ipynb` notebook to clean the dataset, remove missing data, create new user-friendly fields, remove unnecessary labels, etc. You end up with a clean `DataFrame` 10 or 100 times smaller than the original dataset. Next you save this to some file, say `cleaned.csv`. The actual data science stuff can begin.

You create a new `analyze.ipynb` notebook where you start by opening `cleaned.csv` before analyzing it.

Of course, you're keeping track of your work with git by version-controlling your two `ipynb` notebooks. Of course, you *don't* version control the data files because no one does that. To test various ideas, you create various branches, regularly committing changes to your notebooks, you use expicit commit messages, and you automatically get a *graph* of your workflow (the graph of your commits and branches).

At some point, you realize that you need to change something in the cleaned dataset, because you can't possibly get it right the first time. And you need to have a cloose look at the dataset before knowing it's good anyway. Or, maybe your `cleaned.csv` only contained information about some part of the data, and now you want to analyze another part. So you go back to `cleaning.ipynb`, make some tweaks, and save the data again. Now you have two options.

1. Save the new cleaned dataset into `cleaned.csv`, erasing the old one.
2. Save the new cleaned dataset into `cleaned2.csv`, to keep track of the first one.

We immediately see the problems with both (1) and (2).

With (1), you loose the data associated with your temporary steps. After you've done this back-and-forth dance 17 times, if you need to go back at step 12 (which you can do with git), you've lost your cleaned dataset and you need to regenerate it again. Maybe this process takes 15 minutes because your dataset is huge.

With (2), you end up with an awfull mess in your `data` folder with dozens of files with esoteric names like `cleaned_17_old_new_takethisone_FINAAAL.csv`. This is because real-world workflows are rarely linear.

Can we do better?

## Thoughts about a possible solution



