datagit
=======

Random notes about data science workflows with Python, pandas, git, and the Jupyter notebook

## Context

Imagine you're analyzing a big dataset, using Python, pandas, and the IPython/Jupyter notebook. You have a `data/` directory where you store the original, raw datasets. Since these are pretty messy and complicated, you first create a `cleaning.ipynb` notebook to clean the dataset, remove missing data, create new user-friendly fields, remove unnecessary labels, etc. You end up with a clean `DataFrame` 10 or 100 times smaller than the original dataset. Next you save this to some file, say `cleaned.csv`. The actual data science stuff can begin.

You create a new `analyze.ipynb` notebook where you start by opening `cleaned.csv` before analyzing it.

Of course, you're keeping track of your work with git by version-controlling your two `ipynb` notebooks. Of course, you *don't* version control the data files because no one does that. To test various ideas, you create various branches, regularly committing changes to your notebooks, you use explicit commit messages, and you automatically get a *graph* of your workflow (the graph of your commits and branches).

At some point, you realize that you need to change something in the cleaned dataset, because you can't possibly get it right the first time. And you need to have a cloose look at the dataset before knowing it's good anyway. Or, maybe your `cleaned.csv` only contained information about some part of the data, and now you want to analyze another part. So you go back to `cleaning.ipynb`, make some tweaks, and save the data again. Now you have two options.

1. Save the new cleaned dataset into `cleaned.csv`, erasing the old one.
2. Save the new cleaned dataset into `cleaned2.csv`, to keep track of the first one.

We immediately see the problems with both (1) and (2).

With (1), you loose the data associated with your temporary steps. After you've done this back-and-forth dance 17 times, if you need to go back at step 12 (which you can do with git), you've lost your cleaned dataset and you need to regenerate it again. Maybe this process takes 15 minutes because your dataset is huge. And you lose the benefits of git if you're unable to tell which exact notebook you used to generate a given version of your dataset.

With (2), you end up with an awfull mess in your `data` folder with dozens of files with esoteric names like `cleaned_17_old_new_takethisone_FINAAAL.csv`. This is because real-world workflows are rarely linear.

Can we do better?

## Thoughts about possible solutions

I think the best solution is one that uses existing widespread tools as much as possible instead of creating something new, and that doesn't get in the way of the user. The system should adapt to the user rather than the other way around. It should provide clean abstractions and implementations that can be reused by advanced users.

### Track files with a VCS

You *could* use git to track your files along with your notebooks, but you'll end up with huge repos and this probably won't scale to big datasets and complex workflows. That's not the right tool for the job.

Maybe there exists a VCS for binary files, but how to integrate it with git?

### Using git annex?

[?](https://git-annex.branchable.com/)

### Bypassing git

What we want is to save all intermediate datasets along with our git commits. It's essentially a matter of **caching** the output of the cleaning notebooks. Here is a possible solution:

* To generate a new intermediate dataset, instead of doing it manually with e.g. `pd.to_csv(filename)`, you call a special function `datagit.to_csv(data, basename, message)` that does several things:
  * Save the notebook.
  * Commit with the given message.
  * Add a [git note](http://git-scm.com/docs/git-notes) (see also [this blog post](http://git-scm.com/blog/2010/08/25/notes.html)) to the commit with `datagit <basename>`.
  * Save the data in a `.datagit` subdirectory with the filename `<basename>_<commithash>.csv`.

* Now, to load an intermediate dataset, in the current notebook or in anywhere else, you call `data = datagit.from_csv(basename)`. This will load a cached version of the dataset. Specifically:
  * If `.datagit/<basename>_<commithash>.csv` exists, it loads it.
  * If not, it will move back to the previous commit that has a `datagit something` git note, and try again, until it finds a cached file. If nothing is found, an error is raised.

Why wouldn't `.datagit/<basename>_<commithash>.csv` work in the first place? Because you might use the same git repo for your cleaning and analysis notebooks. So there could be many commits without any new data. What you want is the last generated dataset, which should be found when you go back in the git history. This should work fine even when using branches.

Synchronizing git notes doesn't appear to be easy, but it's possible (although limited). Anyway this solution is essentially useful for local work.
