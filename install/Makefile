.PHONY: RandRStudioInstall windowsInstall RStudioServerLogin

all: RandRStudioInstall windowsInstall RStudioServerLogin

clean:
	rm -rf *.md *.html 

%.html: %.Rmd
	echo $(<)
	Rscript -e "library(knitr); knit2html('$(<)')"
