---
layout: blog
title: "Automatically update README.md with Travis-CI"
date: "2018-07-28"
author: ["Siang Lim"]
---

While working on content for the [UBC OpenChemE Initiative](https://opencheme.github.io), I got tired of manually updating the README.md file with new notebooks and decided to look for a more elegant solution. 

# Rewriting README.md with new links
The goal is to update `README.md` automatically with links to new Jupyter notebooks whenever we make a commit on GitHub. The readme file feeds into Jekyll/GitHub Pages and generates the website that we see [here](https://opencheme.github.io/CHBE356/).

We'll do this with a bash script that collects the names of all .ipynb files in the Notebooks folder and dump the contents into README.md with the right markdown formatting and nbviewer links. We'll use Travis CI to run the script every time we make a commit on GitHub.

# Bash script
## Skip to [this section](#final-bash-script) if you don't want the details
We'll start by writing a bash script to grab all directories (`*`) in the Notebooks folder, one level up (`..`). Save this script as `deploy.sh` in a folder called `scripts` in your main project folder.


```bash
for d in ../Notebooks/* ; do
	echo "$d" 
	# Do something with files in this folder
done
```

If you run that, you'll see that `$d` prints out the entire path and not just the directory name. After some Googling, I found out how to split the base paths and the name using [parameter expansion](# https://stackoverflow.com/questions/3362920/get-just-the-filename-from-a-path-in-a-bash-script).

```bash
for d in ../Notebooks/* ; do
	xpath=${d%/*} 
	xbase=${d##*/}
	xfext=${xbase##*.}
	xpref=${xbase%.*}

	# Do something with files in this folder
done
```

Now, we want not only the directory, but also all notebook files in a particular directory, so we'll do something like this to loop through the files:

```bash
for d in ../Notebooks/* ; do
	xpath=${d%/*} 
	xbase=${d##*/}
	xfext=${xbase##*.}
	xpref=${xbase%.*}

	echo "$d"

    for f in "$d"/*.ipynb; do
		fpath=${f%/*} 
		fbase=${f##*/}
		ffext=${fbase##*.}
		fpref=${fbase%.*}

		echo "$f"
    done

done
```

If you run the script and look at the echo outputs, you'll see that we are getting close.

We want to output the directories as Markdown `h1` headings and the files as bullet points. We also want a space between each heading for readibility. Finally, we want to write all this to the README.md file in the root directory.

So here's what we'll do:

```bash

readme_path="../README.md"

for d in ../Notebooks/* ; do
	xpath=${d%/*} 
	xbase=${d##*/}
	xfext=${xbase##*.}
	xpref=${xbase%.*}

    echo "# $xbase" >> "$readme_path"
    
    for f in "$d"/*.ipynb; do
		fpath=${f%/*} 
		fbase=${f##*/}
		ffext=${fbase##*.}
		fpref=${fbase%.*}

    	echo "* $fpref" >> "$readme_path"
    done

    echo -e "\n" >> "$readme_path"

done

```

For the individual files, we want to link to nbviewer directly, and convert spaces to `%20` in the URL. Here's how to do it. The `//` in `${xbase// /%20}` means change all spaces ` ` to `%20`.


```bash

readme_path="../README.md"
nbviewer_path="http://nbviewer.jupyter.org/github/OpenChemE/CHBE356/blob/master/Notebooks" 

for d in ../Notebooks/* ; do
	xpath=${d%/*} 
	xbase=${d##*/}
	xfext=${xbase##*.}
	xpref=${xbase%.*}

    echo "# $xbase" >> "$readme_path"
    
    for f in "$d"/*.ipynb; do
		fpath=${f%/*} 
		fbase=${f##*/}
		ffext=${fbase##*.}
		fpref=${fbase%.*}

    	echo "* [$fpref]($nbviewer_path/${xbase// /%20}/${fbase// /%20})" >> "$readme_path"
    done

    echo -e "\n" >> "$readme_path"

done

```

# Final bash script
Almost there. We want to also create a separate 'header' file for our README.md and append the links below it. We will also add `shopt -s nullglob` in case we have [empty folders](https://unix.stackexchange.com/questions/239772/bash-iterate-file-list-except-when-empty
). Here's the final file:

```bash
#!/bin/bash

# a pattern that matches nothing "disappears", rather than treated as a literal string:
# https://unix.stackexchange.com/questions/239772/bash-iterate-file-list-except-when-empty
shopt -s nullglob

# Our paths for the readme file
header_path="../header.md"
readme_path="../README.md"
nbviewer_path="http://nbviewer.jupyter.org/github/OpenChemE/CHBE356/blob/master/Notebooks" 

# Copy the header over and add a blank line
cat "$header_path" > "$readme_path"
echo -e "\n" >> "$readme_path"

# https://stackoverflow.com/questions/3362920/get-just-the-filename-from-a-path-in-a-bash-script
for d in ../Notebooks/* ; do
	xpath=${d%/*} 
	xbase=${d##*/}
	xfext=${xbase##*.}
	xpref=${xbase%.*}

    echo "# $xbase" >> "$readme_path"
    
    for f in "$d"/*.ipynb; do
		fpath=${f%/*} 
		fbase=${f##*/}
		ffext=${fbase##*.}
		fpref=${fbase%.*}

    	echo "* [$fpref]($nbviewer_path/${xbase// /%20}/${fbase// /%20})" >> "$readme_path"
    done

    echo -e "\n" >> "$readme_path"

done
```

# Travis CI
We're all done with the bash script at this point. Now for the Travis CI part. Travis CI is integrated with GitHub. The service allows us to run scripts every time we make a commit on GitHub. These scripts could be unit tests or any bash script. 

I didn't really understand the value of CI services like Travis CI or Circle CI until I had to implement unit tests for the [UBC Envision](https://www.ubcenvision.com) site to prevent our team members from unintentionally(?) breaking it with bad commits *(to be covered in a separate post)*.

## Set up tokens
We'll need an authentication token from GitHub to allow Travis to make changes to the repository. Run this bash command and save the value of the `token` key on your screen:

```bash 
$ curl -u csianglim -d '{"scopes":["public_repo"],"note":"CI"}' https://api.github.com/authorizations
```

Navigate to your project folder and install the `travis` gem

```
$ gem install travis
```

Make sure you're in your project folder, then run this command to encrypt the token. Travis will automatically add it to your .travis.yml file:

```
$ travis encrypt GIT_NAME="Travis CI" --add
$ travis encrypt GIT_EMAIL="travis@travis-ci.org" --add
$ travis encrypt GH_TOKEN=<token> --add
```

Here's what my .travis.yml file looks like:

```
language: ruby
script:
- bash ./scripts/deploy.sh
branches:
  only:
  - master
env:
  global:
  - secure: <encrypted_token>  
  - secure: <encrypted_token>  
  - secure: <encrypted_token>
```

Now we just need add a few more lines to our `deploy.sh` script and tell Travis to set up git and push to the repo:

```
# Setup and push to master
git config --global user.email ${GIT_EMAIL}
git config --global user.name ${GIT_NAME}
git checkout master
git add .
git commit --message "Travis $TRAVIS_BUILD_NUMBER: $TRAVIS_COMMIT_MESSAGE"
git remote set-url origin https://${GH_TOKEN}@github.com/OpenChemE/CHBE356.git
git push origin master
```

# Conclusion
I am a big advocate of automation to avoid doing any repetitive and tedious work. Reducing unnecessary human input will allow us to spend time on more important and challenging problems, create consistent workflows for all team members and reduce human errors in the final results.