# Repository for [differentialprivacy.org](https://differentialprivacy.org/)
We're currently working to add some content.


## Contribute to [differentialprivacy.org](https://differentialprivacy.org/)
### The easy way: email the editors!
- Write up your content as a markdown file. Check out the [Welcome
Post](https://github.com/differentialprivacy/differentialprivacy/blob/master/_posts/2020-7-11-Welcome.md)
as an example.
- Email your markdown file to
  [admin@differentialprivacy.org](mailto:admin@differentialprivacy.org).


### The GitHub way: create a pull request
- First *Fork* our repo by clicking on the Fork button on the
  top-right corner. This creates a new copy of the repo under your
  GitHub user account under the url
  ```
  https://github.com/<YourUserName>/differentialprivacy
  ```
  You can skip this step if you have already done so previously.

- Next, clone the repo by running the command:
  ```
  git clone https://github.com/<YourUserName>/differentialprivacy
  cd differentialprivacy/
  # Add the remote (original repo that you forked) and call it “upstream”
  git remote add upstream https://github.com/differentialprivacy/differentialprivacy.git 
  ```

  If you have already forked the repo before, you should skip
  this. But you may want to update the forked repo with `git rebase`:
  ```
  # Fetch all branches of remote upstream
  git fetch upstream
  # Rewrite your master with upstream’s master using git rebase.
  git rebase upstream/master
  # Push your updates to your own master. You may need to force the push with “--force”.
  git push origin master --force
  ```

- Create a new branch by 
  ```
  git checkout -b gauss_mech
  ```
  where `gauss_mech` is the name of your new branch.

- Now that you are inside this new branch, you can add or change
  contents. For example, if you want to create a new post, the following code 
  will create a pull request under your branch
  ```
  cd _post/ # go to the post directory
  echo “some test file” > test.md # toy example of a new post
  git add test.md 
  git commit -m "Adding a test file to this branch"
  git push -u origin gauss_mech
  ```
  
- The editors will review your edits and merge the edits if they are
  good!
  

## Installation
- To run the website locally, you would need to install
  [Jekyll](https://jekyllrb.com/).

## Preview your changes
- To preview the website locally, run command line:
```
# go to the repo directory
cd differentialprivacy/
jekyll serve
# then browse to http://localhost:4000
```
