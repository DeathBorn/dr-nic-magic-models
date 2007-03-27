require 'rubygems'
require 'rake'
require 'rake/testtask'
require 'rake/rdoctask'
require 'rake/packagetask'
require 'rake/gempackagetask'
require 'rake/contrib/rubyforgepublisher'
require File.join(File.dirname(__FILE__), 'lib', 'dr_nic_magic_models', 'version')

PKG_BUILD     = ENV['PKG_BUILD'] ? '.' + ENV['PKG_BUILD'] : ''
PKG_NAME      = 'dr_nic_magic_models'
PKG_VERSION   = DrNicMagicModels::VERSION::STRING + PKG_BUILD
PKG_FILE_NAME = "#{PKG_NAME}-#{PKG_VERSION}"

RELEASE_NAME  = "REL #{PKG_VERSION}"

RUBY_FORGE_PROJECT = "magicmodels"
RUBY_FORGE_USER    = "nicwilliams"

PKG_FILES = FileList[
    "lib/**/*", "test/**/*", "examples/**/*", "doc/**/*", "website/**/*", "scripts/**/*", "[A-Z]*", "install.rb", "Rakefile"
].exclude(/\bCVS\b|~$/)


desc "Default Task"
task :default => [ :test_sqlite ] 
task :test    => [ :test_sqlite ]

# Run the unit tests

for adapter in %w( sqlite mysql postgresql ) # UNTESTED - postgresql sqlite sqlite3 firebird sqlserver sqlserver_odbc db2 oracle sybase openbase )
  Rake::TestTask.new("test_#{adapter}") { |t|
    t.libs << "test" << "test/connections/native_#{adapter}"
    t.pattern = "test/*_test{,_#{adapter}}.rb"
    t.verbose = true
  }
end

SCHEMA_PATH = File.join(File.dirname(__FILE__), *%w(test fixtures db_definitions))

desc 'Build the MySQL test databases'
task :build_mysql_databases do 
  puts File.join(SCHEMA_PATH, 'mysql.sql')
  %x( mysqladmin -u root create "#{PKG_NAME}_unittest" )
  cmd = "mysql -u root #{PKG_NAME}_unittest < \"#{File.join(SCHEMA_PATH, 'mysql.sql')}\""
  puts "#{cmd}\n"
  %x( #{cmd} )
end

desc 'Drop the MySQL test databases'
task :drop_mysql_databases do 
  %x( mysqladmin -u root -f drop "#{PKG_NAME}_unittest" )
end

desc 'Rebuild the MySQL test databases'
task :rebuild_mysql_databases => [:drop_mysql_databases, :build_mysql_databases]

desc 'Build the sqlite test databases'
task :build_sqlite_databases do 
  puts File.join(SCHEMA_PATH, 'sqlite.sql')
  %x( sqlite3 test.db < test/fixtures/db_definitions/sqlite.sql )
end

desc 'Drop the sqlite test databases'
task :drop_sqlite_databases do 
  %x( rm -f test.db )
end

desc 'Rebuild the sqlite test databases'
task :rebuild_sqlite_databases => [:drop_sqlite_databases, :build_sqlite_databases]

desc 'Build the PostgreSQL test databases'
task :build_postgresql_databases do 
  %x( createdb -U postgres "#{PKG_NAME}_unittest" )
  %x( psql "#{PKG_NAME}_unittest" postgres -f "#{File.join(SCHEMA_PATH, 'postgresql.sql')}" )
end

desc 'Drop the PostgreSQL test databases'
task :drop_postgresql_databases do 
  %x( dropdb -U postgres "#{PKG_NAME}_unittest" )
end

desc 'Rebuild the PostgreSQL test databases'
task :rebuild_postgresql_databases => [:drop_postgresql_databases, :build_postgresql_databases]

# Generate the RDoc documentation

Rake::RDocTask.new { |rdoc|
  rdoc.rdoc_dir = 'doc'
  rdoc.title    = "Dr Nic's Magic Models - Invisible validations, assocations and Active Record models themselves!"
  rdoc.options << '--line-numbers' << '--inline-source' << '-A cattr_accessor=object'
  rdoc.template = "#{ENV['template']}.rb" if ENV['template']
  rdoc.rdoc_files.include('README', 'CHANGELOG')
  rdoc.rdoc_files.include('lib/**/*.rb')
}

# Enhance rdoc task to copy referenced images also
task :rdoc do
  FileUtils.mkdir_p "doc/files/examples/"
end


# Create compressed packages

dist_dirs = [ "lib", "test", "examples", "website", "scripts"]

spec = Gem::Specification.new do |s|
  s.name = PKG_NAME
  s.version = PKG_VERSION
  s.summary = "Invisible validations, assocations and Active Record models themselves!"
  s.description = %q{Associations and validations are automagically available to your ActiveRecord models. Model classes themselves are automagically generated if you haven't defined them already!}

  s.files = [ "Rakefile", "install.rb", "README", "CHANGELOG" ]
  dist_dirs.each do |dir|
    s.files = s.files + Dir.glob( "#{dir}/**/*" ).delete_if { |item| item.include?( "\.svn" ) }
  end

  s.require_path = 'lib'
  s.autorequire = 'dr_nic_magic_models'

  s.has_rdoc = true
  s.extra_rdoc_files = %w( README )
  s.rdoc_options.concat ['--main',  'README']
  
  s.author = "Dr Nic Williams"
  s.email = "drnicwilliams@gmail.com"
  s.homepage = "http://magicmodels.rubyforge.org"
  s.rubyforge_project = "magicmodels"
end
  
Rake::GemPackageTask.new(spec) do |p|
  p.gem_spec = spec
  p.need_tar = false
  p.need_zip = false
end

task :lines do
  lines, codelines, total_lines, total_codelines = 0, 0, 0, 0

  for file_name in FileList["lib/**/*.rb"]
    next if file_name =~ /vendor/
    f = File.open(file_name)

    while line = f.gets
      lines += 1
      next if line =~ /^\s*$/
      next if line =~ /^\s*#/
      codelines += 1
    end
    puts "L: #{sprintf("%4d", lines)}, LOC #{sprintf("%4d", codelines)} | #{file_name}"
    
    total_lines     += lines
    total_codelines += codelines
    
    lines, codelines = 0, 0
  end

  puts "Total: Lines #{total_lines}, LOC #{total_codelines}"
end


# Publishing ------------------------------------------------------

desc "Publish the release files to RubyForge."
task :release => [ :package ] do
  `ruby scripts/rubyforge login`

  for ext in %w( gem tgz zip )
    release_command = "ruby scripts/rubyforge add_release #{PKG_NAME} #{PKG_NAME} 'REL #{PKG_VERSION}' pkg/#{PKG_NAME}-#{PKG_VERSION}.#{ext}"
    puts release_command
    system(release_command)
  end
end
