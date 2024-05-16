require 'rubygems/package_task'
require 'rake/extensiontask'
require 'rake/testtask'

task default: :test
task test: %w[test:pure_ruby test:compiled]

task benchmark: %w[benchmark:native benchmark:pure]

task :print_ruby_version do
  print "#{RUBY_DESCRIPTION}\n\n"
end

namespace :benchmark do
  task native: :print_ruby_version do |t, args|
    puts '# C Extension'
    load File.expand_path("../benchmark/native.rb", __FILE__)
    puts
  end

  task pure: :print_ruby_version do |t, args|
    puts '# Pure Ruby'
    load File.expand_path("../benchmark/pure.rb", __FILE__)
    puts
  end

  task :measure do
    tags = ENV['TAGS'] ? ENV['TAGS'].split(',') : `git tag --list`.split.select { |v| v.match? /\Av1\.[1-9]\.\d\z/ }
    puts 'version,label,utime,stime,cutime,cstime,real'
    tags.each do |tag|
      sh("git checkout -f #{tag} 1>&2")
      sh('git checkout master -- benchmark 1>&2')
      sh('bundle exec rake clobber compile 1>&2')
      sh("ruby #{File.expand_path("../benchmark/measure.rb", __FILE__)}")
    end
  end
end

task compare: :compile do
  require 'jaro_winkler'
  require 'fuzzystringmatch'
  require 'hotwater'
  require 'amatch'
  @ary = [['henka', 'henkan'], ['al', 'al'], ['martha', 'marhta'], ['jones', 'johnson'], ['abcvwxyz', 'cabvwxyz'], ['dwayne', 'duane'], ['dixon', 'dicksonx'], ['fvie', 'ten'], ['San Francisco', 'Santa Monica']]
  table = []
  table << %w[str_1 str_2 jaro_winkler fuzzystringmatch hotwater amatch]
  table << %w[--- --- --- --- --- ---]
  jarow = FuzzyStringMatch::JaroWinkler.create(:native)
  @ary.each do |str_1, str_2|
    table << ["\"#{str_1}\"", "\"#{str_2}\"", JaroWinkler.similarity(str_1, str_2).round(4), jarow.getDistance(str_1, str_2).round(4), Hotwater.jaro_winkler_distance(str_1, str_2).round(4), Amatch::Jaro.new(str_1).match(str_2).round(4)]
  end
  col_len = []
  table.first.length.times{ |i| col_len << table.map{ |row| row[i].to_s.length }.max }
  table.first.each_with_index{ |title, i| "%-#{col_len[i]}s" % title }
  table.each_with_index do |row|
    row.each_with_index do |col, i|
      row[i] = "%-#{col_len[i]}s" % col.to_s
    end
  end
  table.each{|row| puts row.join(' | ')}
end

if RUBY_ENGINE == 'ruby'
  Rake::ExtensionTask.new 'jaro_winkler_ext' do |ext|
    ext.lib_dir = 'lib/jaro_winkler'
    ext.ext_dir = 'ext/jaro_winkler'
  end
else
  task :compile do
    puts 'Can not compile C extension, fallback to pure Ruby version.'
  end
end

namespace :test do
  Rake::TestTask.new(:compiled => :compile) do |t|
    t.libs << 'test'
    t.test_files = FileList['test/test_jaro_winkler.rb']
    t.verbose = true
  end

  Rake::TestTask.new(:pure_ruby) do |t|
    t.libs << 'test'
    t.test_files = FileList['test/test_pure_ruby.rb']
    t.verbose = true
  end
end

%w[jaro_winkler jaro_winkler.java]
  .map { |name| Gem::Specification.load(File.expand_path("../#{name}.gemspec", __FILE__)) }
  .each { |spec| Gem::PackageTask.new(spec).define }

task 'CHANGELOG.md' do
  sh 'conventional-changelog -p angular -i CHANGELOG.md -s'
end
