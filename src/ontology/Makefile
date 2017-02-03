OBO=http://purl.obolibrary.org/obo
ONT=aro
BASE=$(OBO)/$(ONT)
SRC=$(ONT)-edit.owl
RELEASEDIR=../..
ROBOT= robot
OWLTOOLS= owltools
USECAT= --use-catalog

all: all_imports $(ONT).owl $(ONT).obo
test: all
prepare_release: all
	cp $(ONT).owl $(ONT).obo $(RELEASEDIR) &&\
	mkdir -p $(RELEASEDIR)/imports &&\
	cp imports/*{owl,obo} $(RELEASEDIR)/imports &&\
	git add $(RELEASEDIR)/imports/*{obo,owl} &&\
	(test -f subsets/*owl && cp subsets/*{owl,obo} $(RELEASEDIR)/subsets && git add $(RELEASEDIR)/subsets/* || echo "no subsets") &&\
	echo "Release files are now in $(RELEASEDIR) - now you should commit, push and make a release on github"

$(ONT).owl: $(SRC)
	$(ROBOT)  reason -i $< -r ELK relax reduce -r ELK annotate -V $(BASE)/releases/`date +%Y-%m-%d`/$(ONT).owl -o $@
$(ONT).obo: $(ONT).owl
	$(ROBOT) convert -i $< -f obo -o $(ONT).obo.tmp && mv $(ONT).obo.tmp $@


IMPORTS = chebi
IMPORTS_OWL = $(patsubst %, imports/%_import.owl,$(IMPORTS)) $(patsubst %, imports/%_import.obo,$(IMPORTS))


# Make this target to regenerate ALL
all_imports: $(IMPORTS_OWL)

# Only fetches .owl if it doesn't exist or if .txt has later timestamp.
imports/%_import.owl: imports/%_ontofox.txt
	curl -s -F file=@imports/$*_ontofox.txt -o $@ http://ontofox.hegroup.org/service.php
	$(ROBOT) reduce -i $@ -r ELK -s true --xml-entities -o $@
	# All annotations are manually curated in the ontology-metadata.owl file
	# Because OWLAPI is parsing 1 line commented out annotations, make this step last:
	# Error trap the sed command - on Macs we need the '.bak' after -i to avoid a parsing error; while on other linux it fails
	sed -i '.bak' 's/<owl:AnnotationProperty[^>]*\/>/<!-- & -->/; s/<\/owl:AnnotationProperty>/& -->/; s/<owl:AnnotationProperty[^>]*">/<!-- &/;' $@ ||	sed -i 's/<owl:AnnotationProperty[^>]*\/>/<!-- & -->/; s/<\/owl:AnnotationProperty>/& -->/; s/<owl:AnnotationProperty[^>]*">/<!-- &/;' $@

imports/%_import.obo: imports/%_import.owl
	$(OWLTOOLS) $(USECAT) $< -o -f obo $@


release: $(ONT).owl $(ONT).obo
	cp $^ $(RELEASEDIR) && cp imports/* $(RELEASEDIR)/imports
