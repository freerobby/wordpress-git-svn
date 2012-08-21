require "rake"

require "fileutils"

namespace :plugins do
  WORKFLOWS_DIR = File.join(File.expand_path(File.dirname(__FILE__)), "workflows")
  
  desc "Generate the workspace and svn authors file for a wordpress repo"
  task :prepare_import, :plugin_name do |t, args|
    raise "Must specify name of plugin" if args[:plugin_name].nil?
    
    # Create workflow directory
    workflow_dir = File.join(WORKFLOWS_DIR, args[:plugin_name])
    FileUtils.mkdir_p workflow_dir, :mode => 0775
    
    # Look at log for first revision
    puts "Locating first revision for plugin: #{args[:plugin_name]}. This may take a minute."
    revision = `svn log -r 1:HEAD --limit 1 http://plugins.svn.wordpress.org/#{args[:plugin_name]} | grep -E "r[0-9]+" | awk '{print $1}'`.strip
    raise "No revision found for plugin: #{args[:plugin_name]}" if revision == ""
    puts "Found revision: #{revision}"
    puts ""
    
    # Calculate authors list
    puts "Looking up all revision authors. This may take a minute."
    authors = `svn log http://plugins.svn.wordpress.org/#{args[:plugin_name]} | grep -E "r[0-9]+ \\| .+ \\|" | awk '{print $3}' | sort | uniq`.split("\n")
    puts "Found authors: #{authors.join(", ")}"
    puts ""
    
    # Write authors list to file
    authors_path = File.join(workflow_dir, "authors.txt")
    File.open(authors_path, "w") do |f|
      authors.each {|a| f.puts "#{a} = #{a} <#{a}@svn>"}
    end
    
    puts "Authors file written to: #{authors_path}"
    puts "Please update authors with their github email addresses, then run 'rake plugins:run_import[#{args[:plugin_name]}]'"
    puts ""
  end
  
  desc "Convert a wordpress-hosted SVN repository to a local git repo"
  task :run_import, :plugin_name do |t, args|
    raise "Must specify name of plugin" if args[:plugin_name].nil?
    authors_path = File.join(WORKFLOWS_DIR, args[:plugin_name], "authors.txt")
    raise "No authors file. Please run 'rake plugins:prepare_import[#{args[:plugin_name]}]'" unless File.exists?(authors_path)
    
    # Look at log for first revision
    puts "Locating first revision for plugin: #{args[:plugin_name]}. This may take a minute."
    revision = `svn log -r 1:HEAD --limit 1 http://plugins.svn.wordpress.org/#{args[:plugin_name]} | grep -E "r[0-9]+" | awk '{print $1}'`.strip
    raise "No revision found for plugin: #{args[:plugin_name]}" if revision == ""
    puts "Found revision: #{revision}"
    puts ""
    
    puts "Performing svn import. This is a one-time operation. It can take over an hour for plugins with long histories."
    workflow_dir = File.join(WORKFLOWS_DIR, args[:plugin_name])
    git_dir = File.join(WORKFLOWS_DIR, args[:plugin_name], "#{args[:plugin_name]}-git")
    `git svn clone -s -#{revision} --no-minimize-url http://plugins.svn.wordpress.org/#{args[:plugin_name]} -A #{authors_path} #{git_dir}`
    `git --git-dir=#{git_dir}/.git svn fetch`
    puts ""
    
    puts "Merging history"
    `git --git-dir=#{git_dir}/.git checkout master`
    `git --git-dir=#{git_dir}/.git merge remotes/trunk`
    `git svn rebase`
    puts ""
        
    puts "Git repository generated. Enjoy using your git workflow of choice!"
    puts ""
  end
  
  desc "Deploy your code to Wordpress!"
  task :deploy, :plugin_name, :new_version do |t, args|
    raise "Must specify name of plugin" if args[:plugin_name].nil?
    raise "Must specify verion to deploy" if args[:new_version].nil?
    
    git_dir = File.join(WORKFLOWS_DIR, args[:plugin_name], "#{args[:plugin_name]}-git")
    
    current_branch=`git --git-dir=#{git_dir}/.git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \\(.*\\)/\\1/'`.strip
    raise "You must be on branch master to deploy. You are on #{current_branch}" unless current_branch == "master"
    
    readme_path = File.join(WORKFLOWS_DIR, args[:plugin_name], "#{args[:plugin_name]}-git", "readme.txt")
    raise "Your wordpress plugin does not contain a readme.txt. This is required for listings on wordpress.org." unless File.exists?(readme_path)
    
    # Check "Requires at least"
    required_version = `grep "Requires at least" #{readme_path} | awk '{print $4}'`.strip.to_f
    raise "You must specify a 'Requires at least' version of Wordpress in your readme.txt." if required_version.to_f == 0.0
    
    # Check "Tested up to"
    tested_version = `grep "Tested up to" #{readme_path} | awk '{print $4}'`.strip.to_f
    raise "You must specify a 'Tested up to' version of Wordpress in your readme.txt." if tested_version.to_f == 0.0
    
    # Check "Stable tag"
    stable_tag = `grep "Stable tag" #{readme_path} | awk '{print $3}'`.strip
    raise "You must specify a 'Stable tag' in your readme.txt." if stable_tag == ""
    raise "Your stable tag (#{stable_tag}) must match the version you are attempting to deploy (#{args[:new_version]})" if stable_tag != args[:new_version]
    
    puts "Committing changes to subversion."
    puts `git --git-dir=#{git_dir}/.git svn rebase`
    puts `git --git-dir=#{git_dir}/.git svn dcommit`
    puts ""
    
  end
end