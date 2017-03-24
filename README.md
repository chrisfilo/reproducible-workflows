
Walkthrough for BBSRC Meeting presentation on reproducible workflows

### make a new directory and cd into it
```
mkdir BBSRC-git-demo
cd BBSRC-git-demo
```

### create a new git repository
```
git init
git status
```

### Start up RStudio, and create a new R script called somecode.R containing the following lines (which  will load the data from the [Lewandowsky et al. study](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0075637):
```
# R code
df=read.table('http://data.bris.ac.uk/datasets/swyt56qr4vaj17op9cw3sag7d/LskyetalPLOSONE.csv',
              header=TRUE,sep=',')
head(df)
```
Note: code to be added to the R script is marked with "# R code".  Any cells not marked this way are meant to be typed into the terminal window.

### After you save the file, run the R script using the "source" button in Rstudio

### (Hopefully!) this worked, so let's check our file into the repo
```git status
git add somecode.R
git status
git commit -m"initial add"
git status
```

### let's run a linear regression model to see if conspiracist thinking is related to age

### add the following code to somecode.R and source the file:
```
# R code
lm.result=lm(conspiracist_avg~age,data=df)
summary(lm.result)
```

### This should also complete successfully.  Let's go ahead and check in again
```
git add somecode.R
git commit -m"adding lm"
```

### Now let's put this into a repository on github so that we can share it with others and have a persistent backup.

1. log into [github](github.com)
2. create a new repository (+ sign at top right)
   * give it the same name as your directory (BBSRC-git-demo)
   * just use the defaults (it should be public) and click "create repository"
3. There will be a set of commands in the section titled "…or push an existing repository from the command line" - copy those and paste them into the terminal inside the directory with your git repository. You may need to enter your github username and password.

The commands willl look somethign like:
```
git remote add origin git@github.com:<your username>/BBSRC-git-demo.git
git push -u origin master
```

Click on the repository link at the top of the page to go to the main github repo page. you should see "somecode.R" in the list.


### Now let's set up CircleCI to automatically run a smoke test for us every time we check in a new revision of our code to github.

### create a file in your github repository called circle.yml and add the following lines.  An easy way to do this is to use the "Create new file" button on the github page, which will take you to an editor where you can create the file and then save and commit it with one click.

```
dependencies:
  pre:
    - sudo apt-get update && sudo apt-get -y install r-base
test:
  override:
    - Rscript somecode.R
```

### if you instead created the circle.yml on your own computer using a text editor, then add to repo and commit and push to github

```
git add circle.yml
git commit -m"initial add"
git push origin master
```

### if you created it using the editor on github, then you should pull that change so that your local repository is in sync with github.
```
git pull origin master
```

### next we have to hook this up to the CircleCI continuous integration system

1. go to [CircleCI](http:circleci.com) and log in using your github account
2. click on the "Projects" button (with the + sign)
3. Choose your github account, and then click on the "build project" button for your repo
4. It will then take you to a page showing the status of the build.  for an overview, click on the "builds" button which will take you to a list of builds.

After a couple of minutes it should show that the build succeeded

### look at the log to see what we've done so far
```git log
```

### we were a bit surprised that there is no relation between age and conspiracist thinking, so let's have a closer look at the data

### add the following code and source the file

```
# R code
plot(df$age,df$conspiracist_avg)
```

### Then commit the changes to the git repo.

```
git add somecode.R
git commit -m"adding plot"
```

### let's say that we decided that we don't want the plot in the file. We can go back to the previous commit:

First, use ```git log``` to show the log of previous commits, which will give you the commit ID (a long alphanumeric hash).

Then, revert that partiuclar commit:

```
git revert <commit ID>
```

The change in the file should show up immediately in the RStudio editor window

### It was clear from the plot that something is wrong: there is an outlier in the age distribution
### let's first add a test to check for age outliers
### in this study, subjects were supposed to be adults - let's say the reasonable range of adult ages is 18 to 100

### add the following code above the lm command:

```
# R code
max_age=120
min_age=18
stopifnot(max(df$age)<max_age)
stopifnot(min(df$age)>min_age)
```

### when you source the file, you should see an error.
### let's see what happens when we push this to github

```
git add somecode.R
git commit -m"adding assertion test"
git push origin master
```

### a few seconds later, you will see that the automated test starts on circleci
### you will see that the test fails due to the error

### let's add some code to clean up the outliers
### above the assertion tests, add:

```
# R code
df=subset(df,age>min_age&age<max_age)
```

### run the code again - this time it succeeds and we now see a strong effect of age (as noted in [the correction to the original paper](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0134773)): 

### push it back to github and check circleci again
```
git add somecode.R
git commit -m"adding outlier removal"
git push origin master
```

### add a fancy badge to your github page to show off
1. go to the builds page and click the gear next to your repo
2. click on "status badges" and copy the text under "embed code"
  * it will look something like ```[![CircleCI](https://circleci.com/gh/poldrack/BBSRC-git-demo.svg?style=svg)](https://circleci.com/gh/poldrack/BBSRC-git-demo)```
3. add this into a file called README.md in your github repository


