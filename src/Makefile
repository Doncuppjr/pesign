SRCDIR = .
TOPDIR = $(SRCDIR)/..

include $(TOPDIR)/Make.defaults

PKLIBS = nss
LIBS = popt uuid efivar plc4 nspr4
STATIC_LIBS = $(TOPDIR)/libdpe/libdpe.a
LDFLAGS =
CCLDFLAGS = -L../libdpe $(foreach pklib,$(PKLIBS), $(shell pkg-config --libs-only-L $(pklib))) -pie -fPIE -Wl,-z,relro,-z,now
BUILDFLAGS += -I../include/ $(foreach pklib,$(PKLIBS), $(shell pkg-config --cflags $(pklib)))  -fPIE

TARGETS = pesign authvar client efisiglist efikeygen pesigcheck

all : $(TARGETS)

generic_SOURCES = cms_common.c wincert.c password.c ucs2.c oid.c signed_data.c signer_info.c content_info.c
generic_OBJECTS = $(foreach source,$(generic_SOURCES),$(patsubst %.c,%,$(source)).o)
generic_DEPS = $(foreach source,$(generic_SOURCES),.$(patsubst %.c,%,$(source)).P)
generic.a : $(generic_OBJECTS)

authvar_SOURCES = authvar.c authvar_context.c
authvar_OBJECTS = $(foreach source,$(authvar_SOURCES),$(patsubst %.c,%,$(source)).o) generic.a
authvar_DEPS = $(foreach source,$(authvar_SOURCES),.$(patsubst %.c,%,$(source)).P)
authvar : $(authvar_OBJECTS) $(STATIC_LIBS)

pesign_SOURCES = pesign.c pesign_context.c actions.c daemon.c
pesign_OBJECTS = $(foreach source,$(pesign_SOURCES),$(patsubst %.c,%,$(source)).o) generic.a
pesign_DEPS = $(foreach source,$(pesign_SOURCES),.$(patsubst %.c,%,$(source)).P)
pesign : $(pesign_OBJECTS) $(STATIC_LIBS)

pesigcheck_SOURCES = pesigcheck.c pesigcheck_context.c certdb.c
pesigcheck_OBJECTS = $(foreach source,$(pesigcheck_SOURCES),$(patsubst %.c,%,$(source)).o) generic.a
pesigcheck_DEPS = $(foreach source,$(pesigcheck_SOURCES),.$(patsubst %.c,%,$(source)).P)
pesigcheck : $(pesigcheck_OBJECTS) $(STATIC_LIBS)

client_SOURCES = pesign_context.c actions.c client.c
client_OBJECTS = $(foreach source,$(client_SOURCES),$(patsubst %.c,%,$(source)).o) generic.a
client_DEPS = $(foreach source,$(client_SOURCES),.$(patsubst %.c,%,$(source)).P)
client : $(client_OBJECTS) $(STATIC_LIBS)

efisiglist_SOURCES = efisiglist.c siglist.c
efisiglist_OBJECTS = $(foreach source,$(efisiglist_SOURCES),$(patsubst %.c,%,$(source)).o)
efisiglist_DEPS = $(foreach source,$(efisiglist_SOURCES),.$(patsubst %.c,%,$(source)).P)
efisiglist : $(efisiglist_OBJECTS)

efikeygen_SOURCES = efikeygen.c
efikeygen_OBJECTS = $(foreach source,$(efikeygen_SOURCES),$(patsubst %.c,%,$(source)).o) generic.a
efikeygen_DEPS = $(foreach source,$(efikeygen_SOURCES),.$(patsubst %.c,%,$(source)).P)
efikeygen : $(efikeygen_OBJECTS) $(STATIC_LIBS)

DEPS = $(generic_DEPS) $(authvar_DEPS) $(pesign_DEPS) $(client_DEPS) \
	$(pesigcheck_DEPS) $(efisiglist_DEPS) $(efikeygen_DEPS)

deps : $(DEPS)

depclean :
	@rm -fv .*.P

-include $(DEPS)

clean : depclean
	@rm -rfv *.o *.a *.so $(TARGETS)

install_systemd:
	$(INSTALL) -d -m 755 $(INSTALLROOT)/usr/lib/tmpfiles.d/
	$(INSTALL) -m 644 tmpfiles.conf $(INSTALLROOT)/usr/lib/tmpfiles.d/pesign.conf
	$(INSTALL) -d -m 755 $(INSTALLROOT)/usr/lib/systemd/system/
	$(INSTALL) -m 644 pesign.service $(INSTALLROOT)/usr/lib/systemd/system/

install_sysvinit:
	$(INSTALL) -d -m 755 $(INSTALLROOT)/etc/rc.d/init.d/
	$(INSTALL) -m 755 pesign.sysvinit $(INSTALLROOT)/etc/rc.d/init.d/pesign

install :
	$(INSTALL) -d -m 700 $(INSTALLROOT)/etc/pki/pesign/
	$(INSTALL) -d -m 770 $(INSTALLROOT)/var/run/pesign/
	$(INSTALL) -d -m 755 $(INSTALLROOT)$(PREFIX)/bin/
	$(INSTALL) -m 755 authvar $(INSTALLROOT)$(PREFIX)/bin/
	$(INSTALL) -m 755 pesign $(INSTALLROOT)$(PREFIX)/bin/
	$(INSTALL) -m 755 client $(INSTALLROOT)$(PREFIX)/bin/pesign-client
	$(INSTALL) -m 755 efikeygen $(INSTALLROOT)$(PREFIX)/bin/
	$(INSTALL) -m 755 efisiglist $(INSTALLROOT)$(PREFIX)/bin/
	$(INSTALL) -m 755 pesigcheck $(INSTALLROOT)$(PREFIX)/bin/
	$(INSTALL) -d -m 755 $(INSTALLROOT)/etc/popt.d/
	$(INSTALL) -m 644 pesign.popt $(INSTALLROOT)/etc/popt.d/
	$(INSTALL) -d -m 755 $(INSTALLROOT)/usr/share/man/man1/
	$(INSTALL) -m 644 pesign.1 $(INSTALLROOT)/usr/share/man/man1/
	$(INSTALL) -m 644 pesign-client.1 $(INSTALLROOT)/usr/share/man/man1/
	$(INSTALL) -m 644 efikeygen.1 $(INSTALLROOT)/usr/share/man/man1/
	$(INSTALL) -m 644 pesigcheck.1 $(INSTALLROOT)/usr/share/man/man1/
	$(INSTALL) -d -m 755 $(INSTALLROOT)/etc/rpm/
	$(INSTALL) -m 644 macros.pesign $(INSTALLROOT)/etc/rpm/

.PHONY: all deps clean depclean install

include $(TOPDIR)/Make.rules
