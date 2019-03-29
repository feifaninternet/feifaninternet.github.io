----------------
title: Git Notes
tags: git
categories: work notes
----------------
date: 2018-01-27 09:48:39

### Git operation

### Put your project into remote warehouse
1.you need a project code and a code repository(gitHub or coding)   
2.initialize local repository   

``` bash
$ git init
```

3.add the project code to working tree  
 
``` bash
$ git add .
```

4.commit your project code

``` bash
$ git commit -m "first commit notes"
```
   
5.synchronize the code to the remote repository

``` bash
$ git remote add origin your remote reposiroty address
```

6.create an upstream push to the master branch

``` bash
$ git push -u origin master
```
   
7.now you can create a new dev branch based on the master,and then create your branch based on the dev,keep the developments

### Change your git warehouse address
We use the simplest method.

   ```bash
    git remote set-url origin remote_git_address
   ```
  
### Submit code steps       
1.add your code to the working tree

``` bash
$ git add .
```

2.commit your code

``` bash
$ git commit -m "notes"
```

3.pushed to the remote branch

``` bash
$ git push origin yourBranch
```

4.switch to the dev branch 

``` bash
$ git checkout dev
```

5.synchronize remote dev branch code

``` bash
$ git pull
```

6.merge your branch code into dev

``` bash
$ git merge --no-ff -m "merge: notes"
```

7.pushed to the remote branch

``` bash
$ git push
```

8.switch to your branch 

``` bash
$ git checkout yourBranch
```

9.rebase the dev branch

``` bash
$ git rebase dev
```

10.pushed to the remote branch

``` bash
$ git push origin yourBranch
```

### Resolve git conflict

### Merge conflict   
1.find the conflict file   
2.find the text like this 
  
```
<<<<<<< HEAD  -- dev branch
 dev code
=======
your code
>>>>>>> 6853e5ff961e684d3a6c02d4d06183b5ff330dcc  -- your branch
```
   
3.remove the conflict flag, leaving only one branch of the code (dev code or your code)   
4.add changes to the working tree

 ``` bash
$ git add .
```
   
5.commit your code   

``` bash
$ git commit -m "notes"
```

6.push changes to dev branch   

``` bash
$ git push
```
   
### Rebase conflict   
1.find the conflict file   
2.find the text like this 

   ```
     <<<<<<< HEAD  -- dev branch
     dev code
     =======
     your code
     >>>>>>> 6853e5ff961e684d3a6c02d4d06183b5ff330dcc  -- your branch
   ```

3.remove the conflict flag, leaving only one branch of the code (dev code or your code)   
4.add changes to the working tree

   ``` bash
   $ git add -u
   ```

5.continue rebase   

   ``` bash
   $ git rebase --continue
   ```

6.until all conflicts resolved   
7.if you want to quit rebase   

   ``` bash
   $ git rebase --abort
   ```

*attention: after rebase,don`t need to commit and submit new changes,all git automatically*

### My IDEA .ignore directory

   ```
    target/
    !.mvn/wrapper/maven-wrapper.jar
    
    ### STS ###
    .apt_generated
    *.classpath
    .factorypath
    *.project
    *.settings/
    *.springBeans
    *.class
    
    ### IntelliJ IDEA ###
    *.jar
    # *.war
    *.ear
    *.log
    *.mvn
    
    *.idea
    *.iws
    *.iml
    *.ipr
    
    target
    src/test
    
    ### NetBeans ###
    nbproject/private/
    build/
    nbbuild/
    dist/
    nbdist/
    .nb-gradle/
   ```

need to remove cached first : 

   ```
    git rm -r --cached .
   ```
   
no userName no password every push :

   ```
    git config --global credential.helper store
   ```

  