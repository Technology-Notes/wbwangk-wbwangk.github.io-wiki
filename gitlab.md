### 下面是新建项目后的提示
The repository for this project is empty 

If you already have files you can push them using command line instructions below. 

Otherwise you can start with adding a README, a LICENSE, or a .gitignore to this project. 

You will need to be owner or have the master permission level for the initial push, as the master branch is automatically protected. 




Command line instructions 


```
Git global setup
git config --global user.name "王伟兵"
git config --global user.email "wbwang@inspur.com"
```


Create a new repository
```
git clone --depth 1 http://gitserver/imaidata/docs.git
cd docs
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```


Existing folder
```
cd existing_folder
git init
git remote add origin http://gitserver/imaidata/docs.git
git add .
git commit -m "Initial commit"
git push -u origin master
```


Existing Git repository
```
cd existing_repo
git remote add origin http://gitserver/imaidata/docs.git
git push -u origin --all
git push -u origin --tags
```
