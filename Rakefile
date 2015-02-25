require 'fileutils'

def clone_repo_at_branch(repo, branch)
  unless system("git clone -b #{branch} --single-branch git://github.com.com/cappity-cappity-capstone/#{repo}")
    fail("Unable to clone branch '#{branch}' for #{repo}")
  end
end

%w(backend ui auth backend-nginx).each do |repo|
  namespace repo.to_sym do
    desc "Clear the '#{repo}' state"
    task :clean do
      FileUtils.rm_rf(repo)
    end

    task :link do
      source = ENV["#{repo.upcase}_LOCATION"] || File.expand_path(File.join('..', repo))
      dest = File.expand_path(File.join('.', repo))

      unless File.exist?(dest)
        puts "Symlinking #{source} to #{dest}"
        FileUtils.ln_sf(source, dest)
      end
    end

    task :build do
      Dir.chdir(repo) do
        system("docker build -t #{repo} .") || fail("Unable to build #{repo}")
      end
    end
  end
end

desc 'Clear the state of the repos'
multitask clean: %w(backend:clean ui:clean auth:clean backend-nginx:clean)

desc 'Clone the repos'
multitask link: %w(backend:link ui:link auth:link backend-nginx:link)

desc 'Ensure the docker server is running'
task :ensure_docker_running do
  system('docker info') || fail('docker is not running on your host')
end

task setup: %w(ensure_docker_running link)

desc 'Build the repos'
task build: %w(setup backend:build ui:build auth:build backend-nginx:build)

desc 'Run the auth server'
task auth_up: :setup do
  puts "Please hit http://#{`boot2docker ip`.chomp}:1337 in your browser when the build completes"
  system('fig --file authfig.yml up') || fail('Unable to start fig')
end

desc 'Run the backend server'
task auth_up: :setup do
  puts "Please hit http://#{`boot2docker ip`.chomp}:4567 in your browser when the build completes"
  system('fig --file backendfig.yml up') || fail('Unable to start fig')
end
