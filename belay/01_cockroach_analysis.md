# Cockroach Analysis
### A Statistical Analysis of the Flash and Java Files that Infest the Internet

## Abstract

Java and Flash are and will continue to be popular attack vectors. To combat this, we’ll put these two file formats under the microscope and throw some data science at them. For each file format, we will take a quick look at its layout and then explore some of the file features. Then using a malicious and clean file set, we will walk through the process we took to identify important features and show the results of from several different machine learning algorithms when built from these feature sets. We’ll use several open source tools and libraries to perform the data exploration and analysis, including pandas, scikitlearn as well as the data hacking library we’ve already released. IPython notebooks containing the analysis will be released at the
start of the talk.

## Introduction

Data Science is about asking your data questions and getting actionable information back from it.  It encompasses a broad range of fields from statistics and machine learning to modeling and visualization.  Our presentation applies these fields to detecting malicious java class files and flash files.  It includes a step by step example of the iterative process of exploring your data and extracting new features from it in a pair of IPython notebooks.  These techniques will allow us to detect malicious files without the use of signatures and we will evaluate the classifiers on a large corpus of files collected off the Internet.

For the analysis, we use several existing tools.

   - [IPython notebook](http://ipython.org/notebook.html) for data presentation
   - [pandas Python Data Analysis Library](http://pandas.pydata.org/)
   - [matplotlib](http://matplotlib.org/) for plotting
   - [scikit-learn](http://scikit-learn.org/stable) for machine learning


For both file types, we cover the highlights of the analysis.  More information can be found in the IPython notebooks at the Click Security data hacking [repository](http://clicksecurity.github.io/data_hacking).

## Java Analysis
We collected and labeled about 2000 known malicious and 500 known benign Java Class files.  We also had a large unlabeled corpus of 370,000 files.  While we did not know if these files were malicious or benign, we suspected that the vast majority, if not all, were benign.  To start the analysis, we used only 500 malicious and 500 benign files.

The Java Class file format contains several sections that contain interesting information.  The constant pool contain string information and the method section contains all the information about each method in the file.  We started by extracting basic information like the size of the file and the entropy of the entire file and the numeric values contained in the file.  Next, we examined the method names.  We noticed that the in many files from the malicious set, the method names did not follow normal English character usage.  To turn this information into features our classifier could use, we extracted the longest and average length of consecutive lowercase, uppercase, and number sequences.  Finally, we examined the class name of the file.  Again, we noticed the malicious set did not following normal English character usage.  In addition, the class names were much shorter and did not contain as many slashes as the benign set.  Taking all this data, we train and test a Random Forest classifier.  Our classifier trains on 80% of the data and tests on the remaining 20%.  We got the following results.

	benign/benign: 100.00% (92/92)
	benign/malicious: 0.00% (0/92)
	malicious/benign: 0.0% (0/112)
	malicious/malicious: 100.00% (0/112)

As you can see, we got excellent results.  Next, we had the classifier tell us the most important features.

Feature Name              |  Feature Importance
:------------------------ |:--------------------:
Class Name Slash Count    | 0.30258
Class Name Length         | 0.27841
Class Name Lower Case Max | 0.07751
Entropy                   | 0.06485
Class Name Lower Case Avg | 0.05893

Features extracted from the class name dominate the classifier.  This is both good and bad.  It's good because we got good results.  However, the attacker only has to modify how the class name is generated to defeat this classifier.  It's important to understand the limitations of what we created.

Next we created a new classifier using all of the malicious files and 2000 files from the unknown corpus as the benign set.  When we ran this classifier on the remaining corpus, we got the following results.

	359K files marked as benign
	3.7K files marked as suspicious
	1.3K files marked as malicious  

While not as good as our original testing would indicate, these files results show promise and with additional features, the classifier should be able to be improved.

## SWF Analysis

We collected and labeled only 628 malicious files and 500 benign files.  We also had a large unlabeled corpus of 288,000 files. Again, while we were not sure of the label for these, we suspected that the vast majority, if not all, were benign. To start the analysis, we used all the malicious files and 500 benign files.

The SWF file format contains a small header followed by any number of tags that contain various information about the file.  We started by extracting the data from the header and the number of tags that the file contained.  We then take into consideration the type of tags that are in the file.  A tag can exist multiple times, however, we only take into consideration the existence of a tag in the file, not the number of times it exists.  We then focus on extracting more information from tags that contain ActionScript.  We extract several features, include the number of strings and the ratio of the mean length of the strings to the median length of the strings.  Taking all this data, we train and test a Random Forest classifier. Our classifier trains on 80% of the data and tests on the remaining 20%. We got the following results.

	benign/benign: 98.10% (103/105)
	benign/malicious: 1.90% (2/105)
	malicious/benign: 0.84% (1/119)
	malicious/malicious: 99.16% (118/119)

While the results from the test are not as good as we saw in the Java Class section, we still got promising results.  Again, we want to see what features the classifier determined were the most important.

Feature Name              |  Feature Importance
:------------------------ |:--------------------:
Entropy  | 0.08949          
PlaceObject2 | 0.07960
ActionScript String Count | 0.06956
Size | 0.05679
Number Of Tags | 0.05196
Frame Rate | 0.05035
Frame Perimeter | 0.04962
Version | 0.04840
Frame Area | 0.04566

This classifier is not dominated by any one feature.  This means that to defeat this classifier, the attack will have to modify several features rather than focusing on only a few features.

Next we created a new classifier using all of the malicious files and 5000 files from the unknown corpus as the benign set.  When we ran this classifier on the remaining corpus, we got the following results.

	283K files marked as benign
	679 files marked as suspicious
	407 files marked as malicious  

These results are very promising.  In fact, these results may even be manageable.  It's possible with additional features the classifier could be improved even more.

## Conclusions

Using data science to detect new malicious files is possible.  This allows us to not rely on easily avoidable signatures, but to rely on nonlinear decisions boundary in a high dimensional space.  While the Java classifier is heavily dependent on class name, the SWF classifier is not highly dependent on any one field.  We also showed that training with more data could make the classifier better.

#### Metadata

Tags: Data Science, Machine Learning, malware, swf, java

**Primary Author Name**: David Dorsey  
**Primary Author Affiliation**: Click Security  
**Primary Author Email**: trogdorsey@gmail.com
