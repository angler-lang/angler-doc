all: clean
	pdflatex tesis.tex
	bibtex tesis.aux
	pdflatex tesis.tex
	pdflatex tesis.tex

clean:
	rm -f *.aux *.l* *.toc *.out *.blg *.bbl *-blx.bib *.run.xml tesis.pdf
