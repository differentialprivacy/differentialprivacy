# Repository for [differentialprivacy.org](https://differentialprivacy.org/)
We're currently working to add some content.

## Installation
- To run the website locally, you would need to install
  [Jekyll](https://jekyllrb.com/).

## Preview
- To preview the website locally, run command line:
```
# go to the repo directory
cd differentialprivacy/
jekyll serve
# then browse to http://localhost:4000
```

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

- Next, clone the repo by running the command:
  ```
  git clone https://github.com/<YourUserName>/differentialprivacy
  ```

- Create a new branch by 
  ```
  git checkout -b gauss_mech
  ```
  where `gauss_mech` is the name of your new branch.

- Create a new remote for the upstream repo with the command:
  ```
  git remote add upstream https://github.com/differentialprivacy/differentialprivacy
  ```

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
  
- The editors will review your edits merge the edits if they are good!

