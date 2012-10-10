require 'open3'
require 'rake/clean'

Landslide = '/usr/bin/landslide'
Css       = 'stylesheets/folio.css'
Js        = 'javascripts/folio.js'
Sources   = FileList['[^_]*/index.md']

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

[Css, Js].each do |f|
  unless File.exists? f
    touch f
    $stderr.puts red("Missing file #{f} created.")
  end
end

destinations = []

Sources.each do |source|
  destinations << (destination = source.ext('.html'))
  directory = File.dirname source

  file destination => [
    source,
    Css,
    Js,
    *FileList["#{directory}/media/*"],
    *FileList["#{directory}/code/*"],
    Landslide
  ] do
    $stderr.puts blue(directory)

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
task :compile => destinations

task :default => :compile

CLEAN.include destinations
