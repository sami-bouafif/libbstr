h_sources = bstr.h
c_sources = bstr.c

include_HEADERS = $(h_sources)

lib_LTLIBRARIES= libbstr.la
libbstr_la_SOURCES= $(h_sources) $(c_sources)
libbstr_la_LDFLAGS= -version-info $(GENERIC_LIBRARY_VERSION) -release $(GENERIC_RELEASE)
