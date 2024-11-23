bin_SCRIPTS = spex
CLEANFILES = $(bin_SCRIPTS)
EXTRA_DIST = spex.in

do_subst = sed -e 's,[@]pkgdatadir[@],$(pkgdatadir),g' \
	       -e 's,[@]maude[@],$(M3),g'

spex: spex.in Makefile
	$(do_subst) < $(srcdir)/spex.in > spex
	chmod +x spex
