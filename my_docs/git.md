git 忽略已跟踪文件

There are often times when you want to modify a file but not commit the changes, for example changing the database configuration to run on your local machine.

Adding the file to .gitignore doesn’t work, because the file is already tracked. Luckily, Git will allow you to manually “ignore” changes to a file or directory:

```
git update-index --assume-unchanged <file>
```

And if you want to start tracking changes again, you can undo the previous command using:

```
git update-index --no-assume-unchanged <file>
```

