require 'rake'
require 'rake/testtask'
require 'rake/rdoctask'
require 'rake/gempackagetask'
require 'cucumber/rake/task'

desc 'Default: run unit tests.'
task :default => [:test, :cucumber]

desc 'Test the hoptoad_notifier gem.'
Rake::TestTask.new(:test) do |t|
  t.libs << 'lib'
  t.pattern = 'test/**/*_test.rb'
  t.verbose = true
end

desc 'Run ginger tests'
task :ginger do
  $LOAD_PATH << File.join(*%w[vendor ginger lib])
  ARGV.clear
  ARGV << 'test'
  load File.join(*%w[vendor ginger bin ginger])
end

begin
  require 'yard'
  YARD::Rake::YardocTask.new do |t|
    t.files   = ['lib/**/*.rb', 'TESTING.rdoc']
  end
rescue LoadError
end

GEM_ROOT     = File.dirname(__FILE__).freeze
VERSION_FILE = File.join(GEM_ROOT, 'lib', 'hoptoad_notifier', 'version')

require VERSION_FILE

gemspec = Gem::Specification.new do |s|
  s.name        = %q{hoptoad_notifier}
  s.version     = HoptoadNotifier::VERSION
  s.summary     = %q{Send your application errors to our hosted service and reclaim your inbox.}

  s.files        = FileList['[A-Z]*', 'generators/**/*.*', 'lib/**/*.rb',
                            'test/**/*.rb', 'rails/**/*.rb', 'script/*']
  s.require_path = 'lib'
  s.test_files   = Dir[*['test/**/*_test.rb']]

  s.has_rdoc         = true
  s.extra_rdoc_files = ["README.rdoc"]
  s.rdoc_options = ['--line-numbers', "--main", "README.rdoc"]

  s.authors = ["thoughtbot, inc"]
  s.email   = %q{support@hoptoadapp.com}
  s.homepage = "http://www.hoptoadapp.com"

  s.platform = Gem::Platform::RUBY
end

Rake::GemPackageTask.new gemspec do |pkg|
  pkg.need_tar = true
  pkg.need_zip = true
end

desc "Clean files generated by rake tasks"
task :clobber => [:clobber_rdoc, :clobber_package]

desc "Generate a gemspec file"
task :gemspec do
  File.open("#{gemspec.name}.gemspec", 'w') do |f|
    f.write gemspec.to_ruby
  end
end

LOCAL_GEM_ROOT = File.join(GEM_ROOT, 'tmp', 'local_gems').freeze
RAILS_VERSIONS = IO.read('SUPPORTED_RAILS_VERSIONS').strip.split("\n")
LOCAL_GEMS = [['sham_rack', nil], ['capistrano', nil], ['sqlite3-ruby', nil]] +
  RAILS_VERSIONS.collect { |version| ['rails', version] }

task :vendor_test_gems do
  LOCAL_GEMS.each do |gem_name, version|
    gem_file_pattern = [gem_name, version || '*'].compact.join('-')
    version_option = version ? "-v #{version}" : ''
    pattern = File.join(LOCAL_GEM_ROOT, 'gems', "#{gem_file_pattern}")
    existing = Dir.glob(pattern).first
    unless existing
      command = "gem install -i #{LOCAL_GEM_ROOT} --no-ri --no-rdoc #{version_option} #{gem_name}"
      puts "Vendoring #{gem_file_pattern}..."
      unless system(command)
        $stderr.puts "Command failed: #{command}"
      end
    end
  end
end

Cucumber::Rake::Task.new(:cucumber) do |t|
  t.fork = true
  t.cucumber_opts = ['--format', (ENV['CUCUMBER_FORMAT'] || 'progress')]
end

task :cucumber => [:gemspec, :vendor_test_gems]

OLD_RAILS_VERSIONS = RAILS_VERSIONS[0...-1]

namespace :cucumber do
  namespace :rails do
    OLD_RAILS_VERSIONS.each do |version|
      desc "Test integration of the gem with Rails #{version}"
      task version do
        ENV['RAILS_VERSION'] = version
        system("cucumber --format progress features/rails.feature")
      end
    end

    desc "Test integration of the gem with all Rails versions"
    task :all => [:cucumber, *OLD_RAILS_VERSIONS]
  end
end
