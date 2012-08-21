require "rake"

require "fileutils"

WORKFLOWS_DIR = File.join(File.expand_path(File.dirname(__FILE__)), "workflows")

namespace :plugins do
  namespace :convert_to_git do
    desc "Generate the workspace and svn authors file for a wordpress repo"
    task :prepare, :plugin_name do |t, args|
      raise "Must specify name of plugin" if args[:plugin_name].nil?
      
      # Create git/svn directories
      workflow_dir = File.join(WORKFLOWS_DIR, args[:plugin_name])
      git_dir = File.join(WORKFLOWS_DIR, args[:plugin_name], "#{args[:plugin_name]}-git")
      svn_dir = File.join(WORKFLOWS_DIR, args[:plugin_name], "#{args[:plugin_name]}-svn")
      FileUtils.mkdir_p workflow_dir
      
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
      puts "Please authors with their github email addresses, then run 'rake plugins:convert_to_git:run[#{args[:plugin_name]}]'"
      puts ""
    end
    
    desc "Convert a wordpress-hosted SVN repository to a local git repo"
    task :run, :plugin_name do |t, args|
      raise "Must specify name of plugin" if args[:plugin_name].nil?
      authors_path = File.join(WORKFLOWS_DIR, args[:plugin_name], "authors.txt")
      raise "No authors file. Please run 'rake plugins:convert_to_git:prepare[#{args[:plugin_name]}]'" unless File.exists?(authors_path)
      
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
      
      # You can run `git update-ref refs/remotes/git-svn refs/remotes/origin/master` if histories don't match.
      
      puts "Git repository generated. You may use it locally or push it to github, if you like."
      puts ""
    end
  end
end