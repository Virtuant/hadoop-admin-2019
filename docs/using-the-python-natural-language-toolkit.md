## Using the Python Natural Language Toolkit

### About This Lab

**Objective:** To become familiar with how the Python Natural Language Toolkit (NLTK) is used to process a body of text

**Successful outcome:** You will calculate many different results from various NLTK functions analyzing 60 years of State of the Union addresses

**File location:** `~/labs`

---
### Steps



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>1. Create a New Notebook</h2>

1\.  From your web browser, go to the IPython Notebook page

2\.  Click on the `labs` folder of IPython.

3\.  Click the `New Notebook` button to create a new Notebook.

4\.  Change the name of the Notebook to `NLTK_Lab`:

![using-the-python-natural-language-toolkit-1](https://user-images.githubusercontent.com/21102559/40943134-1461897c-681e-11e8-9b08-dd8493ac42b9.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>2. Import the NLTK Library</h2>

1\.  In the first cell import the `NLTK` library:

```
import nltk
```

2\.  Run the cell.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>3. View the Data</h2>

1\.  Import the `state_union` data and display the words in it:

![using-the-python-natural-language-toolkit-2](https://user-images.githubusercontent.com/21102559/40943135-1472c750-681e-11e8-84a9-c4917c0509a3.png)

> Note  Only the first few words are shown. This document is actually quite large.

h.  Print out the number of words in the corpus. You should see 399,822 as the result:

![using-the-python-natural-language-toolkit-3](https://user-images.githubusercontent.com/21102559/40943136-1482de10-681e-11e8-9bd8-4d5df1a6bbc5.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>4. Segment the Sentences</h2>

1\.  In the next cell add the following code, which attempts to display the sentences of the corpus:

```
sentences = state_union.sents() 
print sentences
```

2\.  Run the cell.

3\.  Open a Terminal window in your classroom VM:

![using-the-python-natural-language-toolkit-4](https://user-images.githubusercontent.com/21102559/40943137-149188c0-681e-11e8-928c-beeb894118d7.png)

4\.  You should see the `.zip` file and the unpacked `punkt` folder in your `/root/nltk_data/tokenizers/` folder:

```
ls /root/nltk_data/tokenizers/ 
punkt punkt.zip 
```

5\.  View the contents of the `punkt` folder. Notice it contains a collection of `.pickle` files for various languages:

```
ls /root/nltk_data/tokenizers/punkt
```

Output:
```
czech.pickle 	english.pickle 	french.pickle 	italian.pickle 	portuguese.pickle
slovene.pickle 	turkish.pickle 	danish.pickle 	estonian.pickle german.pickle 
norwegian.pickle PY3 		spanish.pickle 	dutch.pickle 	finnish.pickle 
greek.pickle 	polish.pickle 	README 		swedish.pickle
```

6\.  Print the number of sentences:

![using-the-python-natural-language-toolkit-5](https://user-images.githubusercontent.com/21102559/40943138-149e9d8a-681e-11e8-9340-2d6072b6f497.png)

> Notice the corpus has 17,930 sentences (as determined by the `Punkt` sentence segmenter).



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>5. Use the Text Class</h2>

1\.  In this step, you will create a `nltk.text.Text` object that acts as a wrapper around a body of text. The `Text` class contains several useful functions. Start by instantiating a new `Text` object from the words in `state_union`:

![using-the-python-natural-language-toolkit-6](https://user-images.githubusercontent.com/21102559/40943139-14ab19b6-681e-11e8-9910-79b22204d829.png)

2\.  The **count** function shows the number of occurrences of a specific word. Run the following in a new cell:

![using-the-python-natural-language-toolkit-7](https://user-images.githubusercontent.com/21102559/40943141-14c0bf00-681e-11e8-9bf7-49b86af5d2f7.png)


3\.  Text has a `concordance` function that is executed on a specific word. The result is every occurrence of the specific word along with words around it (to provide a context). For example, print the concordance of the word "economy":

![using-the-python-natural-language-toolkit-8](https://user-images.githubusercontent.com/21102559/40943142-14d32398-681e-11e8-8f2f-b0ef2b031be8.png)

> Note  the word appears 437 times in the corpus, and IPython is displaying 25 of them.

4\.  Run the `simila`r function on "economy", which displays words which appear in the same context as "economy" with the most similar words first:

![using-the-python-natural-language-toolkit-9](https://user-images.githubusercontent.com/21102559/40943143-14e09d52-681e-11e8-92ee-768f21829305.png)

5\.  The `common_contexts` function finds the contexts where the specified words appear, with the most frequent common contexts first. Call this function on the "economy" and "jobs" words:

![using-the-python-natural-language-toolkit-10](https://user-images.githubusercontent.com/21102559/40943144-14f2d526-681e-11e8-8be3-087211501f1f.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>6. Calculate a Frequency Distribution</h2>

1\.  NLTK contains a `nltk.probability.FreqDist` class for computing frequency distributions for a corpus of words. Let's compute a frequency distribution for `state_union`. Start by importing the `FreqDist` class:

![using-the-python-natural-language-toolkit-11](https://user-images.githubusercontent.com/21102559/40943145-15029a2e-681e-11e8-8bcd-00cf59d10a79.png)

2\.  In a new cell, create a new `FreqDist` object from the `Text` object of `state_union`:

![using-the-python-natural-language-toolkit-12](https://user-images.githubusercontent.com/21102559/40943146-151034c2-681e-11e8-8fd5-a94c6b76f949.png)

![using-the-python-natural-language-toolkit-13](https://user-images.githubusercontent.com/21102559/40943147-152109e6-681e-11e8-9e24-3267146be180.png)

Not surprisingly, the most common words are "the", "of", "to" and so on.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>7. Remove Stop Words</h2>

1\.  Let's take a look at the stop words in the NLTL package:

![using-the-python-natural-language-toolkit-14](https://user-images.githubusercontent.com/21102559/40943148-152e7de2-681e-11e8-8e3d-d03b645193c2.png)

2\.  Remove the stop words from `state_union` and save the result in a new series named `filtered`. This command may take a minute to run:

```
filtered = [w for w in state_union.words() if not w in stopwords.words("english")]
	
len(filtered)
```

> Notice the number of words is down to 246,278.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>8. Calculate the Frequency Distribution</h2>

1\.  Define a new variable named `fdist_filtered` and assign it to a new `FreqDist` object for the `filtered` series.

2\.  Print the 20 most common words in `fdist_filtered`. The output should look like:

![using-the-python-natural-language-toolkit-15](https://user-images.githubusercontent.com/21102559/40943149-153bd032-681e-11e8-8b1c-e2a97480d0ed.png)

3\.  **FreqDist** has a **freq** function for obtaining the frequency of a specific word:

![using-the-python-natural-language-toolkit-16](https://user-images.githubusercontent.com/21102559/40943150-154a4ba8-681e-11e8-9a8f-55accfcb38dc.png)

> Notice the word "**good**" is used 12 times more frequently than the word "**bad**".

4\.  `FreqDist` has a `plot` function that plots the result. Run the following command in a new cell:

![using-the-python-natural-language-toolkit-17](https://user-images.githubusercontent.com/21102559/40943151-155aae94-681e-11e8-881a-36ffd3dcbc95.png)



### Result

After using the Natural Language Toolkit library, you should now have a better understanding of Natural Language processing and the work involved in dealing with large amount of text.

You are finished!
