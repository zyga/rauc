ACLOCAL_AMFLAGS = -I m4 ${ACLOCAL_FLAGS}
AM_MAKEFLAGS = --no-print-directory
AUTOMAKE_OPTIONS = color-tests parallel-tests

EXTRA_DIST = $(top_srcdir)/.version
BUILT_SOURCES = $(top_srcdir)/.version
$(top_srcdir)/.version:
	echo $(VERSION) > $@-t && mv $@-t $@
dist-hook:
	echo $(VERSION) > $(distdir)/.tarball-version

CODE_COVERAGE_IGNORE_PATTERN = "*-generated.c"
@CODE_COVERAGE_RULES@
@VALGRIND_CHECK_RULES@

GC_CFLAGS = -fdata-sections -ffunction-sections
GC_LDFLAGS = -Wl,--gc-sections -Wl,-Map,$@.map
AM_CFLAGS = -DG_LOG_DOMAIN=\"rauc\" $(GC_CFLAGS) $(WARN_CFLAGS) $(GLIB_CFLAGS) $(CURL_CFLAGS) $(NL3_CFLAGS)
AM_LDFLAGS = $(GC_LDFLAGS) $(WARN_LDFLAGS) $(GLIB_LDFLAGS) $(CURL_LDFLAGS) $(OPENSSL_LDFLAGS)
AM_CPPFLAGS = -I${top_srcdir}/include -include ${top_builddir}/config.h $(OPENSSL_INCLUDES)

gdbus_installer_generated = \
	rauc-installer-generated.c \
	rauc-installer-generated.h

BUILT_SOURCES += $(gdbus_installer_generated)
EXTRA_DIST += src/de.pengutronix.rauc.Installer.xml

$(gdbus_installer_generated): src/de.pengutronix.rauc.Installer.xml
	$(AM_V_GEN) gdbus-codegen \
		--generate-c-code rauc-installer-generated \
		--c-generate-autocleanup all \
		--c-namespace R \
		--interface-prefix de.pengutronix.rauc. \
		$<

noinst_LTLIBRARIES = librauc.la

librauc_la_SOURCES = \
	src/bootchooser.c \
	src/bundle.c \
	src/checksum.c \
	src/config_file.c \
	src/context.c \
	src/dm.c \
	src/install.c \
	src/manifest.c \
	src/mark.c \
	src/mbr.c \
	src/mount.c \
	src/service.c \
	src/signature.c \
	src/slot.c \
	src/utils.c \
	src/update_handler.c \
	src/verity_hash.c \
	include/bootchooser.h \
	include/bundle.h \
	include/checksum.h \
	include/config_file.h \
	include/context.h \
	include/dm.h \
	include/emmc.h \
	include/gpt.h \
	include/install.h \
	include/manifest.h \
	include/mark.h \
	include/mbr.h \
	include/mount.h \
	include/service.h \
	include/signature.h \
	include/slot.h \
	include/update_handler.h \
	include/utils.h \
	include/verity_hash.h

if WANT_EMMC_BOOT_SUPPORT
librauc_la_SOURCES += src/emmc.c
endif

if WANT_NETWORK
librauc_la_SOURCES += src/network.c include/network.h
endif

if WANT_GPT
librauc_la_SOURCES += src/gpt.c
endif

if ENABLE_STREAMING
librauc_la_SOURCES += src/nbd.c include/nbd.h src/stats.c include/stats.h
endif

nodist_librauc_la_SOURCES = \
	$(gdbus_installer_generated)
librauc_la_CFLAGS = $(AM_CFLAGS) $(CODE_COVERAGE_CFLAGS)
librauc_la_LDFLAGS = $(AM_LDFLAGS) $(CODE_COVERAGE_LDFLAGS)
librauc_la_LIBADD = $(GLIB_LIBS) $(CURL_LIBS) $(OPENSSL_LIBS) $(FDISK_LIBS) $(NL3_LIBS)

bin_PROGRAMS = rauc

rauc_SOURCES = \
	src/main.c
rauc_CFLAGS = $(AM_CFLAGS) $(JSON_GLIB_CFLAGS) $(CODE_COVERAGE_CFLAGS)
rauc_LDFLAGS = $(AM_LDFLAGS) $(JSON_GLIB_LDFLAGS) $(CODE_COVERAGE_LDFLAGS)
rauc_LDADD = $(JSON_GLIB_LIBS) $(librauc_la_LIBADD) librauc.la

if SYSTEMD
systemdunitdir=$(SYSTEMD_UNITDIR)
nodist_systemdunit_DATA = data/rauc.service
endif

dbussystemservicedir = $(DBUS_SYSTEMSERVICEDIR)
nodist_dbussystemservice_DATA = data/de.pengutronix.rauc.service

dbuspolicydir = $(DBUS_POLICYDIR)
dist_dbuspolicy_DATA = data/de.pengutronix.rauc.conf

dbusinterfacesdir = $(DBUS_INTERFACESDIR)
dist_dbusinterfaces_DATA = src/de.pengutronix.rauc.Installer.xml

dbuswrapperdir = $(libexecdir)
nodist_dbuswrapper_SCRIPTS = data/rauc-service.sh

EXTRA_DIST += data/rauc.service.in \
	      data/de.pengutronix.rauc.service.in \
	      data/rauc-service.sh.in \
	      README.rst \
	      CHANGES \
	      contrib \
	      build-uncrustify.sh \
	      .uncrustify.cfg \
	      uncrustify.sh \
	      rauc_logo.png \
	      rauc_logo_small.png

dist_man_MANS = rauc.1

AM_TESTS_ENVIRONMENT = \
	AM_TAP_AWK='$(AWK)'; export AM_TAP_AWK; \
	CATCHSEGV='$(CATCHSEGV)'; export CATCHSEGV;
TEST_EXTENSIONS = .t .test
T_LOG_DRIVER = $(top_srcdir)/build-aux/tap-driver.sh
T_LOG_COMPILER = $(top_srcdir)/tap-t
TEST_LOG_DRIVER = $(top_srcdir)/build-aux/tap-driver.sh
TEST_LOG_COMPILER = $(top_srcdir)/tap-test

check_PROGRAMS = \
	test/bootchooser.test \
	test/checksum.test \
	test/config_file.test \
	test/context.test \
	test/dm.test \
	test/manifest.test \
	test/signature.test \
	test/update_handler.test \
	test/utils.test \
	test/install.test \
	test/service.test \
	test/bundle.test \
	test/progress.test \
	test/slot.test

if WANT_NETWORK
check_PROGRAMS += test/network.test
endif

if ENABLE_STREAMING
check_PROGRAMS += test/nbd.test
endif

if WANT_JSON
check_PROGRAMS += test/boot_switch.test
endif

noinst_PROGRAMS = test/fakerand

test_fakerand_SOURCES = test/fakerand.c

check_LTLIBRARIES = librauctest.la

EXTRA_DIST += tap-t \
	      tap-test \
	      qemu-test qemu-test-init qemu-test-rauc-config \
	      test/Dockerfile \
	      test/bin \
	      test/install-content \
	      test/openssl-ca \
	      $(wildcard test/*.raucb) \
	      $(wildcard test/*.raucm) \
	      test/dummy.verity \
	      test/dummy.verity.info \
	      test/minimal-test.conf \
	      test/nginx.conf \
	      test/nginx.htpasswd \
	      test/nginx_backend.py \
	      test/rauc.t \
	      test/rootfs.raucs \
	      test/sharness.sh \
	      test/system.conf \
	      test/test.conf \
	      test/test-global.conf \
	      test/get-coverity.sh \
	      test/openssl-ca-create.sh \
	      test/openssl-ca-check.sh \
	      test/run-coverity.sh

librauctest_la_SOURCES = \
	test/common.c \
	test/common.h \
	test/install-fixtures.c \
	test/install-fixtures.h
librauctest_la_LIBADD = $(GLIB_LIBS) librauc.la

TESTS = $(check_PROGRAMS) test/rauc.t

test_bootchooser_test_SOURCES = test/bootchooser.c
test_bootchooser_test_LDADD = librauctest.la

test_boot_switch_test_CFLAGS = $(AM_CFLAGS) $(JSON_GLIB_CFLAGS)
test_boot_switch_test_LDFLAGS = $(AM_LDFLAGS) $(JSON_GLIB_LDFLAGS)
test_boot_switch_test_SOURCES = test/boot_switch.c
test_boot_switch_test_LDADD = librauctest.la $(JSON_GLIB_LIBS)

test_bundle_test_SOURCES = test/bundle.c
test_bundle_test_LDADD = librauctest.la

test_checksum_test_SOURCES = test/checksum.c
test_checksum_test_LDADD = librauctest.la

test_config_file_test_SOURCES = test/config_file.c
test_config_file_test_LDADD = librauctest.la

test_context_test_SOURCES = test/context.c
test_context_test_LDADD = librauctest.la

test_dm_test_SOURCES = test/dm.c
test_dm_test_LDADD = librauctest.la

test_manifest_test_SOURCES = test/manifest.c
test_manifest_test_LDADD = librauctest.la

test_nbd_test_SOURCES = test/nbd.c
test_nbd_test_LDADD = librauctest.la

test_service_test_CFLAGS = $(AM_CFLAGS) -DTEST_SERVICES=\""$(abs_top_builddir)"\"
test_service_test_SOURCES = test/service.c rauc-installer-generated.h
test_service_test_LDADD = librauctest.la

test_signature_test_SOURCES = test/signature.c
test_signature_test_LDADD = librauctest.la

test_update_handler_test_SOURCES = test/update_handler.c
test_update_handler_test_LDADD = librauc.la librauctest.la

test_utils_test_SOURCES = test/utils.c
test_utils_test_LDADD = librauctest.la

test_install_test_SOURCES = test/install.c
test_install_test_LDADD = librauctest.la

test_network_test_SOURCES = test/network.c
test_network_test_LDADD = librauctest.la

test_progress_test_SOURCES = test/progress.c
test_progress_test_LDADD = librauctest.la

test_slot_test_SOURCES = test/slot.c
test_slot_test_LDADD = librauctest.la

SED_REPLACE = $(SED) \
       -e 's|[@]bindir[@]|$(bindir)|g' \
       -e 's|[@]libexecdir[@]|$(libexecdir)|g' \
       -e 's|[@]datadir[@]|$(pkgdatadir)|g' \
       -e 's|[@]prefix[@]|$(prefix)|g'

data/%: data/%.in
	$(AM_V_GEN)$(MKDIR_P) $(dir $@) && \
	$(SED_REPLACE) $< > $@

SPHINXOPTS          =
SPHINXBUILD         = sphinx-build
PAPER               =
DOCDIR              = $(top_srcdir)/docs
SPHINXBUILDDIR      = $(top_builddir)/docs/build

.PHONY: check-valgrind check-valgrind-tool

doc:
	$(SPHINXBUILD) -b html $(SPHINXOPTS) $(DOCDIR) $(SPHINXBUILDDIR)/html
	@echo
	@echo "Build finished. The HTML pages are in $(SPHINXBUILDDIR)/html."

EXTRA_DIST += docs/conf.py
EXTRA_DIST += docs/images
EXTRA_DIST += docs/extensions
EXTRA_DIST += docs/release-checklist.txt
EXTRA_DIST += docs/favicon.ico docs/RAUC_Logo_outline.svg
EXTRA_DIST += $(wildcard docs/*.rst)

AM_DISTCHECK_CONFIGURE_FLAGS = \
    --with-systemdunitdir=$$dc_install_base/$(systemdunitdir) \
    --with-dbuspolicydir=$$dc_install_base/$(dbuspolicydir) \
    --with-dbussystemservicedir=$$dc_install_base/$(dbussystemservicedir) \
    --with-dbusinterfacesdir=$$dc_install_base/$(dbusinterfacesdir) \
    --enable-gpt

CLEANFILES = $(gdbus_installer_generated) \
	     $(nodist_systemdunit_DATA) \
	     $(nodist_dbussystemservice_DATA) \
	     $(nodist_dbuswrapper_SCRIPTS) \
	     rauc.map \
	     data/rauc.service \
	     test/empty.dat \
	     test/*.map \
	     test/test-results/rauc.*.counts \
	     test/savedmanifest.raucm \
	     test/savedslot.raucs \
	     test/grubenv.test \
	     test/random.dat \
	     test/test-temp.conf \
	     test/system.raucs \
	     test/global.raucs
