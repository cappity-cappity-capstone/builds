require 'fileutils'

def clone_repo_at_branch(repo, branch)
  unless system("git clone -b #{branch} --single-branch git://github.com.com/cappity-cappity-capstone/#{repo}")
    fail("Unable to clone branch '#{branch}' for #{repo}")
  end
end

%w(backend ui).each do |repo|
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
  end
end

desc 'Clear the state of both repos'
multitask clean: %w(backend:clean ui:clean)

desc 'Clone both repos'
multitask link: %w(backend:link ui:link)

desc 'Ensure the docker server is running'
task :ensure_docker_running do
  system('docker info') || fail('docker is not running on your host')
end

desc 'Build both repos'
task build: %w(ensure_docker_running link) do
  system('fig build') || fail('Unable to build repos')
end

desc 'Run the server'
task :up do
  puts "Please hit http://#{`boot2docker ip`.chomp}:1337 in your browser when the build completes"
  system('fig up') || fail('Unable to start fig')
end
