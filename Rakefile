require 'time'

desc "blog post tasks"
namespace :posts do
    desc "list posts"
    task :index do
        puts JekyllTask::Post.select
    end

    desc "create a new post"
    task :new  do
        content_type = ENV["CONTENT_TYPE"] || 'markdown'
        content_type.downcase!
        if !['markdown', 'textile'].member?(content_type)
            STDERR.puts "unsupported CONTENT_TYPE; #{content_type}"
            exit 1
        end

        print "title: "
        title = STDIN.readline
        title.strip!

        post = JekyllTask::Post.new(title, content_type)
        filename = post.filename
        content = post.default_content
        begin
            file = File.open(filename, 'w')
            file.print(content)
            file.close
        rescue Exception => err
            STDERR.puts "error writing post file: #{err}"
            begin
                File.delete(filename)
            rescue Errno::ENOENT
            rescue Exception => err
                STDERR.puts "error removing post file: #{err}"
            end
            exit 1
        end

        finalized = false
        until finalized do
            sh "$EDITOR #{filename}"
            action = nil
            while action.nil? do
                print "done?[Yes/no/edit] "
                resp = STDIN.readline.chomp.strip.downcase
                resp = 'yes' if resp.empty?
                case
                when 'yes'.start_with?(resp)
                    action = :finalize
                when 'no'.start_with?(resp)
                    action = :forget
                when 'edit'.start_with?(resp)
                    action = :rework
                end
            end
            break if action == :forget
            finalized = action == :finalize
        end

        if !finalized
            begin
                File.delete(filename)
            rescue Errno::ENOENT
                STDERR.puts "post file already removed"
            rescue Exception => err
                STDERR.puts "error removing post file: #{err}"
            else
                STDERR.puts "removed post file"
            end
            exit 1
        end
    end
end

desc "create a new post"
task :new  do
    content_type = ENV["CONTENT_TYPE"] || 'markdown'
    content_type.downcase!
    if !['markdown', 'textile'].member?(content_type)
        STDERR.puts "unsupported CONTENT_TYPE; #{content_type}"
        exit 1
    end

    print "title: "
    title = STDIN.readline
    title.strip!

    post = JekyllTask::Post.new(title, content_type)
    filename = post.filename
    content = post.default_content
    begin
        file = File.open(filename, 'w')
        file.print(content)
        file.close
    rescue Exception => err
        STDERR.puts "error writing post file: #{err}"
        begin
            File.delete(filename)
        rescue Errno::ENOENT
        rescue Exception => err
            STDERR.puts "error removing post file: #{err}"
        end
        exit 1
    end

    finalized = false
    until finalized do
        sh "$EDITOR #{filename}"
        action = nil
        while action.nil? do
            print "done?[Yes/no/edit] "
            resp = STDIN.readline.chomp.strip.downcase
            resp = 'yes' if resp.empty?
            case
            when 'yes'.start_with?(resp)
                action = :finalize
            when 'no'.start_with?(resp)
                action = :forget
            when 'edit'.start_with?(resp)
                action = :rework
            end
        end
        break if action == :forget
        finalized = action == :finalize
    end

    if !finalized
        begin
            File.delete(filename)
        rescue Errno::ENOENT
            STDERR.puts "post file already removed"
        rescue Exception => err
            STDERR.puts "error removing post file: #{err}"
        else
            STDERR.puts "removed post file"
        end
        exit 1
    end
end

desc 'compile and serve the blog'
task :serve do
    sh "bundle exec jekyll serve"
end

desc 'compile and serve the blog'
namespace :serve do

    desc 'compile and serve the blog (including drafts)'
    task :drafts do
      sh "bundle exec jekyll serve --drafts"
    end
end


task :bundle
task :bundle do
    sh "bundle > /dev/null"
end

module JekyllTask

    class Post
        POST_DIR = "_posts"
        DRAFT_DIR = "_drafts"

        attr_reader :created, :title, :content_type

        def initialize(title, content_type = 'markdown')
            @created = Time.now
            @title = title.to_s
            @content_type = (content_type || 'markdown').downcase
        end

        def self.select(options = {})
            options ||= {}

            files = Dir.glob("#{POST_DIR}/*")
            names = files.map do |f| File.basename(f) end

            order = options[:order]
            order ||= '-created'
            order.downcase!
            case order
            when "-created"
                names.sort! do |a, b| b <=> a end
            when "created", "+created"
                names.sort!
            end
            names
        end

        def self.parse_filename(filename)
            name = File.basename(filename)
            name.match(/^()-(.*).([^.])$/)
        end

        def filename(draft = true)
            date = created.to_date.to_s

            normal = title.dup
            normal.gsub!(/([[:space:]]|[^[:alpha:]])+/, '-')
            normal.gsub!(/^-+/, '')
            normal.gsub!(/-+$/, '')
            normal.downcase!

            if draft
                "#{DRAFT_DIR}/#{normal}.#{content_type}"
            else
                "#{POST_DIR}/#{date}-#{normal}.#{content_type}"
            end
        end

        def default_content
            title_esc = @title.gsub(/["\\]/) { |c| "\\" + c }
            content =  "---\n"
            content << "layout: post\n"
            content << "title: \"#{title_esc}\"\n"
            content << "date: #{@created}\n" if @created
            content << "categories:\n"
            content << "---\n"
        end
    end
end
