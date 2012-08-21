# wordpress-git-svn

Painlessly manage your Wordpress plugins with git, and with one click, deploy them to the wordpress.org plugin repository.

## Requirements

wordpress-git-svn assumes you have the following installed:

* git
* svn
* ruby 1.9 (a .rvmrc is included for rvm users)

## Getting Started

Clone the repository:

    git clone git://github.com/freerobby/wordpress-git-svn.git

Install ruby dependencies:

    cd wordpress-git-svn/
    gem install bundler
    bundle install

### Create your first workspace

If your plugin were called "artpal", here's how you'd set up its workspace.

    rake plugins:convert_to_git:prepare[artpal]
    
This will query the subversion server and create a ./workspaces/artpal/authors.txt file containing a list of svn authors in the format `svn_user = name <email>`. Update these email addresses to reflect the github addresses of anybody you know in the file. This will map the git/svn author histories. Once complete, run:

    rake plugins:convert_to_git:run[artpal]

This will take a while -- at least a few minutes, possibly a few hours. The reason for this is that the wordpress.org plugin repo is one gigantic SVN repository, and wordpress-git-svn has to cull through every single commit between when your repo was created and its last commit.

Upon completion of this command, the git repository will be created at `./workspaces/artpal/artpal-git`. This is your git working directory. Use it as you would any other git repo, commit changes, push to github, etc.

