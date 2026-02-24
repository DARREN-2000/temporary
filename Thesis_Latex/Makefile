all:
	latexmk -e '$$max_repeat=5' -verbose -f -pdf -bibtex thesis.tex

clean:
	rm -rf *.bbl *.run.xml thesis.pdf
	latexmk -c thesis.tex
	rm -rf `biber --cache`
