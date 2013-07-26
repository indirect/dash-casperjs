require 'nokogiri'

namespace :docs do
  task :download do
    # Download the API docs
    sh "curl -O https://media.readthedocs.org/dash/casperjs/latest/CasperJS.tgz"
    sh "tar xfz CasperJS.tgz"
    rm "CasperJS.tgz"

    # Copy the Info.plist into the right place
    cp "Info.plist", "CasperJS.docset/Contents"

    # Install the favicon
    cp "icon.png", "CasperJS.docset"
  end

  task :index do
    def sql(*cmd)
      IO.popen('sqlite3 CasperJS.docset/Contents/Resources/docSet.dsidx', 'w+') do |sql|
        cmd.each{|l| sql << l }
      end
    end

    def index(name, type, path)
      sql("INSERT OR IGNORE INTO searchIndex(name, type, path) VALUES ('#{name}', '#{type}', '#{path}');")
    end

    # Create the database and schema
    db = "CasperJS.docset/Contents/Resources/docSet.dsidx"
    rm_rf db if File.exist?(db)
    sql "CREATE TABLE searchIndex(id INTEGER PRIMARY KEY, name TEXT, type TEXT, path TEXT);",
      "CREATE UNIQUE INDEX anchor ON searchIndex (name, type, path);"

    # Index the Casper class, which is a slightly weird page
    index "Casper", "Class", "modules/casper.html#the-casper-class"
    index "Casper.options", "Property", "modules/casper.html#index-1"

    docsdir = "CasperJS.docset/Contents/Resources/Documents"
    casper = Nokogiri::HTML(File.read("#{docsdir}/modules/casper.html"))

    items = casper.css("h3").map do |item|
      name = item.css("tt span")
      next nil if name.empty?
      [name.text, "casper.html" + item.css("a").attr("href").value]
    end.compact

    items.each do |item|
      if item.first =~ /\(\)/
        # this must be an instance function
        index "Casper.#{item.first[0..-3]}", "Method", "modules/#{item.last}"
      else
        # this must be an options attribute
        index "Casper.options.#{item.first}", "Attribute", "modules/#{item.last}"
      end
    end

    # Index the other modules
    %w(Clientutils Colorizer Tester utils).each do |modname|
      index modname, "Class", "modules/#{modname.downcase}.html"

      parsed = Nokogiri::HTML(File.read("#{docsdir}/modules/#{modname.downcase}.html"))
      parsed.css("h3").map do |item|
        name = item.css("tt span")
        next nil if name.empty?
        link = item.css("a").attr("href").value
        index "Casper.#{name.text[0..-3]}", "Method", "modules/#{modname.downcase}.html#{link}"
      end.compact
    end

    # Index the cli sections
    index "Casper.cli", "Property", "cli.html#using-the-command-line"
    index "Casper.cli.raw", "Property", "cli.html#raw-parameter-values"

    # Document the weird require requirement
    index "patchRequire", "Function", "writing_modules.html#writing-casperjs-modules"

    # Index the events and filters list
    parsed = Nokogiri::HTML(File.read("#{docsdir}/events-filters.html"))
    %w(Event Filter).each do |type|
      parsed.css("##{type.downcase}s-reference h4").map do |item|
        name = item.css("tt span")
        next nil if name.empty?
        link = item.css("a").attr("href").value
        index name.text, type, "events-filters.html" + link
      end
    end

  end

  desc 'generate a .docset for CasperJS'
  task :generate => [:download, :index]
end

task :default => "docs:generate"
