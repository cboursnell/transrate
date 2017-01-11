require 'rake/testtask'
require 'rake/extensiontask'
require 'bundler/setup'
require 'bindeps'
require 'rbconfig'

Rake::ExtensionTask.new('transrate') do |ext|
  ext.lib_dir = "lib/transrate"
end

Rake::TestTask.new do |t|
  t.libs << 'test'
end

Rake::TestTask.new do |t|
  t.name = :recip
  t.libs << 'test'
  t.test_files = ['test/test_reciprocal.rb']
end

Rake::TestTask.new do |t|
  t.name = :comp
  t.libs << 'test'
  t.test_files = ['test/test_comp_metrics.rb']
end

Rake::TestTask.new do |t|
  t.name = :contig_metrics
  t.libs << 'test'
  t.test_files = ['test/test_contig_metrics.rb']
end

Rake::TestTask.new do |t|
  t.name = :read
  t.libs << 'test'
  t.test_files = ['test/test_read_metrics.rb']
end

Rake::TestTask.new do |t|
  t.name = :bowtie
  t.libs << 'test'
  t.test_files = ['test/test_bowtie.rb']
end

Rake::TestTask.new do |t|
  t.name = :rater
  t.libs << 'test'
  t.test_files = ['test/test_transrater.rb']
end

Rake::TestTask.new do |t|
  t.name = :bin
  t.libs << 'test'
  t.test_files = ['test/test_bin.rb']
end

Rake::TestTask.new do |t|
  t.name = :contig
  t.libs << 'test'
  t.test_files = ['test/test_contig.rb']
end

Rake::TestTask.new do |t|
  t.name = :assembly
  t.libs << 'test'
  t.test_files = ['test/test_assembly.rb']
end

Rake::TestTask.new do |t|
  t.name = :snap
  t.libs << 'test'
  t.test_files = ['test/test_snap.rb']
end

Rake::TestTask.new do |t|
  t.name = :salmon
  t.libs << 'test'
  t.test_files = ['test/test_salmon.rb']
end

Rake::TestTask.new do |t|
  t.name = :optimiser
  t.libs << 'test'
  t.test_files = ['test/test_optimiser.rb']
end

Rake::TestTask.new do |t|
  t.name = :cmdline
  t.libs << 'test'
  t.test_files = ['test/test_cmdline.rb']
end



desc "Run tests"
task :default => :test

# PACKAGING

require 'bundler/setup'

PACKAGE_NAME = "transrate"
VERSION = `bundle exec bin/transrate -v`.chomp
TRAVELING_RUBY_VERSION = "20150715"
TARFILE = "rel-#{TRAVELING_RUBY_VERSION}.tar.gz"
TRAVELING_RUBY_URL = "https://github.com/phusion/traveling-ruby/archive/rel-#{TRAVELING_RUBY_VERSION}.tar.gz"
TRAVELING_RUBY_PACKAGE = File.join("packaging", "packaging", TARFILE)

desc "Package transrate"
task :package => ['package:linux']

namespace :package do

  desc "Package transrate for linux x86_64"
  task :linux => [:bundle_install, TRAVELING_RUBY_PACKAGE] do
    create_package()
  end

  file TRAVELING_RUBY_PACKAGE do
    download_runtime()
  end

  desc "Install gems to local directory"
  task :bundle_install do
    if RUBY_VERSION !~ /^2\.2\./
      msg "You can only 'bundle install' using Ruby 2.2, because that's"
      msg << " what Traveling Ruby uses."
      abort msg
    end
    Bundler.with_clean_env do
      sh "env BUNDLE_IGNORE_CONFIG=1 bundle install --path packaging/vendor"
    end
    sh "rm -f packaging/vendor/*/*/cache/*"
  end

end

def create_package
  package_name = "#{PACKAGE_NAME}-#{VERSION}"
  package_dir = File.join("packaging", package_name)
  sh "rm -rf #{package_dir}"
  sh "mkdir -p #{package_dir}/lib/app"

  # copy things from gem to package
  sh "cp -r bin #{package_dir}/lib/" # bin
  sh "cp -r lib #{package_dir}/lib/" # lib
  sh "cp -r deps #{package_dir}/lib/" # deps
  sh "cp files.txt #{package_dir}/lib/" # deps
  sh "cp Gemfile* #{package_dir}/lib/" # Gemfiles
  sh "cp *gemspec #{package_dir}/lib/" # gemspec

  # download travelling ruby
  sh "mkdir #{package_dir}/lib/ruby"
  sh "tar -xzf packaging/packaging/#{TARFILE} -C #{package_dir}/lib/ruby"
  sh "cp packaging/transrate #{package_dir}/transrate"
  sh "cp -pR packaging/vendor/* #{package_dir}/lib/"
  sh "cp Gemfile Gemfile.lock #{package_dir}/lib/"

  # install binary dependencies
  sh "mkdir -p #{package_dir}/bin"
  sh "mkdir -p #{package_dir}/lib"
  sh "wget -nc https://github.com/HibberdLab/snap/releases/download/v100/snap-aligner.tar.gz"
  sh "wget -nc https://github.com/Blahah/transrate-tools/releases/download/v1.0.0/bam-read_v1.0.0_linux.tar.gz"
  sh "wget -nc https://github.com/COMBINE-lab/salmon/releases/download/v0.7.2/Salmon-0.7.2_linux_x86_64.tar.gz"
  sh "find . -maxdepth 1 -name '*.tar.gz' -exec tar xzf '{}' \\;" # unpack
  sh "cp snap-aligner #{package_dir}/bin/."
  sh "cp bam-read #{package_dir}/bin/."
  sh "cp Salmon-0.7.2_linux_x86_64/bin/salmon #{package_dir}/bin/."
  sh "cp -r Salmon-0.7.2_linux_x86_64/lib/* #{package_dir}/lib/."

  sh "cp packaging/libruby.* #{package_dir}/lib/."

  sh "mkdir #{package_dir}/lib/.bundle"
  sh "cp packaging/bundler-config #{package_dir}/lib/.bundle/config"

  # create package
  if !ENV['DIR_ONLY']
    sh "cd packaging && tar -czf #{package_name}.tar.gz #{package_name}"
    # sh "rm -rf #{package_dir}"
  end
  # clean up
  # sh "rm -rf packaging/vendor packaging/bindeps .bundle"
end

def download_runtime
  sh "mkdir -p packaging/packaging && " +
     "cd packaging/packaging && " +
     "curl -L -O --fail " + TRAVELING_RUBY_URL
end
