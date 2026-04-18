#!/usr/bin/make -f

# Portions of this file contributed by NIST are governed by the
# following statement:
#
# This software was developed at the National Institute of Standards
# and Technology by employees of the Federal Government in the course
# of their official duties. Pursuant to Title 17 Section 105 of the
# United States Code, this software is not subject to copyright
# protection within the United States. NIST assumes no responsibility
# whatsoever for its use by other parties, and makes no guarantees,
# expressed or implied, about its quality, reliability, or any other
# characteristic.
#
# We would appreciate acknowledgement if the software is used.

SHELL := /bin/bash

top_srcdir := $(shell cd .. ; pwd)

uco_srcdir := $(top_srcdir)/dependencies/UCO

RDF_TOOLKIT_JAR := $(uco_srcdir)/lib/rdf-toolkit.jar

# TODO: This variable lists files for Make to generate and delete on
# 'make clean' calls.  This variable should be adapted to list each file
# to normalize.
# Note that to support the catalog-v001.xml preconditions, this list
# must only be files in this directory.
normalized_ttls := \
  formatted-prov-o.ttl

all: \
  catalog-v001.xml \
  .prov-o-rl.ttl.dl.done.log

$(RDF_TOOLKIT_JAR):
	@echo "DEBUG:dependencies/Makefile:top_srcdir=$(top_srcdir)" >&2
	@echo "ERROR:dependencies/Makefile:rdf-toolkit.jar not found.  Did you run `make` from the top source directory?" >&2
	@test -r $@

.prov-o-dl.ttl.dl.done.log: \
  prov-o-dl.ttl \
  $(top_srcdir)/.venv.done.log \
  UCO/ontology/owl/owl.ttl
	source $(top_srcdir)/venv/bin/activate \
	  && pyshacl \
	    --shacl UCO/ontology/owl/owl.ttl \
	    $<
	touch $@

.prov-o-rl.ttl.dl.done.log: \
  prov-o-rl.ttl \
  $(top_srcdir)/.venv.done.log \
  UCO/ontology/owl/owl.ttl
	source $(top_srcdir)/venv/bin/activate \
	  && pyshacl \
	    --shacl UCO/ontology/owl/owl.ttl \
	    $<
	touch $@

# This recipe intentionally depends on normalized_ttls without using
# them, due to using only a subgraph of prov-o.ttl downstream.
catalog-v001.xml: \
  $(normalized_ttls) \
  $(top_srcdir)/.venv.done.log \
  $(top_srcdir)/etc/domain_directories.tsv \
  $(top_srcdir)/etc/dependency_files.tsv \
  $(top_srcdir)/src/create-catalog-v001.xml.py \
  .prov-o-dl.ttl.dl.done.log
	rm -f _$@
	source $(top_srcdir)/venv/bin/activate \
	  && python3 $(top_srcdir)/src/create-catalog-v001.xml.py \
	    _$@ \
	    $(top_srcdir)/etc/domain_directories.tsv \
	    $(top_srcdir)/etc/dependency_files.tsv \
	    "$(top_srcdir)" \
	    prov-o-dl.ttl
	mv _$@ $@

check: \
  all

clean:
	@rm -f \
	  $(normalized_ttls) \
	  .*.done.log \
	  catalog-v001.xml

formatted-prov-o.ttl: \
  $(RDF_TOOLKIT_JAR) \
  $(top_srcdir)/.venv.done.log
	rm -f __$@ _$@
	wget \
	  --output-document __$@ \
	  https://www.w3.org/ns/prov-o
	java -jar $(RDF_TOOLKIT_JAR) \
	  --inline-blank-nodes \
	  --source __$@ \
	  --source-format turtle \
	  --target _$@ \
	  --target-format turtle
	rm __$@
	mv _$@ $@
