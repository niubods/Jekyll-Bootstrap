#

task :default => :server

desc 'create new post. args: type (post|draft), title'
# rake new type=(draft|post) title="New post title goes here" slug="slug-override-title"
task :new do
  type = ENV["type"] || "post"
  title = ENV["title"] || "New Title"
  slug = (ENV["slug"] || title).gsub(' ','-').downcase

  if type == "post"
    TARGET_DIR = "_posts"
  else
    TARGET_DIR = "_drafts"
  end

  filename = "#{Time.new.strftime('%Y-%m-%d')}-#{slug}.markdown"
  
  path = File.join(TARGET_DIR, filename)
  post = <<-HTML
--- 
layout: TYPE
title: "TITLE"
date: DATE
---

HTML
  post.gsub!('TITLE', title).gsub!('DATE', Time.new.to_s).gsub!('TYPE', type)
  File.open(path, 'w') do |file|
    file.puts post
  end
  puts "new #{type} generated in #{path}"
  system "mate #{path}"
end

desc 'Clean up generated site'
task :clean do
  puts "clean up"
  cleanup
end

desc 'Minify all assets'
task :jammit do
  puts "jamming assets"
  system "jammit -o assets -c _assets.yml"
end

desc 'Build site with Jekyll'
task :build => [:clean, :jammit] do
  puts "running jekyll"
  system 'jekyll'
end

desc 'Start server with --auto'
task :server => :clean do
  jekyll('--server --auto')
end

desc 'Check links for site already running on localhost:4000'
task :check_links do
  begin
    require 'anemone'
    root = 'http://localhost:4000/'
    Anemone.crawl(root, :discard_page_bodies => true) do |anemone|
      anemone.after_crawl do |pagestore|
        broken_links = Hash.new { |h, k| h[k] = [] }
        pagestore.each_value do |page|
          if page.code != 200
            referrers = pagestore.pages_linking_to(page.url)
            referrers.each do |referrer|
              broken_links[referrer] << page
            end
          end
        end
        broken_links.each do |referrer, pages|
          puts "#{referrer.url} contains the following broken links:"
          pages.each do |page|
            puts "  HTTP #{page.code} #{page.url}"
          end
        end
      end
    end

  rescue LoadError
    abort 'Install anemone gem: gem install anemone'
  end
end

def cleanup
  system 'rm -rf publish assets'
end

def jekyll(opts = '')
  system 'jekyll ' + opts
end
