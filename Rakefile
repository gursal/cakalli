require 'find'
require 'open3'
require 'rake/clean'

Landslide = '/usr/bin/landslide'
Css       = 'stylesheets/folio.css'
Js        = 'javascripts/folio.js'

def colorize(text, color_code)
  "\033[1m\033[38;5;#{color_code}m#{text}\033[0m"
end

def red(text);   colorize(text, 198); end
def green(text); colorize(text, 120); end
def blue(text);  colorize(text, 117); end

def run(*cmd)
  Open3.popen3(*cmd) do |_, out_p, err_p, thr_p|
    response, error = [out_p, err_p].map { |io| io.read.strip }
    status          = thr_p.value.exitstatus
    ok              = (status == 0)

    yield ok, response, error if block_given?

    status
  end
end

unless File.exist? Landslide
  $stderr.puts read("#{Landslide} not found!")
  abort
end

# Css and Js files are not mandatory.
[Css, Js].each do |f|
  unless File.exists? f
    touch f
    $stderr.puts red("Missing file #{f} created.")
  end
end

# Locate folio sources in the form of 'foo/bar/index.md'.
Sources   = []
FileList['[^_.]*'].select { |path| FileTest.directory?(path) }.each do |dir|
  Find.find(dir) do |path|
    basename = File.basename(path)
    if FileTest.directory?(path)
      Find.prune if %w(. _).any? { |prefix| basename[0] == prefix }
    elsif basename == 'index.md' && File.open(path, &:readline).start_with?('#')
      Sources << path
    end
  end
end

# Compute destinations and create a file task for each destination.
Destinations = []
Sources.each do |source|
  Destinations << (destination = source.ext('.html'))
  directory = File.dirname source
  label = directory

  file destination => [
    source,
    Css,
    Js,
    *FileList["#{directory}/media/*"], # media files, i.e. images
    *FileList["#{directory}/code/*"],  # code files
    Landslide
  ] do
    $stderr.puts blue(label)

    run(*%W[
      #{Landslide}
        --embed
        --linenos no
        --css #{Css}
        --js #{Js}
        --theme light
        --destination #{destination}
        #{source}
    ]) do |ok, response, error|
      if not ok
        rm_f destination
        $stderr.puts red("landslide error: %s" % error)
      end
    end
  end
end

desc 'Compile folios.'
task :compile => Destinations

task :default => :compile

CLEAN.include Destinations
