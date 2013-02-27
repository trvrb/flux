prefix = "flux"

#desc 'Full compile'
#task :default => '#{prefix}.aux' do
	# nothing left to do
#end

desc "Initial compile"
file "#{prefix}.aux" => "#{prefix}.tex"  do
	puts "1 pdflatex #{prefix}"
	`pdflatex #{prefix}`
end

desc "Bib compile"
file "#{prefix}.bbl" => ["#{prefix}.aux", "#{prefix}.tex"]  do
	puts "1 bibtex #{prefix}"
	`bibtex #{prefix}`
end

desc "Subsequent compiles"
task :compile => ["#{prefix}.aux", "#{prefix}.bbl", "#{prefix}.tex"]  do
	puts "2 pdflatex #{prefix}"
	`pdflatex #{prefix}`
	while dirty?(prefix) do 
		puts "3+ pdflatex #{prefix}"	
		`pdflatex #{prefix}`
	end	
end

desc "Look at log file and check if rerun is necessary"
def dirty?(string)
	dirty = false
	log = File.open("#{string}.log", "r")
	m = log.read.match(/LaTeX Warning: There were undefined references|LaTeX Warning: Label(s) may have changed. Rerun to get/)
	log.close
	if m != nil
		puts m
		dirty = true
	end	
	return dirty
end