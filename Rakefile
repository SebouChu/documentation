require 'yaml'
require 'open-uri'
require 'nokogiri'
require 'webpacker'
require 'active_support/all'

ENV['NODE_ENV'] ||= ENV['JEKYLL_ENV']
ENV['RAILS_ENV'] ||= ENV['JEKYLL_ENV']

# Fake a few Rails stuff to be able to use the webpacker gem
class Rails
  def self.root
    Pathname.new(Dir.pwd)
  end

  def self.env
    ENV['RAILS_ENV']
  end
end

Webpacker::Compiler.module_eval do
  def webpack_env
    env.merge("NODE_ENV" => @webpacker.env, "WEBPACKER_ASSET_HOST" => "")
  end
end

HOMEPAGE_URL = ENV['HOMEPAGE_URL'] || "https://scalingo.com/"

namespace :assets do
  desc "assets precompilation"
  task :precompile do
    Rake::Task["webpacker:compile"].execute
    exec("jekyll build")
  end
end

$stdout.sync = true

def ensure_log_goes_to_stdout
  old_logger = Webpacker.logger
  Webpacker.logger = ActiveSupport::Logger.new(STDOUT)
  yield
ensure
  Webpacker.logger = old_logger
end

namespace :webpacker do
  desc "Compile JavaScript packs using webpack for production with digests"
  task :compile do
    ensure_log_goes_to_stdout do
      if Webpacker.compile
        # Successful compilation!
      else
        # Failed compilation
        exit!
      end
    end
  end
end

namespace :fetch_from_homepage do
  task all: [:header, :footer]

  task :header do
    doc = fetch_homepage
    toolbar = hack(doc.css("header.mdc-toolbar")).to_xhtml
    sidebar = doc.css("aside").to_xhtml
    template_path = "_includes/header.html"
    File.open(template_path, 'w+') { |file|
      file.write(toolbar)
      file.write(sidebar)
    }
  end

  task :footer do
    doc = fetch_homepage
    footer = doc.css("footer").to_xhtml
    template_path = "_includes/footer.html"
    File.open(template_path, 'w+') { |file|
      file.write(footer)
    }
  end

  # Don't know why but the default SVG button make the page scroll automatically
  # This hack is here to circumvent this problem by replacing the SVG by the same
  # glyph coming from the material design font icon
  def hack doc
    doc.css('button svg').first.replace('<i class="material-icons">menu</i>')
    doc
  end

  def fetch_homepage
    uri = URI.parse(HOMEPAGE_URL)
    str = if uri.user || uri.password
      uri2 = uri.clone
      uri2.user = nil
      uri2.password = nil
      open(uri.to_s, http_basic_authentication: [uri.user, uri.password])
    else
      open(uri)
    end
    Nokogiri::HTML(str)
  end
end

namespace :utility do
  task :unify_filenames do
    Dir["_posts/**/*"].map{|filepath|
      basename = File.basename(filepath)
      dirname = File.dirname(filepath)
      new_basename = basename.gsub(/\d{4}-\d{2}-\d{2}/, "2000-01-01")
      new_filepath = dirname + "/" + new_basename
      if basename != new_basename
        FileUtils.mv filepath, new_filepath
      end
    }
  end
end