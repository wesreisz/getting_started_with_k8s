** Hugo Basics**
* To run hugo in development mode, type `hugo server -D`. -D makes all drafts published.
* to build hugo for deployment (the files will target docs), run `hugo --buildDrafts` and then commit the generated files to git and push them to master. The docs folder is what github is configured to deploy.