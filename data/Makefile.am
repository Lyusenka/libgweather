
SUBDIRS =

if ENABLE_GLADE_CATALOG
SUBDIRS += glade
endif

libgweatherdtddir = $(pkgdatadir)
libgweatherdtd_DATA = locations.dtd

libgweatherlocationsdir = $(pkgdatadir)
libgweatherlocations_in_files = Locations.xml.in

if USE_ONE_BIG_XML

LOCATIONS_STAMP = 

libgweatherlocations_DATA = $(libgweatherlocations_in_files:.xml.in=.xml$(COMPRESS_EXT))

%.xml$(COMPRESS_EXT): %.xml.in $(wildcard $(top_srcdir)/po-locations/*.po)
	$(AM_V_GEN)LC_ALL=C $(INTLTOOL_MERGE) -x -u -c $(top_builddir)/po-locations/.intltool-merge-cache $(top_srcdir)/po-locations $< `echo $@ | sed "s/.xml$(COMPRESS_EXT)/.xml/"` && \
	if test "x$(COMPRESS_EXT)" = "x.gz"; then				\
		gzip --force `echo $@ | sed "s/.xml$(COMPRESS_EXT)/.xml/"`;	\
	fi

else # USE_ONE_BIG_XML

LOCATIONS_STAMP = stamp-Locations.xml

PO_LOCATIONS = $(shell if test -n "$(LINGUAS)"; then for lang in $(LINGUAS); do if test -f "$(top_srcdir)/po-locations/$$lang.po"; then echo "$(top_srcdir)/po-locations/$$lang.po "; fi; done; else for pofile in $(top_srcdir)/po-locations/*.po; do echo $$pofile; done; fi)

# Helper variable
libgweatherlocations_data = $(libgweatherlocations_in_files:.xml.in=.xml)

libgweatherlocations_DATA = $(shell echo $(PO_LOCATIONS) | sed "s|$(top_srcdir)/po-locations/|Locations.|g;s|\.po|.xml$(COMPRESS_EXT)|g") $(libgweatherlocations_data)$(COMPRESS_EXT)

# We need this step so that we merge all the make Locations.xy.xml destinations
# into one unique destination. This makes -j2 work. (Else, we end up with
# multiple and conflicting calls to intltool-merge)
$(libgweatherlocations_DATA): $(LOCATIONS_STAMP)

$(LOCATIONS_STAMP): $(libgweatherlocations_in_files) $(PO_LOCATIONS) Makefile
	$(AM_V_at)LC_ALL=C $(INTLTOOL_MERGE) --multiple-output --xml-style --utf8 --cache=$(top_builddir)/po-locations/.intltool-merge-cache $(top_srcdir)/po-locations $< $(libgweatherlocations_data) && \
	for pofile in $(PO_LOCATIONS); do \
		locale=`echo $$pofile | sed "s;$(top_srcdir)/po-locations/\(.*\)\.po;\1;"`;	\
		xmllint --noblanks -o Locations.$$locale.xml $$locale/$(libgweatherlocations_data); \
		rm -f $$locale/$(libgweatherlocations_data);		\
		test -d $$locale && rmdir $$locale;			\
		if test "x$(COMPRESS_EXT)" = "x.gz"; then		\
			gzip --force Locations.$$locale.xml;		\
		fi;							\
	done &&								\
	xmllint --noblanks -o Locations.xml C/$(libgweatherlocations_data) && \
	rm -f C/$(libgweatherlocations_data) &&				\
	test -d C && rmdir C &&						\
	if test "x$(COMPRESS_EXT)" = "x.gz"; then			\
		gzip --force Locations.xml;				\
	fi &&								\
	touch $@

endif # USE_ONE_BIG_XML

### Locations.xml.in rebuild
rebuild-locations: locationdb.sqlite update-locations.py
	$(AM_V_GEN)($(srcdir)/update-locations.py > Locations.xml.in.new && mv Locations.xml.in.new Locations.xml.in) || rm -f Locations.xml.in.new

locationdb.sqlite: build-locationdb.pl major-cities.txt sources/nsd_cccc.txt sources/POP_PLACES.txt sources/US_CONCISE.txt sources/geonames_dd_dms_date_*.txt city-fixups.pl station-fixups.pl
	$(AM_V_GEN)$(srcdir)/build-locationdb.pl

check:
	xmllint --valid --noout $(top_srcdir)/data/Locations.xml.in
	$(srcdir)/check-timezones.sh $(srcdir)/Locations.xml.in

EXTRA_DIST =	\
	$(libgweatherlocations_in_files)	\
	$(libgweatherdtd_DATA)			\
	check-timezones.sh			\
	README					\
	README.timezones

CLEANFILES =	\
	$(libgweatherlocations_DATA)		\
	$(LOCATIONS_STAMP)

-include $(top_srcdir)/git.mk
