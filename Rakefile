require 'html-proofer'

task :deploy do
  user = 'cacqw7_clettenberg'
  server = 'ssh.phx.nearlyfreespeech.net'
  sh "rsync -crz --delete _site/ #{user}@#{server}:/home/public"
end

task :test do
  sh "bundle exec jekyll build"
  HTMLProofer.check_directory("./_site", {
    :url_ignore => [/linkedin.com/],
  }).run
end
