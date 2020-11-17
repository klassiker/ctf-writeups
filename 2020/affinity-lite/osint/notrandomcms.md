# NotRandomCMS

## Task


A friend started to develop a CMS for his company using a PHP framework, can you check if there is any security issue? I removed the application secrets to be safe to share it.

File: CMS.7z

## Solution

Unzip, open all files. Look through all of them. You find:

`views/layouts/main.php`

Inside there is a link to the source repository:

https://github.com/notrandomcms/notrandomcmsv1

Look through commits: `Removing secret files` /commit/6cdec47e7b78394095de5c8856fd67e2a9b6410c

Last diff:

`AFFCTF{thisShouldBeASecret!}`
