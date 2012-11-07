require 'rubygems'
require 'github_api'

# This file stores $github_login and $github_password which are
# used for publishing a download
if File.exist?('Rakefile.config')
  load 'Rakefile.config'
end

# The user and repository name on GitHub. Used when publishing a download.
$github_user='ase-lab'
$github_repo='ZeroConfigNetServices'

desc 'Archives both the Publisher and the Browser'
task :archive => ['browser:archive', 'publisher:archive']

namespace :browser do

  desc 'Archive Bonjour Browser'
  task :archive do
    cd 'Bonjour Browser/publish' do
      system('tar cvzf "Bonjour Browser.tar.gz" --exclude="Bonjour Browser.tar.gz" *')
    end
  end

end

namespace :publisher do

  desc 'Archive Bonjour Publisher'
  task :archive do
    cd 'Bonjour Publisher/publish' do
      system('tar cvzf "Bonjour Publisher.tar.gz" --exclude="Bonjour Publisher.tar.gz"  *')
    end
  end

end

desc 'Initialize and update all submodules recursively'
task :init do
  system('git submodule update --init --recursive')
  system('git submodule foreach --recursive "git checkout master"')
end

desc 'Pull all submodules recursively'
task :pull => :init do
  system('git submodule foreach --recursive git pull')
end

def publish(name, version)
  file = name + '/publish/' + name  + '.tar.gz'
  name = name + '-' + version + '.tar.gz'
  description = name + ' Version ' + version

  size = File.size(file)

  github = Github.new(:user => $github_user,
                      :repo => $github_repo,
                      :login => $github_login,
                      :password => $github_password)
  res = github.repos.downloads.create $github_user, $github_repo,
    'name' => name,
    'size' => size,
    'description' => description,
    'content_type' => 'application/x-gzip'
  github.repos.downloads.upload res, file
end

desc 'Publish a new version to GitHub'
task :publish, :version do |t, args|
  if !args[:version]
    puts('Usage: rake publish[version]');
    exit(1)
  end
  if !defined? $github_login
    puts('$github_login is not set');
    exit(1)
  end
  if !defined? $github_password
    puts('$github_password is not set');
    exit(1)
  end
  version = args[:version]
  # check that version is newer than current_version
  current_version = open('Version').gets.strip
  if Gem::Version.new(version) < Gem::Version.new(current_version)
    puts('New version (' + version + ') is smaller than current version (' + current_version + ')')
    exit(1)
  end
  # write version into versionfile
  File.open('Version', 'w') {|f| f.write(version) }

  Rake::Task['archive'].invoke
  
  # build was successful, increment version and push changes
  system('git add Version')
  system('git commit -m "Bump version to ' + version + '"')
  system('git tag -a v' + version + ' -m "Version ' + version + '."')
  system('git push')
  system('git push --tags')

  publish('Bonjour Browser', version)
  publish('Bonjour Publisher', version)
end