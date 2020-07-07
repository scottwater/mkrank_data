namespace :keycaps do
  require "json"

  class MetadataParser
    require "yaml"
    attr_reader :text, :data

    def initialize(text)
      @text = text
      @data = {}
      parse
    end

    def parse
      @text =~ /^(---\s*\n.*?\n?)^(---\s*$\n?)/m
      @data = YAML.load($1) if $1
      @text = $' if $1
    end
  end

  class DataFromMatrixzj
    attr_reader :data
    def initialize(base_path)
      @data = []
      generate_data(base_path || "/Users/scott/src/matrixzj.github.io")
    end

    def type(name)
      name.split(" ")[0].downcase
    end

    def generate_data(base_path)
      keycap_path = "#{base_path}/docs/**/*.md"
      image_regex = /src=\"{{\s*[\'\"](.+)[\'\"]\s*\|\s+(relative_url)*\s*\}\}/i
      ref_link_regex = /^ref link:\s\[(.+)\]\((.+)\)/
      Dir[keycap_path].each do |path|
        md = MetadataParser.new(File.read(path))
        image_matches = md.text.scan(image_regex)
        img = image_matches.find { |m| m[0] =~ /kits_pics/ }
        if img && img[0]
          keycap = {}
          keycap[:title] = md.data["title"].to_s.gsub(/\W|_/, " ").gsub(/\s{2,}/, " ").strip
          keycap[:type] = type(md.data["parent"])
          keycap[:tags] = ["keycaps", type(md.data["parent"])]
          keycap[:uid] = "#{keycap[:type]}-#{keycap[:title].gsub(/\s/, "-")}".downcase.strip
          keycap[:image] = File.join(base_path, img[0])

          md.text.scan(ref_link_regex).each do |name, url|
            (keycap[:links] ||= []) << {name: name, url: url}
          end

          data << keycap
        end
      end
    end
  end

  task :pull do
    DataFromMatrixzj.new(ENV["MATRIXZJ_PATH"]).data.each do |keycap|
      json_path = "data/keycaps/#{keycap[:type]}/#{keycap[:uid]}.json"
      unless File.exist?(json_path)
        puts "Generating #{keycap[:title]} data"
        FileUtils.mkdir_p("data/keycaps/#{keycap[:type]}")
        image_name = "#{keycap[:uid]}#{File.extname(keycap[:image])}"
        image_data_path = "data/keycaps/#{keycap[:type]}/#{image_name}"
        FileUtils.copy(keycap[:image], image_data_path)
        keycap[:image] = image_data_path
        keycap[:title] = "#{keycap[:type].upcase} #{keycap[:title]}"
        File.write("data/keycaps/#{keycap[:type]}/#{keycap[:uid]}.json", keycap.to_json)
      end
    end
  end

  task :addbase do
    Dir["data/keycaps/gmk/*.json"].each do |path|
      data = JSON.parse(File.read(path))
      unless data["skip"]
        unless data["tags"].include?("base")
          data["tags"] << "base"
          File.write(path, data.to_json)
        end
      end
    end
  end
end
