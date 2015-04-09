require 'bundler/setup'
require 'rspec/core/rake_task'

Bundler::GemHelper.install_tasks
RSpec::Core::RakeTask.new(:spec)

module Helpers
  module_function
  def binary_gemspec(platform = Gem::Platform.local)
    gemspec = eval(File.read('libv8.gemspec'))
    gemspec.platform = platform
    gemspec
  end

  def binary_gem_name(platform = Gem::Platform.local)
    File.basename binary_gemspec(platform).cache_file
  end
end

desc "compile v8 via the ruby extension mechanism"
task :compile => :devkit do
  sh "ruby ext/libv8/extconf.rb"
end

desc "build a binary gem #{Helpers.binary_gem_name}"
task :binary => :compile do
  gemspec = Helpers.binary_gemspec
  gemspec.extensions.clear

  # We don't need most things for the binary
  gemspec.files = []
  gemspec.files += ['lib/libv8.rb', 'lib/libv8/version.rb']
  gemspec.files += ['ext/libv8/arch.rb', 'ext/libv8/location.rb', 'ext/libv8/paths.rb']
  gemspec.files += ['ext/libv8/.location.yml']

  # V8
  gemspec.files += Dir['vendor/v8/include/*']
  gemspec.files += Dir['vendor/v8/out/**/*.a']

  FileUtils.chmod 0644, gemspec.files
  FileUtils.mkdir_p 'pkg'

  package = if Gem::VERSION < '2.0.0'
    Gem::Builder.new(gemspec).build
  else
    require 'rubygems/package'
    Gem::Package.build(gemspec)
  end

  FileUtils.mv(package, 'pkg')
end

task :clean_submodules do
  sh "git submodule --quiet foreach git reset --hard"
  sh "git submodule --quiet foreach git clean -df"
end

desc "clean up artifacts of the build"
task :clean => [:clean_submodules] do
  sh "rm -rf pkg"
  sh "git clean -df"
end

task :devkit do
  begin
    if RUBY_PLATFORM =~ /mingw/
      require "devkit"
    end
  rescue LoadError => e
    abort "Failed to activate RubyInstaller's DevKit required for compilation."
  end
end

task :default => [:compile, :spec]
task :build => [:clean_submodules]
