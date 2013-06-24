desc "create a new post"
task :new  do |t, args|
    content_type = ENV["CONTENT_TYPE"] || 'markdown'
    content_type.downcase!
    if !['markdown', 'textile'].member?(content_type)
        STDERR.puts "unsupported CONTENT_TYPE; #{content_type}"
        exit 1
    end

    print "title: "
    title = STDIN.readline
    title.strip!

    now = Time.now
    today = Date.today.to_s
    normal = title.dup
    normal.gsub!(/([[:space:]]|[^[:alpha:]])+/, '-')
    normal.gsub!(/^-+/, '')
    normal.gsub!(/-+$/, '')
    normal.downcase!

    create = lambda do |filename, content|
        begin
            file = File.open(filename, 'w')
            file.print(content)
            file.close
        rescue
            begin
                File.delete(filename)
            rescue Errno::ENOENT
                return
            rescue Exception => err
                STDERR.puts "unable to remove post"
                raise err
            end
        end

        until finalized do
            sh "$EDITOR #{filename}"
            categories = `grep '^categories:' #{filename} | awk -F: '{print $2}'`
            categories.strip!
            categories = categories.split
            puts categories.inspect
        end
    end
    filename = "_posts/#{today}-#{normal}.#{content_type}"
    create.call(filename, (<<"HEAD"))
---
layout: post
title:  "#{title.gsub(/(["\\])/) do |c| "\\" + c end}"
date:   #{now}
categories:
---
HEAD
end

task :serve
task :serve do
    sh "bundle exec jekyll serve"
end

task :bundle
task :bundle do
    sh "bundle > /dev/null"
end
