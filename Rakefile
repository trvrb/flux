TEX = FileList["*.tex"]
PDF = TEX.ext("pdf")

desc "Full compile"
task :default => PDF 

desc "LaTeX aux"
rule ".aux" => ".tex" do |t|
	prefix = get_prefix(t.name)
	puts "pdflatex -draftmode #{prefix}"
	`pdflatex -draftmode #{prefix}`
end

desc "LaTeX log"
rule ".log" => ".tex" do |t|
	prefix = get_prefix(t.name)
	puts "pdflatex -draftmode #{prefix}"
	`pdflatex -draftmode #{prefix}`
end

desc "LaTeX compile"
# require log file and proceed if references are incomplete
rule ".pdf" => [".aux", ".bbl", ".log", ".tex"]  do |t|
	prefix = get_prefix(t.name)
#	while ref?(prefix) do
		puts "pdflatex #{prefix}"
		`pdflatex #{prefix}`
#	end
end

desc "BibTex compile"
# look for .bib file in top-level directory
rule ".bbl" => [".aux", ".log", ".tex"]  do |t|
	prefix = get_prefix(t.name)
	if cite?(prefix)
		puts "bibtex #{prefix}"
		`bibtex #{prefix}`
		puts "pdflatex -draftmode #{prefix}"
		`pdflatex -draftmode #{prefix}`
	end
end

desc "Get file prefix"
def get_prefix(file)
	return file.sub(/\.[^.]*$/, "")
end

desc "Look at log file and check if references are complete"
def ref?(string)
	dirty = false
	file = File.open("#{string}.log", "r")
	m = file.read.match(/LaTeX Warning: There were undefined references|LaTeX Warning: Label(s) may have changed. Rerun to get|^LaTeX Warning: Reference/)
	file.close
	if m != nil
		puts m
		dirty = true
	end	
	return dirty
end

desc "Are citations up to date?"
def cite?(string)
	dirty = false
	if File.exists?("#{string}.bbl")
		if cite_missing(string) == true
			dirty = true
		else
			aux_list = cite_aux(string)
			bbl_list = cite_bbl(string)
			extra = (aux_list - bbl_list).length
			missing = (bbl_list - aux_list).length
			if extra > 0 || missing > 0
				dirty = true
			end
		end
	else
		dirty = true
	end
	return dirty
end

desc "Look at log file and check if citations are complete"
def cite_missing(string)
	dirty = false
	file = File.open("#{string}.log", "r")
	m = file.read.match(/^LaTeX Warning: Citation/)
	file.close
	if m != nil
		puts m
		dirty = true
	end	
	return dirty
end

desc "Find citations in .aux"
def cite_aux(string)
	list = Array.new
	file = File.open("#{string}.aux", "r")
	cites = file.read.scan(/\\citation{([^\}]+)}/)
	file.close
	if cites != nil
		cites.each {|m|
			list += m[0].split(",")
		}
	end	
	list.uniq!
	return list
end

desc "Find citations in .bbl"
def cite_bbl(string)
	list = Array.new
	file = File.open("#{string}.bbl", "r")
	cites = file.read.scan(/\\bibitem{([^\}]+)}/)
	file.close
	if cites != nil
		cites.each {|m|
			list += m[0].split(",")
		}
	end	
	list.uniq!
	return list
end