prefix = "flux"

desc "Full compile"
task :default => [:bibtex, :latex] 

desc "LaTeX log"
file "#{prefix}.log" => "#{prefix}.tex" do
	puts "pdflatex #{prefix}"
	`pdflatex #{prefix}`
end

desc "LaTeX compile"
# Require log file and proceed if references are incomplete
task :latex => "#{prefix}.log"  do
	while ref?(prefix) do
		puts "pdflatex #{prefix}"
		`pdflatex #{prefix}`
	end
end

desc "BibTex compile"
task :bibtex => "#{prefix}.log"  do
	if cite?(prefix)
		puts "bibtex #{prefix}"
		`bibtex #{prefix}`
	end
end

desc "Look at log file and check if references are complete"
def ref?(string)
	dirty = false
	log = File.open("#{string}.log", "r")
	m = log.read.match(/LaTeX Warning: There were undefined references|LaTeX Warning: Label(s) may have changed. Rerun to get|^LaTeX Warning: Reference/)
	log.close
	if m != nil
		puts m
		dirty = true
	end	
	return dirty
end

desc "Look at log file and check if citations are complete"
def cite?(string)
	dirty = false
	log = File.open("#{string}.log", "r")
	m = log.read.match(/^LaTeX Warning: Citation/)
	log.close
	if m != nil
		puts m
		dirty = true
	end	
	return dirty
end