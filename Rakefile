require 'fileutils'
require 'rmagick'
require 'pry'
include Magick


FRAME_RATE = 30
SLIDE_DURATION = 1
CROSSFADE_DURATION = 1
ACCEPTED_TYPES = [".gif",".jpg",".png"]



task :default => [:info]

task :info do
  puts "Specify a working folder by name with the start task, for example 'rake start[myfolder_name]'"
end

task :start, [:source_dir] do | t, args |
	
	SOURCE_DIR = "./#{args[:source_dir]}"
	
	
	# Read screens and categories
	screens = []
	categories = {}
	
	Dir.entries(SOURCE_DIR).each do |entry| 
		if File.directory? File.join(SOURCE_DIR,entry) and !(entry =='.' || entry == '..')
			case entry.split("_")[0]
				when "category"
					add_category entry, categories
				when "screen"
					screens.push entry
					puts "Found screen #{entry}"
			end
		end
	end
	
	
	screens.each do |screen|
		slides = []
		temp_categories = Hash.new
		categories.each_pair do |key, arr|
			temp_categories[key] = arr.shuffle
		end
		
		puts "Collecting slides for #{screen}"
		screen_path = File.join SOURCE_DIR, screen
		temp_path = File.join SOURCE_DIR, "temp"
		output_path = File.join SOURCE_DIR, "#{screen}_OUTPUT"
		FileUtils.rm_rf temp_path
		Dir.mkdir temp_path
		FileUtils.rm_rf output_path
		Dir.mkdir output_path
		
		Dir.entries(screen_path).each do |entry| 
			comps = entry.split("_")
			ordinal = comps.shift
			type = comps[0]
			next if !ordinal.match(/\A[0-9]+\Z/)
			case type
				when "category"
					puts "Get category #{comps[1]}"
					slides.push temp_categories[comps[1]].shift
				else
					input_image_path = File.join screen_path, entry
					puts "Use file #{input_image_path}"
					slides.push input_image_path
			end
			
			
			
		end
		
		puts "Created slide list: #{slides}"
		slides.each_index do |i|
			if i === 0
				write_x_fade(slides[slides.length - 1], slides[0], temp_path)
			else
				write_x_fade(slides[i - 1], slides[i], temp_path)
			end
			write_slide slides[i], temp_path
		end
		
		puts "Wrote #{slide_count()} frames to temp directory."
		output_file = "#{screen}.mp4"
		cmd = "ffmpeg -framerate 30 -pattern_type glob -i '#{File.join temp_path, "*.png" }' -c:v libxvid -qscale:v 2 -pix_fmt yuv420p '#{File.join output_path, output_file}'"
		puts "Running #{cmd}"
		encode = system cmd
		if encode
			puts "Successfully encoded #{File.join output_path, output_file}."
		else
			puts "Error encoding file."
		end
		
		

		
	end
	

end
	
	
def write_slide(slide, temp_path)
	slide_image = Image.read(slide).first
	
	(SLIDE_DURATION * FRAME_RATE).times do
		slide_count(1)
		output_file = File.join temp_path, "#{slide_count()}_temp.png"
		slide_image.write output_file
	end	
	puts "Wrote #{(SLIDE_DURATION * FRAME_RATE)} frames of #{slide}."
end

def write_x_fade(slide1, slide2, temp_path)
	
	frames = CROSSFADE_DURATION * FRAME_RATE
	
	bottom = Image.read(slide1).first
	top = Image.read(slide2).first
	
	output_image_path = File.join temp_path, "output.png"

	(0...frames).each do |x|
		slide_count(1)
		output_file = File.join temp_path, "#{slide_count()}_temp.png"
		perc = x / (frames * 1.0)
		result = bottom.blend(top, perc, 1-perc)
		result.write output_file
	end
	
	puts "Wrote x-fade from #{slide1} to #{slide2}"
	
end

def slide_count(inc=0)
	@slide_count ||= 0
	@slide_count += inc
	@slide_count.to_s.rjust(5, "0")
end

# entry is a dirname
# categories is the Hash of category arrays
def add_category(entry, categories)
	
	comps = entry.split("_")
	comps.shift
	catname = comps.join("_")
	valids = Dir.entries(File.join SOURCE_DIR, entry ).select do |image|
		!File.directory?(File.join(SOURCE_DIR,entry,image)) && ACCEPTED_TYPES.include?(image[-4..-1])
	end 
	categories[catname] = valids.map do |image|
		File.join SOURCE_DIR,entry,image
	end
end